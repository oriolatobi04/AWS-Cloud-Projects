**🔐 Phase 0 – IAM Governance**

The admin assigns groups to the user and gives them the necessary permissions that are required to perform their operations successfully. Any future permission will be attached to each user as an inline policy.

- **Admin** creates IAM groups:
  - HA-NetworkAdmin (VPC, SG) – Responsible for the networking side of this project
  - HA-ComputeEngineer (EC2, ASG, LT, ALB) – Responsible for the compute side of this project
  - HA-StorageManager (EFS, S3) - Responsible for the storage side of this project
- **HA-Engineer** is assigned to the three groups to inherit all permissions necessary to complete this project

**🌐 Phase 1 – Networking & VPC**

- VPC CIDR: 10.1.0.0/16
- **Public Subnet A** 10.1.1.0/24 (AZ‑a)
- **Public Subnet B** 10.1.2.0/24 (AZ‑b)
- IGW attached; route table 0.0.0.0/0 → IGW for both public subnets.
- Auto‑assign public IPv4 enabled.

**🗄️ Phase 2 – Security Groups and Amazon EFS**

- Create security group ha-sg for EFS
- **Security‑group rules**:
  - Inbound **NFS 2049** from sg‑web‑efs (self‑reference)
  - Inbound **HTTP 80** from anywhere (for Apache access)
  - Inbound **SSH 22** from anywhere (for admin access)
- Create the file‑system **ha‑efs** in the VPC.
- Add mount targets in both public subnets using security group sg‑web‑efs.

**🚀 Phase 3 – Launch Template**

- **Launch Template** (lt‑ha):
  - AMI: Amazon Linux 2023
  - Security Group: sg‑efs
  - User‑data:
        1. Install amazon‑efs‑utils and httpd.
        2. Create mount directory: mkdir -p /mnt/photos. That EFS directory becomes your **shared document root**. Any image or file uploaded here (manually or via script) is immediately available to every EC2 instance in the Auto Scaling Group.
        3. Replace Apache root with EFS
        4. Create default shared index.html echo "&lt;h1&gt;Welcome to the Highly Available Web App&lt;/h1&gt;" > /mnt/photos/index.html
        5. Enable & start Apache.

**⚖️ Phase 4 – Application Load Balancer & Auto Scaling Group**

Follow this order to avoid dependency problems:

1. **Create Target Group (ha-tg)**  
    • Target type **Instances**, port **80**, health check path **/index.html**.
2. **Create Application Load Balancer (ha-alb)**  
    • Internet-facing, listener **:80**  
    • Attach **existing target group ha-tg**
3. **Create Auto Scaling Group (asg-ha-web)**  
    • Launch Template: **ha-lt** (Phase 3)  
    • Subnets: both public AZs  
    • Desired = 2, Min = 1, Max = 2  
    • Health check type: **ELB**  
    • Attach to existing target group **ha-tg** (instances auto‑register)

This order guarantees the ASG can immediately register instances with the target group, and the ALB is fully configured (including S3 fail‑over) before traffic arrives.

**🗂️ Phase 5 – Static Website Assets & Error Page in S3**

- **Create a single bucket** (e.g., ha-error) to hold both the **error.html** fail‑over page _and_ the six websie image**s**
- Set error.html as the error and index document.
- Make each object **publicly readable** via bucket policy.

**🔥 Phase 6 – Simulate instance failure and validate 302 redirect to S3**

1. Navigate to the ALB and edit rules on the existing listener
2. Click on add rule
3. Set the Path condition value to /\*
4. Redirect the URL to the s3 hosted error page and set the priority to 1, status code should be 302 and create

**Expected behavior:** As long as the Auto Scaling instances are healthy, visitors see the updated index.html created by the auto-scaling group. If all targets become unhealthy, the ALB’s listener rule automatically redirects users to the S3‐hosted error.html page.

**🔥 Phase 7 – Fail‑Over & Access Testing**

1. Open the ip addresses of your two running instances to confirm they are well mounted. If they are well mounted. They should have the same shared content from your index.html created in the launch template.
2. Visit ALB DNS → see site served from Auto Scaling instances.
3. Stop/terminate all instances or set health check to fail → ALB should redirect to S3 error page.
4. Restart ASG desired capacity → traffic returns to EC2.
5. Verify EFS consistency by uploading a file to /mnt/photos on one instance and reading from the other.
