#  🛡️ Full-Scale AWS Infrastructure Deployment with IAM Control, Networking, Bastion Host & S3 Integration

 This comprehensive hands-on project is designed to simulate the planning, deployment, access control, and secure operation of an AWS-based infrastructure, following enterprise-level best practices. It begins with IAM governance to ensure correct role-based access and builds out a robust VPC environment with public and private subnets, EC2 provisioning, EC2 & S3-hosted content, and SSH-based bastion access. This project provides a step-by-step guide with interactive images of how this can be achieved.

## 🔐 Phase 0: IAM Governance & Role Delegation
### 👑 Admin User Setup
![Screenshot 2025-06-16 125231](https://github.com/user-attachments/assets/9d429bd5-b793-473a-aa39-9a2f2b7446a8)

An AWS Admin user (with AdministratorAccess) is responsible for creating and managing users, groups, and policies. The idea of creating multiple groups is to have control over the permission set for users attached to any of these groups.


#### For this project, the admin will:
##### 1.	Create IAM Groups:
- NetworkAdmin (allow s3, EC2 and VPC full access)
![Screenshot 2025-06-16 125605](https://github.com/user-attachments/assets/71dfec4e-0cd7-4113-9c4b-1c3a947341ba)
-	ComputeEngineer (allow s3, EC2 and VPC full access)
![Screenshot 2025-06-16 125741](https://github.com/user-attachments/assets/01035b95-274d-4ecc-9b05-f9542489d290)
-	StorageManager (allow s3 full access)
![Screenshot 2025-06-16 125846](https://github.com/user-attachments/assets/17d02cd6-2b30-4a09-bcfd-c68982210398)
##### 2.	Assign IAM Users:
-	Create a devops-user
-	Attach the user to all three groups
![Screenshot 2025-06-16 130207](https://github.com/user-attachments/assets/1ea4deb1-6d51-4da4-97c7-69bb59afda93)

##### 3.	Enable MFA and strong password policy for security best practices

![Screenshot 2025-06-16 130420](https://github.com/user-attachments/assets/360e4eb2-8d7a-41ca-b999-05b354143d79)

The MFA is authenticated with using your device. This is required when you are logging in to your devops-user account

![Screenshot 2025-06-16 130442](https://github.com/user-attachments/assets/6802de77-cf7c-479c-b067-f07cfca05a32)


### 👷 DevOps User Flow
The console sign-in link for ‘devops-user’ is required to access the console

![Screenshot 2025-06-16 131015](https://github.com/user-attachments/assets/148c81ea-8b2d-4839-b539-9c8fc7479b63)

Now that we have successfully logged in to our devops-user account, the first thing we notice is that there are limited access to some resources. It is okay! We are only given access ‘by the admin’ to the resources we need JUST for this project.

![Screenshot 2025-06-16 131245](https://github.com/user-attachments/assets/4d2cc748-bacb-47a5-9cfb-30937278eb2b)

### After this project, the devops-user:

- Logs in using their credentials
- Provisions VPC, subnets, EC2, and S3 using their scoped permissions
- Configures the AWS CLI using provided access keys to interact with AWS via terminal
- Launches EC2 instances and uploads website files through SSH and Apache
- Uses IAM roles to interact with S3 and CloudWatch as permitted
- Encounters errors if trying unauthorized actions like IAM changes, which confirms the effectiveness of role-based access controls

⚠️ **Note:** All permissions granted to the devops-user for this project must first be configured and approved by the Admin user. The Admin is responsible for assigning the correct IAM groups and policies to ensure the devops-user has the appropriate access to perform all the tasks in this project.

-----

Here’s an at-a-glance architecture diagram of this project:

![project-1 architecture](https://github.com/user-attachments/assets/74744481-6e32-4400-a9b7-e8b6488f0dbf)

**🚀 Phase 1: Networking & VPC**

- Create a custom VPC with /16 CIDR block

![Screenshot 2025-06-16 132250](https://github.com/user-attachments/assets/96686ede-f1df-4d67-bbe7-0443e66300f4)

---
 
- Create 2 public subnets and 2 private subnets across two Availability Zones

![Screenshot 2025-06-16 141337](https://github.com/user-attachments/assets/1bd4b324-120c-441f-b79f-560a902e09a8)

---

- Enable auto-assign public IPv4 address for public subnets 
- Create and attach an Internet Gateway to the VPC

![Screenshot 2025-06-16 141447](https://github.com/user-attachments/assets/6a1e183b-9435-4dd0-8b12-1b2f55b99513)

---

- Associate route tables to public subnets to route traffic through the Internet Gateway
![Screenshot 2025-06-16 141603](https://github.com/user-attachments/assets/7fa662a9-5ddf-49ae-8b16-9e5c2629b0d2)

---

- Do not associate the Internet Gateway or related routes with private subnets — these must remain isolated from the internet

 ![Screenshot 2025-06-16 141714](https://github.com/user-attachments/assets/75a61608-c136-42de-9a26-74497a8eaa21)

---

- Confirm network configuration using the AWS VPC resource map to ensure correctness
![Screenshot 2025-06-16 141730](https://github.com/user-attachments/assets/b9758b9a-aa27-4326-a004-c5f16b613c9c)
![Screenshot 2025-06-16 141816](https://github.com/user-attachments/assets/493b9e1b-19bb-4650-980d-7e06e17c4714)
![Screenshot 2025-06-16 141829](https://github.com/user-attachments/assets/89abe149-ec46-497f-8f0a-b18885080d65)

---




**We finally completed the network configuration of this project!!!**

**🖥️ Phase 2: AWS CLI Configuration**

- As devops-user, generate a new AWS CLI access key from the IAM console
![Screenshot 2025-06-16 150239](https://github.com/user-attachments/assets/5b5c730e-16f1-4ab5-89c3-824dd94210e4)
![Screenshot 2025-06-16 150324](https://github.com/user-attachments/assets/19516b17-0904-47d0-a5aa-36324e5e6b43)

- Configure the AWS CLI locally using aws configure
```
aws configure
```
- Verify your identity using aws sts get-caller-identity
```
aws sts get-caller-identity
```
- Use CLI to test limited access permissions based on assigned IAM groups
 
- Try commands like aws ec2 describe-vpcs, aws s3 ls, aws ec2 describe-instances to confirm permissions

```
aws ec2 decribe-vpcs
```

**Breakpoint**

- You will notice that you cannot generate an access key because you do not have permission to do that
- Go to your admin user account and allow devops-user to generate access key for ONLY its account. Meaning the resources would be like this “arn:aws:iam::848616929791:user/devops-user” instead of ‘\*’
- To achieve this, you need to add an inline-policy in JSON for devops-user
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowUserSelfKeyManagement",
      "Effect": "Allow",
      "Action": [
        "iam:CreateAccessKey",
        "iam:ListAccessKeys",
        "iam:DeleteAccessKey",
        "iam:UpdateAccessKey"
      ],
      "Resource": "arn:aws:iam::*:user/${aws:username}"
    }
  ]
}

```
 
-----
**🔧 Phase 3: IAM Roles for EC2**

- Create IAM role for EC2 with attached policies:
  - AmazonS3ReadOnlyAccess
![Screenshot 2025-06-16 152730](https://github.com/user-attachments/assets/258a173e-ff51-4768-b0f0-9c4ef26a4e17)

 ```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*",
                "s3:Describe*",
                "s3-object-lambda:Get*",
                "s3-object-lambda:List*"
            ],
            "Resource": "*"
        }
    ]
}
```

This role will allow our EC2 to communicate with content inside of S3 buckets.


**Breakpoint**

- You will notice that you cannot even create a role and list policies, talkless of creating the policy you need for EC2 to have access to s3
- Go to your admin user account and create the policy from there to allow access to s3
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowRoleCreation",
      "Effect": "Allow",
      "Action": [
        "iam:CreateRole",
        "iam:AttachRolePolicy",
        "iam:PutRolePolicy",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
```
-----

#### Hopefully, at this point you have been able to give required permission to an IAM user, create a new in-line policy, modify the inline-policy and attach it to the IAM user. You have also been able to build your own custom network within AWS which includes (VPC, RT, Subnet, IG and CIDR Block IP). We also created a standalone role for a resource to communicate with another resources. We can now configure our aws from our local machine, and access some of our resources depending on the permission that was given to us by the admin-user.

---
**🖥️ Phase 4: EC2 Instance Setup & Security Groups**

- Launch a public EC2 instance (bastion host) in one of the public subnets
- Launch a private EC2 instance in one of the private subnets (You can use launch more like this after one instance is created)
  
![Screenshot 2025-06-16 154407](https://github.com/user-attachments/assets/4778c6b6-8b1b-44c4-9ebe-54187eb6a084)

---
 
- Create a key pair from AWS Console and download the .pem file securely

 ![Screenshot 2025-06-16 153820](https://github.com/user-attachments/assets/2856545a-230c-4b56-a1de-a75cdfea88f0)

---
- Edit Network settings and use the VPC you created
- Also Edit the subnet for the public instance to the public subnet you created. Do the same for Private.
  
![Screenshot 2025-06-16 154236](https://github.com/user-attachments/assets/1b85c9d7-1a22-439d-9bfd-182aec8e182c)

---

- Create and attach security groups:
  - Bastion EC2: Allow SSH (port 22) and HTTP (port 80) from anywhere
  - Private EC2: Allow SSH only from the CIDR block (IPv4) of the public subnet, to simulate access strictly through the bastion host network range. Disable auto-assign public IP
    
 ![Screenshot 2025-06-16 154320](https://github.com/user-attachments/assets/00dc57e9-32dd-4285-bee7-55940ff17510)

---
- Assign IAM roles as required for the public instance. After launching the instances, go to the EC2 console, select the public instance, and choose 'Actions' → 'Security' → 'Modify IAM Role' to attach the previously created IAM role to the instance.
  
![Screenshot 2025-06-16 160656](https://github.com/user-attachments/assets/5b63f249-0a03-4cf3-98ca-7a281c497c43)

  
- Ensure subnet and routing configurations support their roles (internet for public, no internet for private)
-----

**🔐 Phase 5: SSH Access from Terminal**

- From your local terminal, SSH into the bastion EC2 using the .pem key. Refer to your EC2 SSH Client
```
chmod 400 "your-key-pair-name.pem"
```
```
 ssh -i "your-key-pair-name.pem" ec2-user@<your-public-ip>
 ```
  
Upload the same .pem file to the bastion (Public EC2) using SCP
```
scp -i "your-key-pair-name.pem" your-key-pair-name.pem ec2-user@<your-public-ip>:/home/ec2-user
```
#### It is very important to have our key pair id in our EC2 instance block storage so we can connect our public instance with our private instance - Known as Bastion host

- SSH into Private EC2 on the public EC2 instance connect terminal (NOT local terminal, because you can not SSH into your private instance from there) – Ensure that the pem file is present and permission is set to 400 – Refer to private instance SSH Client for the right code after you are sure the pem is in the public instance (this was already done in ther local terminal using SCP in code 3 Phase 4)
 
- Use the bastion EC2 to SSH into the private EC2 using its private IP address
```
ssh -i "Phase4Key.pem" ec2-user@<your-private-ip>
```
- Optionally test EC2 Instance Connect for the Private EC2— it should fail to connect
- Validate connectivity and confirm isolation of the private instance
- Validate the access using ‘hostname’ and ‘ip addr” on your EC instance connect terminal
```
hostname
ip addr
```

**Breakpoint**

- You will notice that you cannot access EC2 instance connect due to permission
- Go to your admin user account and create the policy from there to allow access to EC2Instance connect
```
{
  "Effect": "Allow",
  "Action": [
    "ec2-instance-connect:SendSSHPublicKey",
    "ec2:DescribeInstances",
    "ec2:GetConsoleOutput",
    "ec2:DescribeInstanceStatus"
  ],
  "Resource": "*"
}
```
-----

**🌐 Phase 6: Apache & Static Website with EC2 and S3**

- Install Apache HTTP server on the public EC2 instance using EC2 Instance Connect
```
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>This is just a test for our index page</h1>" > sudo /var/www/html/index.html

cd /var/www/html
#html> ls

#if you can not find index.html
#create another 'index.html' inside the html folder
#html> sudo nano index.html
```
- ‘echo "This is just a test for our index page </h1>" > sudo /var/www/html/index.html’. This creates a file for our static website. Make sure that this index.html file is created by going inside the html folder using ls. If there is nothing in the html folder, create the index.html file inside the html folder
- Edit the web server's root directory /var/www/html/index.html using nano or vi
- Paste the provided HTML content into the index.html file
- Inside the code. 6 images are required. Install them inside your s3 bucket
- Create an S3 bucket in your account and enable public access
  
![Screenshot 2025-06-16 170823](https://github.com/user-attachments/assets/ef2f3afd-c2cf-46af-852f-7fd895e7cf10)

---

- Upload the 6 provided image files to your respective S3 bucket
![Screenshot 2025-06-16 193317](https://github.com/user-attachments/assets/7a873ea6-b865-4330-977f-13529f7d01b3)

---

- Edit the bucket policy to allow the public to view objects
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::your-bucket-name"
      ]
    }
  ]
}
```
- On your EC2 instance, locate your s3 bucket and the files you attached to that s3
 ```
aws ls
```
```
aws s3 ls s3://your-bucket-name
```
- The HTML you pasted contains 6 image placeholders titled from image-1 to image-6. Ensure each image is added to the HTML using <img src="..."> syntax
- For each <img src="image-1"> to <img src="image-6">, update the src attribute to link directly to the publicly accessible image URL in your S3 bucket (e.g., https://<your-bucket>.s3.amazonaws.com/image-1.png)
- All <a href="..."> links in the HTML should be updated to point to your GitHub project page (e.g., https://github.com/<your-username>/<repo-name>)
- Ensure that all 6 images are correctly visible when visiting the public EC2 instance's IP in a browser
![Screenshot 2025-06-16 200628-1](https://github.com/user-attachments/assets/b2845f91-83c5-4109-8bc2-b080040139d3)


We can also host a static website on S3. Upload your index.html to your bucket. Under the properties, enable static web hosting. After creating that, you can access the url of your index.html and your website will appear
![Screenshot 2025-06-16 201548](https://github.com/user-attachments/assets/a147a063-ec8b-4a4e-af3c-d867afe31081)
![Screenshot 2025-06-16 201559](https://github.com/user-attachments/assets/dca61bf5-a77f-4dc0-bbea-d3067ce682fc)
![Screenshot 2025-06-16 201619](https://github.com/user-attachments/assets/7458fcec-aedd-4ecd-9936-41ff910bbae2)
![Screenshot 2025-06-16 201714](https://github.com/user-attachments/assets/35a963c3-fc10-4315-a64e-f7e4ad2a76f4)


-----
**Breakpoint**

- If you do not have permission to perform taks that are required in this section, refer to your admin-user and give necessary permission by adding in-line policies
  ```
  #dev-ops permission to edit bucket policy
  {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:ListPolicies",
        "iam:GetPolicy",
        "iam:GetPolicyVersion",
        "iam:AttachRolePolicy",
        "iam:ListAttachedRolePolicies"
      ],
      "Resource": "*"
    },
    {
    "Effect": "Allow",
    "Action": [
      "access-analyzer:ValidatePolicy"
    ],
    "Resource": "*"
    }
  ]
  }
```
```
**🛑 Phase 7: Controlled Shutdown by Secondary IAM User**

- After completing all phases, a new IAM user (not the devops-user) must perform a shutdown procedure
- This user (ops-user) should only be added to the ComputeEngineer IAM group by the Admin

![Screenshot 2025-06-17 001333](https://github.com/user-attachments/assets/737c493d-bc6a-4f0f-9c01-161cb468b70e)

---
- The Admin must generate and share a new AWS CLI access key for this ops-user together with the sign-in credentials.
- The ops-user must:
  - Configure AWS CLI locally using aws configure
  - Verify limited permission to ONLY EC2 operations
  - Obtain the correct .pem file to SSH into the bastion EC2 instance
  - SSH into the bastion and from there into the private EC2
  - Use aws ec2 stop-instances command to stop both instances from the terminal only
  - Use aws ec2 describe-instance-status to verify shutdown success

![Screenshot 2025-06-17 001536](https://github.com/user-attachments/assets/0fcb9842-8a27-4681-82e7-38d44739c0aa)
![Screenshot 2025-06-17 001933](https://github.com/user-attachments/assets/c40c2804-6b1c-4be2-a075-108644e2bc2c)
![Screenshot 2025-06-17 001956](https://github.com/user-attachments/assets/47f5be3b-2e35-4ab3-9c4a-320349018420)
![Screenshot 2025-06-17 125330](https://github.com/user-attachments/assets/77811efc-8abe-41ad-ba83-f6a1b8fcc555)
![Screenshot 2025-06-17 002043](https://github.com/user-attachments/assets/36342df1-7777-4605-8df3-699d08a7d10e)
![Screenshot 2025-06-17 125350](https://github.com/user-attachments/assets/fba52319-a6b1-47ee-8879-6716e03d7079)


-----
