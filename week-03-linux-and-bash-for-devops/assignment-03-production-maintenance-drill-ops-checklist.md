# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

![](<screenshots/Assignment 3/Screenshot 1(task 1).png>)

---

#### Screenshot 2 — Output of `ip a`

![](<screenshots/Assignment 3/Screenshot 2(task 1).png>)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![](<screenshots/Assignment 3/Screenshot 3(task 1).png>)

---

#### Screenshot 4 — Output of `sudo ufw status`

![](<screenshots/Assignment 3/Screenshot 4(task 1).png>)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

Nginx is proven to be listening on 0.0.0.0:80 by the output of the sudo ss -tulpen command. It shows Nginx in the LISTEN state on 0.0.0.0:80, meaning the web server is accepting HTTP connections on all network interfaces.

---

**2. What proves SSH is active on port 22?**

SSH is proven to be active on port 22 by the output of the sudo ss -tulpen command. It shows a service in the LISTEN state on 0.0.0.0:22, confirming that the SSH server is running and accepting remote connections on port 22..

---

**3. Did you find any unexpected open ports? Explain briefly.**

No, I did not find any unexpected open ports. The only open ports were port 22 for SSH remote access and port 80 for the Nginx web server. This indicates that only the required services are exposed, which helps reduce the server's security risk..

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![](<screenshots/Assignment 3/Screenshot 1(task 2).png>).

---

#### Screenshot 2 — Output of `sudo nginx -t`

![](<screenshots/Assignment 3/Screenshot 2(task 2).png>).

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![](<screenshots/Assignment 3/Screenshot 3(tasks 2).png>).

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart in production, the web application may become unavailable to users because the web server is no longer serving HTTP requests. This can lead to website downtime, failed user requests, and a poor user experience. It is important to test the Nginx configuration with sudo nginx -t before restarting to avoid service interruptions.

---

**2. What's your basic rollback plan?**

I would restore the previous working Nginx configuration or application files from a backup, test the configuration using sudo nginx -t, and restart Nginx. Finally, I would verify that the application is accessible to confirm the rollback was successful.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![](<screenshots/Assignment 3/Screenshot 1(task 3).png>).

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![](<screenshots/Assignment 3/Screenshot 2(task 3).png>).

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![](<screenshots/Assignment 3/Screenshot 3(task 3).png>).

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No, I did not find any errors in the logs. The log entries show HTTP status code 200, which indicates that Nginx successfully handled the requests and served the React application without any issues. This confirms that the web server was functioning correctly during testing..

---

**2. If there were no errors, what does that indicate about the system?**

The absence of errors in the error log indicates that the system is operating normally. It means Nginx is running correctly, the React application is being served successfully, and there were no recent issues such as configuration problems, failed requests, or service crashes during the time of the check. This suggests the web server is stable and functioning as expected.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes, my curl requests were visible in the access log. The log entries show requests with the User-Agent curl/8.18.0 and a 200 OK status code. This proves that the requests successfully reached the Nginx web server, were processed correctly, and the server returned a successful response. It confirms that network traffic is flowing properly between the client and the server.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![](<screenshots/Assignment 3/Screenshot 1(task4).png>).

---

#### Screenshot 2 — Output of `free -h`

![](<screenshots/Assignment 3/Screenshot 2(task4).png>).

---

#### Screenshot 3 — Output of `df -h`

![](<screenshots/Assignment 3/Screenshot 3(task 4).png>).

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![](<screenshots/Assignment 3/Screenshot 4(task 4).png>).

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Disk usage is the most critical resource because it has the highest utilization (60%). However, the server is still healthy since the CPU load is 0.00 and there is sufficient available memory (567 MiB), indicating no immediate performance concerns.

---

**2. What happens if disk becomes 100% full in a production server?**

If the disk becomes 100% full on a production server, the system may not be able to create or update files, including application data, logs, and temporary files. This can cause applications to fail, services like Nginx to stop working correctly, and users may experience downtime or errors. Monitoring disk usage and cleaning up unnecessary files regularly helps prevent this issue.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![](<screenshots/Assignment 3/Screenshot 1(task 5).png>).

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![](<screenshots/Assignment 3/Screenshot 2(task 5).png>).

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![](<screenshots/Assignment 3/Screenshot 3(task 5).png>).

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

I confirmed that the correct version of the application was deployed by checking the contents of the /var/www/html directory using ls -lah to verify that the React production build files, including index.html and the static folder, were present. I also verified that my custom changes had been deployed by confirming that the application displayed the "Deployed by: Hope Odu" line, along with the deployment date. Since Nginx was configured to use /var/www/html as its web root, I confirmed it was serving the correct application. Finally, I opened the application in a web browser using the EC2 instance's public IP address and verified that it loaded successfully and displayed the updated content, confirming a successful deployment.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![](<screenshots/Assignment 3/Screenshot 1(tasks 6).png>).

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

![](<screenshots/Assignment 3/Screenshot 2(task 6).png>).

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![](<screenshots/Assignment 3/Screenshot 3(task 6).png>).

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

The configuration failure was caused by a syntax error in the Nginx configuration file. Specifically, a required semicolon (;) was removed from one of the directives, causing sudo nginx -t to fail the configuration validation. Because of the invalid syntax, Nginx could not reload or restart until the error was corrected.

---

**2. How did you fix the issue?**

I identified the syntax error using sudo nginx -t, corrected the missing semicolon in the configuration file, verified the configuration again until it passed, and then restarted Nginx using sudo systemctl restart nginx. This restored the web server to normal operation..

---

**3. How can you avoid this kind of issue in real production systems?**

To avoid configuration issues in production, always test the Nginx configuration with sudo nginx -t before restarting the service. Maintain backups, use version control, test changes in a staging environment, and monitor the server after deployment to ensure everything works correctly.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![](<screenshots/Assignment 3/Screenshot 1 (task 7).png>).

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![](<screenshots/Assignment 3/Screenshot 2(task 7).png>).

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The application broke because the required application files were unavailable or incorrectly deployed, preventing Nginx from serving the React application. During the failure simulation, the application became inaccessible until the correct build files were restored to the Nginx web root (/var/www/html). This demonstrated how missing or incorrect deployment files can cause an application outage.

---

**2. How did you fix the issue and restore the application?**

I fixed the issue by restorimg the correct React build files to /var/www/html, verified the files were present, restarted Nginx, and tested the application in the browser using the server's public IP. The application loaded successfully with my updated details, confirming the issue had been resolved..

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

To prevent this kind of issue in a real production environment, I would use a structured deployment process. First, I would test the application in a staging environment before deploying it to production. I would keep backups of the previous working version so that I could quickly roll back if needed. I would also use version control (such as Git) and a CI/CD pipeline to automate deployments and reduce human error. After deployment, I would verify that the application files were correctly copied, ensure Nginx was serving the correct web root, and monitor the application and server logs to detect any issues early. These practices help improve reliability, reduce downtime, and make recovery faster if a problem occurs.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH key-based authentication is more secure than passwords because it uses a pair of cryptographic keys: a private key stored securely on the user's device and a public key stored on the server. Unlike passwords, private keys are extremely difficult to guess or brute-force. Since the private key is never transmitted over the network during authentication, the risk of interception or theft is greatly reduced. This makes SSH key authentication a safer and more reliable method for accessing production servers.

---

**2. Why should only required ports be open on a production server?**

Only the required ports should be open on a production server to reduce the system's attack surface and improve security. Every open port is a potential entry point for attackers, so limiting access to only essential services such as SSH (port 22) for administration and HTTP/HTTPS (ports 80 and 443) for web traffic—helps prevent unauthorized access and reduces the risk of cyberattacks. It also simplifies server management and supports the principle of least privilege.

---

**3. Why is it important for Nginx to be enabled on boot?**

It is important for Nginx to be enabled on boot so that the web server starts automatically whenever the server is restarted or recovers from an unexpected shutdown. This ensures that the website or application remains available without requiring manual intervention, reducing downtime and improving service reliability. Automatic startup is an important best practice for maintaining production systems.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Sharing secrets, keys, or credentials publicly can give unauthorized users access to servers, applications, databases, or cloud resources. This can lead to data breaches, service disruptions, financial losses, and unauthorized changes to systems. To protect sensitive information, credentials should be stored securely using secret management tools or environment variables and should never be committed to public repositories or shared online.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Unused cloud resources should be stopped or terminated to reduce costs, improve security, and prevent unnecessary resource consumption. This helps maintain an efficient, well-managed, and secure cloud environment.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/hope-odu_devops-aws-linux-activity-7487491065235615763-ve3T?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFi9QKUB8YFTHgZLGCY97rJ9u0Stdj7th8Q`

---

#### Screenshot — Published LinkedIn post

![](<screenshots/Assignment 3/Screenshot on linkedin post(tasks 8).png>)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
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