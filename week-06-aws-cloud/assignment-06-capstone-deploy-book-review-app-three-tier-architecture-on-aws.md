# Assignment 6 — Capstone Assignment — Deploy Book Review App (Three-Tier Architecture) on AWS

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

This is the most important assignment of the course. You will deploy the Book Review App in a fully production-style three-tier architecture on AWS: a Next.js Web Tier behind Nginx and a public ALB, a private Node.js/Express App Tier behind an internal ALB, and a private Multi-AZ MySQL RDS database with a read replica. You are expected to design, deploy, isolate, debug, and document the result independently.

---

# Task 1 — Architecture Diagram

## Goal

Create an architecture diagram showing the custom VPC (10.0.0.0/16), the six subnets across two Availability Zones (two public Web Tier, two private App Tier, two private Database Tier), the public ALB, Web Tier EC2/Nginx, internal ALB, private App Tier EC2, private Multi-AZ RDS with its read replica, and the permitted traffic flow.

### Evidence

#### Diagram image or link

![](<screenshots/Assignment 6/Screenshot of Architecture diagram.png>).

---

# Task 2 — AWS Region & Services Used

## Goal

Record the AWS Region used and list every AWS service used across networking, compute, load balancing, security, and the database.

### Notes

**Region:**

AWS Region

Field	                     Value
Region Name	                 Europe (Stockholm)
Region Code	                 eu-north-1
Availability Zones Used	     eu-north-1a + eu-north-1b.

---

**Services:**

AWS Services Used.

Complete Service Count

Category	       Services Used
Networking	       VPC, Subnets, IGW, NAT Gateway, Elastic IP, Route Tables
Compute	           EC2, Launch Template, Auto Scaling Group, Key Pairs, User Data
Load Balancing	   ALB, Target Group, Listener, Health Checks
Security	       Security Groups (3), KMS Encryption, Key Pair
Database	       RDS MySQL, Multi-AZ, DB Subnet Group, Automated Backups
Total	           20+ AWS services and features

---

# Task 3 — Public Entry Point

## Goal

Confirm the Book Review App loads through the public ALB DNS name.

### Evidence

#### Public ALB DNS

Paste your public ALB DNS name here:

`http://book-review-dev-public-alb-1334174441.eu-north-1.elb.amazonaws.com/`

---

# Task 4 — Evidence Screenshots

## Goal

Capture visual proof of every tier and load balancer.

### Evidence

#### Web EC2

![](<screenshots/Assignment 6/Screenshot of web tier EC2 in public subnet.png>).

---

#### App EC2

![](<screenshots/Assignment 6/Screenshot of app-Ec2.png>).

---

#### Public ALB

![](<screenshots/Assignment 6/Screenshot of public -alb.png>).

---

#### Internal ALB

![](<screenshots/Assignment 6/Screenshot of internal-alb.png>).

---

#### RDS + Replica

![](<screenshots/Assignment 6/Screenshot of RDS+REPLICA(1).png>).
![](<screenshots/Assignment 6/Screenshot of RDS+REPLICA(2).png>)

---

#### App UI proof

Add your screenshot here.

---

# Task 5 — Summary

## Goal

Summarize what worked in the final deployment, the issues encountered and how each was fixed, and the tools or sources used to research and debug.

### Notes

**What worked:**

•  The AWS infrastructure was deployed, and the public ALB became reachable. 
•  SSH access worked using the newly created capstone-aws-key. 
•  The web instances were reachable and Nginx was installed and running. 
•  The public ALB was correctly forwarding traffic to the web target group on port 3000. 
•  Nginx was configured to listen on port 3000 and reverse-proxy requests to the application on port 8080.
.

---

**Issues + fixes:**

Issue 1 — SSH private-key permissions
Private key permissions initially needed tightening.
Fix: an chmod 400 ~/.ssh/capstone-aws-key

Issue 2 — Public ALB returned 502 Bad Gateway
ALB could reach the web target, but the target itself returned 502
Fix: Investigated the EC2 instance and Nginx configuration.

Issue 3 — Target group unhealthy
book-review-dev-web-tg showed targets unhealthy/draining
Fix: Health check reached Nginx on port 3000, but Nginx couldn't reach the backend

Issue 4 — Nothing listening on port 8080
ss and curl showed no application listening on 127.0.0.1:8080
Fix: This identified the actual backend problem

Issue 5 — Old EC2 instance disappeared
SSH to the earlier instance eventually timed out
Fix: Recognized that the ASG could replace instances, so debugging needed to move to the current instance rather than relying on the old server

Issue 6 — Nginx returned 502
Nginx configuration showed proxy_pass http://127.0.0.1:8080
Fix: Confirmed Nginx itself was working; the missing backend on 8080 was the cause.

---

**Tools/sources used:**

AWS Management Console:

EC2 instances
Security groups
Target groups
Load balancers
RDS
Auto Scaling Groups.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post sharing the capstone deployment, including the public ALB DNS (or a redacted screenshot), three to five lines on what you built and why it is production-style, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/hope-odu_devops-aws-terraform-activity-7503808469733408768-ns_y?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFi9QKUB8YFTHgZLGCY97rJ9u0Stdj7th8Q

---

#### Screenshot of LinkedIn post

![](<screenshots/Assignment 6/Screenshot of linkedin post.png>).

---

# Submission Instructions

- Add all required screenshots and links in your submission
- Do not expose passwords, RDS credentials, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: Architecture diagram completed
- [ ] Task 2: AWS Region and services documented
- [ ] Task 3: Public ALB DNS confirmed working
- [ ] Task 4: All six evidence screenshots captured (Web Tier, App Tier, both ALBs, RDS + replica, app UI)
- [ ] Task 5: Deployment summary completed (what worked, issues/fixes, tools/sources)
- [ ] LinkedIn post published and URL submitted
- [ ] App Tier and Database Tier confirmed not publicly accessible
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*