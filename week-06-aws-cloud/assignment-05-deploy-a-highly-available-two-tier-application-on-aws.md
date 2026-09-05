# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS (VPC + ALB + ASG + Multi-AZ RDS)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will design and deploy a highly available two-tier web application on AWS: highly available networking across two Availability Zones, an Application Load Balancer, an Auto Scaling Group for the web tier, and a private Multi-AZ RDS database. You must prove high availability with real failure tests.

---

# Task 1 — Create HA Networking (VPC + 4 Subnets + IGW + NAT + Route Tables)

## Goal

Build a VPC (10.0.0.0/16) with two public and two private subnets across two Availability Zones, an Internet Gateway, a NAT Gateway, and the matching public/private route tables.

### Evidence

#### Screenshot 1 — VPC details showing CIDR 10.0.0.0/16

![](<screenshots/Assignment 5/Screenshot 1.png>).

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![](<screenshots/Assignment 5/Screenshot 2(a).png>).
![](<screenshots/Assignment 5/Screenshot 2(b).png>)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

![](<screenshots/Assignment 5/Screenshot 3.png>).

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![](<screenshots/Assignment 5/Screenshot 4.png>).

---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![](<screenshots/Assignment 5/Screenshot 5.png>).

---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![](<screenshots/Assignment 5/Screenshot 6.png>).

---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![](<screenshots/Assignment 5/Screenshot 7.png>).

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![](<screenshots/Assignment 5/Screenshot 8.png>).

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![](<screenshots/Assignment 5/Screenshot 9.png>).

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![](<screenshots/Assignment 5/Screenshot 10.png>).

---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![](<screenshots/Assignment 5/Screenshot 11.png>).

---

#### Screenshot 12 — A running instance created from the template showing that the application responds on port 80 through a local test or browser using its public IP

![](<screenshots/Assignment 5/Screenshot  12.png>).

---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![](<screenshots/Assignment 5/Screenshot 13.png>).

---

#### Screenshot 14 — Target group showing at least one healthy target

![](<screenshots/Assignment 5/Screenshot 14.png>).

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![](<screenshots/Assignment 5/Screenshot 15(a).png>).
![](<screenshots/Assignment 5/Screenshot 15(b).png>)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![](<screenshots/Assignment 5/Screenshot 16.png>).

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![](<screenshots/Assignment 5/Screenshot 17.png>).

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![](<screenshots/Assignment 5/Screenshot 18.png>).

---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB.

Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance; timestamps are helpful

![](<screenshots/Assignment 5/Screenshot 19.png>).

---

#### Screenshot 20 — Target group showing healthy targets after replacement

![](<screenshots/Assignment 5/Screenshot 20.png>).

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![](<screenshots/Assignment 5/Screenshot 21(a).png>).

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![](<screenshots/Assignment 5/Screenshot 22.png>).

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram, which may be hand-drawn, or an AWS console overview showing the components

![](<screenshots/Assignment 5/Screenshot 23.png>).

---

### Notes

Summarize the VPC and subnets across the two Availability Zones.

VPC Name: MY-HA-APP
CIDR Block: 10.0.0.0/16
Region: eu-north-1 (Europe — Stockholm)
Internet Gateway: ha-igw (attached to MY-HA-APP VPC)
NAT Gateway: ha-nat-gw (deployed in ha-public-1a with Elastic IP).\
Both public subnets span two Availability Zones (eu-north-1a and eu-north-1b) ensuring the ALB distributes traffic across two independent fault domains. Both private subnets also span two AZs ensuring the Multi-AZ RDS has a primary instance in eu-north-1a and an automatic standby in eu-north-1b for failover.

Summarize the ALB and Auto Scaling Group setup.

The ASG maintains a minimum of 2 instances, one in each Availability Zone at all times. If either instance fails its ELB health check, the ASG automatically terminates it and launches a replacement in the same AZ. The ALB detects unhealthy targets and stops routing traffic to them within seconds, while continuing to serve traffic through the healthy instance in the other AZ. Once the replacement instance passes its health checks, the ALB resumes sending traffic to both AZs restoring full redundancy.

Summarize the private Multi-AZ RDS setup.

The RDS instance uses Multi-AZ deployment which maintains a synchronous standby replica in a separate Availability Zone. AWS automatically replicates all data from the primary instance in eu-north-1b to the standby in eu-north-1a in real time. If the primary instance fails, experiences a hardware fault, or requires maintenance, AWS automatically promotes the standby to primary within 60-120 seconds without any manual intervention. The endpoint DNS name remains the same throughout failover so the application reconnects automatically.

Summarize the results of both high-availability tests.

The ALB is the critical HA component it continuously monitors instance health and reroutes traffic within seconds of detecting a failure. The ASG ensures minimum capacity is always maintained by automatically replacing terminated instances. The Multi-AZ RDS provides database-level HA independently of the web tier even if the primary database AZ fails, the standby promotes automatically without any application change required. Together these three components ALB, ASG, and Multi-AZ RDS create redundancy at every layer of the two-tier architecture ensuring no single point of failure can take down the application.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/hope-odu_highly-available-two-tier-application-on-activity-7501260315515564034-9b1n?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFi9QKUB8YFTHgZLGCY97rJ9u0Stdj7th8Q`

---

#### Screenshot of LinkedIn post

![](<screenshots/Assignment 5/Screenshot of linkedin post.png>).

---

# Submission Instructions

- Add all required screenshots in your submission
- Do not expose passwords, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: VPC, four subnets, IGW, NAT Gateway, and route tables created (Screenshots 1–5)
- [ ] Task 2: Least-privilege ALB, EC2, and RDS security groups created (Screenshots 6–8)
- [ ] Task 3: Private Multi-AZ RDS created (Screenshots 9–10)
- [ ] Task 4: Self-configuring Launch Template created and tested (Screenshots 11–12)
- [ ] Task 5: ALB created across both public subnets (Screenshots 13–14)
- [ ] Task 6: Auto Scaling Group running two instances across two AZs (Screenshots 15–16)
- [ ] Task 7: Application verified through the ALB with a database read and write (Screenshots 17–18)
- [ ] Task 8: Both high-availability tests completed (Screenshots 19–22)
- [ ] Task 9: Architecture and test-results summary completed (Screenshot 23 & Notes)
- [ ] LinkedIn post published and URL submitted
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