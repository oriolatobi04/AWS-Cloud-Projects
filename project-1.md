#  🛡️ Full-Scale AWS Infrastructure Deployment with IAM Control, Networking, Bastion Host & S3 Integration

 This comprehensive hands-on project is designed to simulate the planning, deployment, access control, and secure operation of an AWS-based infrastructure, following enterprise-level best practices. It begins with IAM governance to ensure correct role-based access and builds out a robust VPC environment with public and private subnets, EC2 provisioning, EC2 & S3-hosted content, and SSH-based bastion access. This project provides a step-by-step guide with interactive images of how this can be achieved.

## 🔐 Phase 0: IAM Governance & Role Delegation
👑 Admin User Setup
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
