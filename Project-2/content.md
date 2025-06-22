**🔐 Phase 0 – IAM Governance**

The admin assigns groups to the user and gives them the necessary permissions that are required to perform their operations successfully. Any future permission will be attached to each user as an inline policy.

- **Admin** creates IAM groups:
  ![Screenshot 2025-06-19 103456](https://github.com/user-attachments/assets/284ed510-dd08-4f21-bea6-7f24cd92c23d)
---

  - HA-NetworkAdmin (VPC, SG) – Responsible for the networking side of this project
  ![Screenshot 2025-06-19 145357](https://github.com/user-attachments/assets/b8961ea1-4947-4ed0-9461-8e505fc6cba6)
---

 - HA-ComputeEngineer (EC2, ASG, LT, ALB) – Responsible for the compute side of this project
 ![Screenshot 2025-06-19 145524](https://github.com/user-attachments/assets/1b20be11-4da3-4bc0-b56f-7b4d3b22fe3b)
---

 - HA-StorageManager (EFS, S3) - Responsible for the storage side of this project
 ![Screenshot 2025-06-19 145623](https://github.com/user-attachments/assets/e3f78387-85e4-40e5-b29b-932650578451)
---
    
- **HA-Engineer** is assigned to the three groups to inherit all permissions necessary to complete this project
![Screenshot 2025-06-19 145959](https://github.com/user-attachments/assets/c3cf87c2-9f76-4bcd-897e-f66231a60d59)
---

**🌐 Phase 1 – Networking & VPC**
- VPC CIDR: 10.0.0.0/16
- **Public Subnet A** 10.0.1.0/24 (AZ‑a)
- **Public Subnet B** 10.0.2.0/24 (AZ‑b)
- IGW attached; route table 0.0.0.0/0 → IGW for both public subnets.
- Auto‑assign public IPv4 enabled.
  
![Screenshot 2025-06-20 003313](https://github.com/user-attachments/assets/94e8467c-b3bf-4d8b-9825-592ea238f572)

---


**🗄️ Phase 2 – Security Groups and Amazon EFS**

- Create security group ha-sg for EFS
- **Security‑group rules**:
  - Inbound **NFS 2049** from anywhere
  - Inbound **HTTP 80** from anywhere (for Apache access)
  - Inbound **SSH 22** from anywhere (for admin access)
    
![Screenshot 2025-06-20 005333](https://github.com/user-attachments/assets/4e016de6-5515-4135-a2e7-bddf69485582)

---

- Create the file‑system **ha‑efs** in the VPC.
- Add mount targets in both public subnets using security group sg‑efs.
![Screenshot 2025-06-20 010551](https://github.com/user-attachments/assets/c817e2c3-34df-4391-820b-8a606b10cd12)

**🚀 Phase 3 – Launch Template**

- **Launch Template** (lt‑ha):
  - AMI: Amazon Linux 2023
  - Security Group: sg‑efs
![Screenshot 2025-06-20 013503](https://github.com/user-attachments/assets/4d03c676-69c5-4923-96f1-9879096bcf52)

---

  - User‑data:
    1. Install amazon‑efs‑utils and httpd.
    2. Create mount directory: mkdir -p /mnt/photos. That EFS directory becomes your **shared document root**. Any image or file uploaded here (manually or via script) is immediately available to every EC2 instance in the Auto Scaling Group.
    3. Replace Apache root with EFS
    4. Create default shared index.html echo "&lt;h1&gt;Welcome to the Highly Available Web App&lt;/h1&gt;" > /mnt/photos/index.html
    5. Enable & start Apache.
---
    
```
#!/bin/bash
yum install -y amazon-efs-utils httpd

# Create and mount EFS
mkdir -p /mnt/photos
mount -t efs <your-efs-id>:/ /mnt/photos

# Auto-mount on reboot
echo "<your-efs-id>:/ /mnt/photos efs defaults,_netdev 0 0" >> /etc/fstab

# Replace Apache root with EFS
rm -rf /var/www/html
ln -s /mnt/photos /var/www/html

# Create default shared index.html
echo "<h1>Welcome to the Highly Available Web App</h1>" > /mnt/photos/index.html

# Start Apache
systemctl start httpd
systemctl enable httpd
```

**⚖️ Phase 4 – Application Load Balancer & Auto Scaling Group**

1. **Create Target Group (ha-tg)**  
    • Target type **Instances**, port **80**, health check path **/index.html**.
   
![Screenshot 2025-06-20 013635](https://github.com/user-attachments/assets/71819228-5e2a-4299-8fbc-19ec5c029021)
![Screenshot 2025-06-20 013656](https://github.com/user-attachments/assets/a114245a-fc77-4e03-978e-1bbd0a3adbb1)

---


2. **Create Application Load Balancer (ha-alb)**  
    • Internet-facing, listener **:80**  
    • Attach **existing target group ha-tg**
   
   ![Screenshot 2025-06-20 014106](https://github.com/user-attachments/assets/20bfb776-41b1-4389-9787-dff97e6a6831)

   ---

3. **Create Auto Scaling Group (asg-ha-web)**  
    • Launch Template: **ha-lt** (Phase 3)
    • Attach the existing load balancer and the balancer target groups
   ![Screenshot 2025-06-20 014434](https://github.com/user-attachments/assets/044fa3ba-6f89-4466-80fe-363c1f8d74a9)

---

    • Subnets: both public AZs  
    • Desired = 2, Min = 1, Max = 2  
    • Health check type: **ELB**  
    • Attach to existing target group **ha-tg** (instances auto‑register)

This order guarantees the ASG can immediately register instances with the target group, and the ALB is fully configured (including S3 fail‑over) before traffic arrives.
![Screenshot 2025-06-20 014708](https://github.com/user-attachments/assets/2d382e8d-8927-4376-bb02-a7f81c241c2f)
![Screenshot 2025-06-20 111335](https://github.com/user-attachments/assets/1ff1b437-ea27-459b-a58d-853e7424b4ef)

---


**🗂️ Phase 5 – Static Website Assets & Error Page in S3**

- **Create a single bucket** (e.g., ha-error) to hold both the **error.html** fail‑over page
  
![Screenshot 2025-06-20 113525](https://github.com/user-attachments/assets/17c311bb-1ca8-4999-ada1-a21b40fb7cec)

---
- Enable static web hosting and set error.html as the index document.
  What this means is that the index document is been replaced by the error page.
  ![Screenshot 2025-06-21 180447](https://github.com/user-attachments/assets/cb4b42e6-ae6f-41db-b01e-6906ad4160a6)

  ---

- Make each object **publicly readable** via bucket policy.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicReadAccess",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<your-bucket-name>/*"
        }
    ]
}
```

**🔥 Phase 6 – Simulate instance failure and validate 302 redirect to S3**

1. Navigate to the ALB and edit rules on the existing listener
2. Click on add rule
![Screenshot 2025-06-20 124637](https://github.com/user-attachments/assets/9f803225-869c-4b31-8689-fd9c8f39274a)

---

3. Set the Path condition value to /\*
4. Redirect the URL to the s3 hosted error page and set the priority to 1, status code should be 302
   
![Screenshot 2025-06-20 124908](https://github.com/user-attachments/assets/1c3f458e-1b6b-4cd5-b31e-d64b52429f32)

---


**Expected behavior:** As long as the Auto Scaling instances are healthy, visitors see the updated index.html created by the auto-scaling group. If all targets become unhealthy, the ALB’s listener rule automatically redirects users to the S3‐hosted error.html page.

**🔥 Phase 7 – Fail‑Over & Access Testing**

1. Open the ip addresses of your two running instances to confirm they are well mounted. If they are well mounted. They should have the same shared content from your index.html created in the launch template.
   
![Screenshot 2025-06-20 121446](https://github.com/user-attachments/assets/ba7bb6e9-90bf-48a3-adf3-3aa7d23e2f86)

---

2. Visit ALB DNS → see site served from Auto Scaling instances.
   
![Screenshot 2025-06-20 124233](https://github.com/user-attachments/assets/28fb72ac-91ad-4c7d-9ca3-381c46880d8f)

---


3. Stop/terminate all instances or set health check to fail → ALB should redirect to S3 error page.
 
![Screenshot 2025-06-20 121825](https://github.com/user-attachments/assets/d91f158f-4d20-4b81-8a21-677973c95c89)
![Screenshot 2025-06-21 181601](https://github.com/user-attachments/assets/5ebd8ad3-eedc-4b60-bfcc-58211f13c5d9)

---

4. This failure is temporal because our ASG desired capacity is 2. If the two instances get terminated, ASG automatically spins up another instance. This might take a couple of minutes. However, while our traffic is waiting for the website to be back on, they are automatically redirected to our S3 custom error page. After the instances run → traffic returns to EC2.
   
![Screenshot 2025-06-20 124233-1](https://github.com/user-attachments/assets/255f0dc8-8426-482f-8689-a1182cbe3436)

---
 

