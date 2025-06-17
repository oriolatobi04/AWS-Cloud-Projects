**🛡️ Full-Scale AWS Infrastructure Deployment with IAM Control, Networking, Bastion Host & S3 Integration**

This comprehensive hands-on project is designed to simulate the planning, deployment, access control, and secure operation of an AWS-based infrastructure, following enterprise-level best practices. It begins with IAM governance to ensure correct role-based access and builds out a robust VPC environment with public and private subnets, EC2 provisioning, EC2 & S3-hosted content, and SSH-based bastion access. The project concludes with real-world operations like automated shutdowns, CLI-based controls, and full GitHub documentation, testing your proficiency across IAM, networking, compute, storage, and DevOps practices.-----

**🔐 Phase 0: IAM Governance & Role Delegation**

**👑 Admin User Setup**

An AWS Admin user (with AdministratorAccess) is responsible for creating and managing users, groups, and policies. The admin will:

1. Create IAM Groups:
   1. NetworkAdmin (allow s3, EC2 and VPC full access)
   1. ComputeEngineer (allow s3, EC2 and VPC full access)
   1. StorageManager (allow s3 full access)
1. Assign IAM Users:
   1. Create a devops-user
   1. Attach the user to all three groups
1. Enable MFA and strong password policy for security best practices

**👷 DevOps User Flow**

After assignment, the devops-user:

- Logs in using their credentials
- Provisions VPC, subnets, EC2, and S3 using their scoped permissions
- Configures the AWS CLI using provided access keys to interact with AWS via terminal
- Launches EC2 instances and uploads website files through SSH and Apache
- Uses IAM roles to interact with S3 and CloudWatch as permitted
- Encounters errors if trying unauthorized actions like IAM changes, which confirms the effectiveness of role-based access controls

⚠️ **Note:** All permissions granted to the devops-user for this project must first be configured and approved by the Admin user. The Admin is responsible for assigning the correct IAM groups and policies to ensure the devops-user has the appropriate access to perform all the tasks in this project.

-----
**🚀 Phase 1: Networking & VPC**

- Create a custom VPC with /16 CIDR block
- Create 2 public subnets and 2 private subnets across two Availability Zones
- Enable auto-assign public IPv4 address for public subnets
- Create and attach an Internet Gateway to the VPC
- Associate route tables to public subnets to route traffic through the Internet Gateway
- Do not associate the Internet Gateway or related routes with private subnets — these must remain isolated from the internet
- Confirm network configuration using the AWS VPC resource map to ensure correctness
- Optional: Add NAT Gateway to private subnets if they need limited outbound internet access
-----
**🖥️ Phase 2: AWS CLI Configuration**

- As devops-user, generate a new AWS CLI access key from the IAM console
- Configure the AWS CLI locally using aws configure
- Verify your identity using aws sts get-caller-identity
- Use CLI to test limited access permissions based on assigned IAM groups
- Try commands like aws ec2 describe-vpcs, aws s3 ls, aws ec2 describe-instances to confirm permissions

**Breakpoint**

- You will notice that you cannot generate an access key because you do not have permission to do that
- Go to your admin user account and allow devops-user to generate access key for ONLY its account. Meaning the resources would be like this “arn:aws:iam::848616929791:user/devops-user” instead of ‘\*’
- To achieve this, you need to add an inline-policy in JSON for devops-user
-----
**🔧 Phase 3: IAM Roles for EC2**

- Create IAM role for EC2 with attached policies:
  - AmazonS3ReadOnlyAccess
  - CloudWatchAgentServerPolicy
- Attach this role to both EC2 instances

**Breakpoint**

- You will notice that you cannot even create a role and list policies, talkless of creating the policy you need for EC2 to have access to s3 and CloudWatch
- Go to your admin user account and create the policy from there to allow access to s3 and CloudWatch
-----
**🖥️ Phase 4: EC2 Instance Setup & Security Groups**

- Launch a public EC2 instance (bastion host) in one of the public subnets
- Launch a private EC2 instance in one of the private subnets
- Create a key pair from AWS Console and download the .pem file securely
- Edit Network settings and use the VPC you created
- Also Edit the subnet for the public instance to the public subnet you created. Do the same for Private.
- Create and attach security groups:
  - Bastion EC2: Allow SSH (port 22) and HTTP (port 80) from anywhere
  - Private EC2: Allow SSH only from the CIDR block (IPv4) of the public subnet, to simulate access strictly through the bastion host network range. Disable auto-assign public IP
- Assign IAM roles as required for the public instance. After launching the instances, go to the EC2 console, select the public instance, and choose 'Actions' → 'Security' → 'Modify IAM Role' to attach the previously created IAM role to the instance.
- Ensure subnet and routing configurations support their roles (internet for public, no internet for private)
-----
**🔐 Phase 4: SSH Access from Terminal**

- From your local terminal, SSH into the bastion EC2 using the .pem key. Refer to your EC2 SSH Client
- Upload the same .pem file to the bastion (Public EC2) using SCP
- Use the bastion EC2 to SSH into the private EC2 using its private IP address
- Optionally test EC2 Instance Connect for the Private EC2— it should fail to connect
- Validate connectivity and confirm isolation of the private instance
  - SSH into Private EC2 on the public EC2 instance connect terminal – Ensure that the pem file is present and permission is set to 400 – Refer to SSH Client for the right code after you are sure the pem is in the public instance (this was already done in ther local terminal using SCP in Step 3 Phase 4)
  - Validate the access using ‘hostname’ and ‘ip addr” on your EC instance connect terminal

**Breakpoint**

- You will notice that you cannot access EC2 instance connect due to permission
- Go to your admin user account and create the policy from there to allow access to EC2Instance connect
-----
**🌐 Phase 5: Apache & Static Website with EC2 and S3**

- Install Apache HTTP server on the public EC2 instance using EC2 Instance Connect
- ‘echo "<h1>This is just a test for our index page </h1>" > sudo /var/www/html/index.html’. This creates a file for our static website. Make sure that this index.html file is created by going inside the html folder using ls. If there is nothing in the html folder, create the index.html file inside the html folder
- Edit the web server's root directory /var/www/html/index.html using nano or vi
- Paste the provided HTML content into the index.html file
- Inside the code. 6 images are required. Install them inside your s3 bucket
- Create an S3 bucket in your account and enable public access
- Upload the 6 provided image files to your respective S3 bucket
- Update the S3 Bucket Policy to allow the public to view objects
- On your EC2 instance, locate your s3 bucket and the files you attached to that s3
- Remember, when we installed the Apache Web 
- The HTML contains 6 image placeholders titled from image-1 to image-6. Ensure each image is added to the HTML using <img src="..."> syntax
- For each <img src="image-1"> to <img src="image-6">, update the src attribute to link directly to the publicly accessible image URL in your S3 bucket (e.g., https://<your-bucket>.s3.amazonaws.com/image-1.png)
- All <a href="..."> links in the HTML should be updated to point to your GitHub project page (e.g., https://github.com/<your-username>/<repo-name>)
- Ensure that all 6 images are correctly visible when visiting the public EC2 instance's IP in a browser
- To host a website on S3. Upload your index.html to your bucket. Under the properties, enable static web hosting. After creating that, you can access the url of your index.html and your website will appear
- This tests your understanding of S3 permissions, HTML editing, S3 and EC2-based web hosting

-----
**Breakpoint**

- If you do not have permission to do anything that are required in this section, refer to your admin-user and give necessary permission by adding in-line policies

**✅ Phase 6: Access Testing**

- Public EC2 instance is accessible from your local machine using SSH and its public IP
- Private EC2 instance is only accessible from the public instance (bastion) via SSH
- Ensure IAM policies restrict the devops-user from performing unauthorized actions
- Attempt to use EC2 Instance Connect on the private EC2 directly — it should fail
- Try uploading an object to S3 from a user without s3:PutObject permission — confirm it's denied
- Verify that CloudWatch logs are not accessible unless the role has permissions
- Attempt to detach IAM role from EC2 as devops-user — expect failure unless explicitly allowed
- Use AWS CLI to describe VPCs — ensure it's permitted by the assigned policy
-----
**🛑 Phase 7: Controlled Shutdown by Secondary IAM User**

- After completing all phases, a new IAM user (not the devops-user) must perform a shutdown procedure from the terminal only
- This user (ops-user) should only be added to the ComputeEngineer IAM group by the Admin
- The Admin must generate and share a new AWS CLI access key for this ops-user together with the sign-in credentials.
- The ops-user must:
  - Configure AWS CLI locally using aws configure
  - Verify limited permission to ONLY EC2 operations
  - Obtain the correct .pem file to SSH into the bastion EC2 instance
  - SSH into the bastion and from there into the private EC2
  - Use aws ec2 stop-instances command to stop both instances from the terminal only
  - Use aws ec2 describe-instance-status to verify shutdown success
  - If the private EC2 does not shut down from the terminal, please shut it down from the console under ops-user account. Mr. Daniel said something about not been able to access the private instance from the terminal.
-----


**📂 Phase 8: GitHub Deliverables & Documentation**

- Create a folder named screenshots/ in your project repository. Inside this folder, organize all your screenshots clearly by phase (e.g., phase-0-iam-setup, phase-1-vpc, phase-2-cli, etc.). Each image file should be named to reflect the specific step it represents (e.g., iam-user-creation.png, vpc-cidr.png, ec2-security-group.png).
- Add all screenshots of your AWS Console and terminal commands for every phase:
  - IAM user and group creation
  - IAM policy attachments
  - VPC and subnet setup
  - Route tables and internet gateway configuration
  - EC2 instance launches
  - Security group settings
  - CLI setup and identity verification
  - Apache server installation
  - S3 bucket and image uploads
  - SSH session captures (local to bastion, bastion to private EC2)
  - Web server output showing linked images
- Add this README file as your main documentation
- Commit your changes with clear, meaningful messages describing each addition
- Push the entire folder (including the screenshots and README) to a new GitHub repository
- Ensure the repository includes:
  - Clear project overview
  - Phase-by-phase documentation
  - Full screenshots to demonstrate successful completion
  - Access testing results
  - Bonus challenge notes (if completed)
  - Author and license section

This phase tests your understanding of Git, documentation, and your ability to present infrastructure work professionally.

- Optionally use .gitignore to exclude temporary files or credentials
- Tag your final version (e.g., v1.0) as a release in GitHub
-----
**📝 Phase 9: Final Submission as PDF**

- In addition to your GitHub repository, create a detailed PDF report of the entire project
- The PDF must include:
  - A structured breakdown of all phases with step-by-step explanations
  - Embedded screenshots (same as the ones uploaded in the screenshots/ folder)
  - Clear headings and descriptions of what was done in each step
  - AWS Console views and CLI outputs where relevant
  - Any challenges encountered and how they were resolved
- The PDF file should be named aws-secure-infra-report.pdf and uploaded in the root of your GitHub repository
- This serves as your final project submission





