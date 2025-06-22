**☁️ Highly‑Available AWS Web Application – Auto Scaling, ALB, EFS & S3 Fail‑Over**

**🔍 The Problem**

Modern web applications, especially content-heavy or event-driven websites, face several common challenges:

- **Unpredictable traffic spikes** that overwhelm fixed infrastructure
- **Single points of failure** (a server dies, the whole site goes down)
- **Inconsistent content** when multiple web servers aren’t synchronized
- **Poor user experience** during outages (e.g., raw 503 errors)

A solution must provide:

- Elastic compute capacity
- Shared file consistency
- Load-balanced access across zones
- A resilience fail-over path

**✅ The Solution: What This Architecture Implements**

This highly available architecture addresses these problems by combining key AWS services into a fault-tolerant stack:

| **Component** | **Purpose** |
| --- | --- |
| **Auto Scaling Group (ASG)** | Automatically adds EC2 instances based on failures |
| **Application Load Balancer** | Distributes traffic across AZs, performs health checks, triggers fail-over |
| **Amazon EFS** | Centralized file storage shared across all instances |
| **Amazon S3 Static Hosting** | Acts as a public-facing backup page if the main web tier fails |
| **Multi-AZ Network Design** | Guarantees resilience even if an AZ goes down |

**💼 Where This Is Applied in the Real World**

| **Scenario** | **How This Architecture Solves It** |
| --- | --- |
| A news site experiences a surge during breaking news | ASG launches new EC2s instantly to serve traffic |
| A web server crashes during a live campaign | ALB stops routing traffic to it and replaces it via ASG |
| A shared image is uploaded by a content editor | All EC2s access it instantly via EFS |
| Traffic spikes cause latency issues | Auto Scaling expands capacity based on CPU usage |
| All EC2s fail during a deployment | ALB redirects users to a styled error page in S3—no broken links |


![project-2 drawio](https://github.com/user-attachments/assets/71b2e3a8-1dfa-4236-a48d-f0a57bff946c)
