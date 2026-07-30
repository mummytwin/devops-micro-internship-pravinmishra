# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

![](<screenshots/Assignment 6/Screenshot 1.png>).

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

![](<screenshots/Assignment 6/Screenshot 2.png>).

---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

systemctl is-active nginx returns active, confirming the Nginx service is currently running and managed by systemd.

---

**2. What proves that the server is listening for HTTP traffic?**

ss -ltn | grep ':80' shows a socket in LISTEN state on port 80, confirming the server is actively waiting for incoming HTTP connections on that port.

---

**3. Why must you capture a healthy baseline before simulating an incident?**

A healthy baseline gives you a reference point to compare against during and after the incident. Without it, you cannot definitively prove what changed, confirm that your diagnosis is correct, or verify that recovery actually restored the system to its original working state. It is the foundation of evidence-based incident response.

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

![](<screenshots/Assignment 6/Screenshot 3(a).png>)
![](<screenshots/Assignment 6/Screenshot 3(b).png>)


---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

Without project-specific rules, Claude operates on general defaults and may take actions that are inappropriate for the specific environment such as modifying files, executing commands, or making assumptions without evidence.

---

**2. Why is the human required to execute the recovery command?**

Automated recovery without human review can cause more damage than the original incident. For example, restarting a service that crashed due to a misconfiguration will just crash it again. A human reviews the suggested command, understands the context, checks for unintended consequences, and makes an informed decision before acting. This is a core principle of safe agentic AI in production environments.

---

**3. Which rule prevents Claude from making an unsupported diagnosis?**

The rule "Claude must NEVER assume a cause without evidence from the Bash report" prevents unsupported diagnosis. It forces Claude to base every conclusion strictly on the collected evidence rather than guessing, which keeps the analysis trustworthy and audit-ready.

---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results

![](<screenshots/Assignment 6/Screenshot 4(a).png>)
![](<screenshots/Assignment 6/Screenshot 4(b).png>)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The Gather phase is represented by Claude running read-only Bash commands to inspect the server checking Nginx status, port 80, HTTP response, disk, and memory before producing any plan. It collects evidence first, then reasons from it.

---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes — running find ~/linux-triage-project -type f | sort after the Claude Code session showed only CLAUDE.md existed. No scripts or reports were created, confirming Claude respected the read-only constraint.

---

**3. Why is planning before coding useful in DevOps automation?**

Planning forces you to understand what evidence you actually need before writing any code. A script built without a plan often checks the wrong things, misses critical signals, or produces output that is hard to interpret. Planning also reveals gaps for example, realizing you need disk and memory checks alongside service checks that you might miss if you start coding immediately.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array

![](<screenshots/Assignment 6/Screenshot 5(a).png>)
![](<screenshots/Assignment 6/Screenshot 5(b).png>)
![](<screenshots/Assignment 6/Screenshot 5(c).png>)

---

#### Screenshot 6 — Middle section showing check functions and conditionals

![](<screenshots/Assignment 6/Screenshot 6(a).png>).
![](<screenshots/Assignment 6/Screenshot 6(b).png>)
![](<screenshots/Assignment 6/Screenshot 6(c).png>)

---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

![](<screenshots/Assignment 6/Screenshot 7(a).png>)
![](<screenshots/Assignment 6/Screenshot 7(b).png>)
![](<screenshots/Assignment 6/Screenshot 7(c).png>)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

![](<screenshots/Assignment 6/Screenshot 8.png>).

---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**

The checks array stores the names of the health check functions as strings — for example ("check_disk" "check_memory" "check_cpu" "check_nginx_service" "check_port80"). Each string is the exact name of a Bash function defined earlier in the script.

---

**2. How does the `for` loop use that array?**

he for loop iterates through each function name stored in the checks array, calling each one in sequence. For every item in the array, the loop executes that function by name, which runs the corresponding health check, prints the evidence, and updates the exit code if needed. This means adding a new check only requires adding a function and its name to the array the loop handles the rest automatically.

---

**3. Why are the health checks separated into functions?**

Separating checks into functions makes the script modular and maintainable. Each function handles exactly one responsibility checking disk, memory, Nginx status, and so on so if one check needs updating, you edit only that function without touching the rest of the script.

---

**4. What is the purpose of `$(...)` in this script?**

$(...) is command substitution, it runs a command inside the parentheses and captures its output as a value. For example, TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S') runs the date command and stores the result in the variable. Without $(...), the script would store the literal text date '+%Y-%m-%d %H:%M:%S' instead of the actual current timestamp.

---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

Exit codes allow other tools, scripts, and monitoring systems to programmatically read the result without parsing text output. Exit code 0 (HEALTHY) is the universal Linux success signal. Exit code 1 (WARN) signals a degraded but not broken state. Exit code 2 (FAIL) signals a critical failure requiring immediate action. This means the triage script can be plugged into CI/CD pipelines, cron jobs, or alerting systems that act differently based on which code is returned making it production-grade automation rather than just a human-readable report.

---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

#### Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

![](<screenshots/Assignment 6/Screenshot 9.png>).

---

#### Screenshot 10 — Output showing the captured exit code and final summary

![](<screenshots/Assignment 6/Screenshot 10.png>)

---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

HEALTHY — exit code 0, all five checks returned OK with no WARN or FAIL results.

---

**2. Which exact Linux evidence proves the application is serving traffic?**

Two pieces of evidence together prove it: the Nginx service check showing active via systemctl is-active nginx, and the port 80 listener check showing a bound socket via ss -ltnp | grep ':80' confirming the server is actively accepting HTTP connections.

---

**3. Did your script return exit code 0 or 1? Explain why.**

Exit code 0 (HEALTHY) — because all five checks passed without hitting any WARN or FAIL thresholds. The escalate function only raises the exit code, never lowers it, so a clean run where no check triggers a threshold results in the default exit code of 0.

---

**4. What is the difference between a warning and a failure in this script?**

A warning (exit code 1) means a check detected a concerning condition that hasn't yet caused a service disruption for example disk usage above 80% or memory pressure approaching limits. A failure (exit code 2) means a check detected a condition that is actively breaking the service such as Nginx not running or nothing listening on port 80. Warnings require monitoring; failures require immediate action..

---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

![](<screenshots/Assignment 6/Screenshot 11.png>).

---

#### Screenshot 12 — `/linux-triage` output for the healthy server

![](<screenshots/Assignment 6/Screenshot 12.png>)

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

The skill is strictly read-only by design. Bash runs the triage script and inspects the system, Read opens report files for analysis, and Grep searches through output for specific patterns. Write is excluded because the skill must never modify files, configs, or system state that would violate the core safety rule that only a human operator can make changes.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

It prevents Claude from reasoning or hallucinating beyond the evidence collected by the Bash script. When disabled, Claude must base its entire analysis strictly on the actual command output rather than drawing on general knowledge or assumptions which is exactly what evidence-based incident triage requires. Every conclusion must be traceable to a specific line in the report.

---

**3. What part is performed by Bash, and what part is performed by Claude?**

Bash performs the Gather phase it runs the five checks, collects raw system evidence, and saves the timestamped report. Claude performs the Analyze phase it reads the evidence, identifies patterns, determines the most likely cause of any failure, and formulates a suggested recovery command..

---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**

Without evidence, Claude can only guess based on general knowledge, it has no visibility into your actual server state, Nginx version, disk usage, or error logs. With the triage script providing structured evidence, Claude's analysis is grounded in real system data. This makes the diagnosis accurate, reproducible, and audit-ready,the difference between a doctor examining a patient versus guessing symptoms over the phone.

---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails

![](<screenshots/Assignment 6/Screenshot 13.png>)

---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

![](<screenshots/Assignment 6/Screenshot 14.png>)

---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name

![](<screenshots/Assignment 6/Screenshot 15.png>)

---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**

Only two checks actually failed, Check 4 (Nginx service status showing inactive) and Check 5 (nothing listening on port 80). Checks 1, 2, and 3 (disk, memory, CPU) all passed since stopping Nginx doesn't affect system resources.

---

**2. What evidence supports the conclusion that Nginx is unavailable?**

Two pieces of evidence together confirm it: systemctl is-active nginx returned inactive proving the service is stopped, and ss -ltnp found no listener on port 80 proving no process is accepting HTTP connections.

---

**3. Did Claude execute the recovery command? Why is that important?**

No, Claude suggested the recovery command but explicitly stated it did not execute it, ending with "Awaiting human operator approval." This is important because automated recovery without human review can mask deeper problems. If Nginx crashed due to a config error, restarting it blindly would just crash it again a human must review the cause first.

---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

The Gather phase, the Bash script collected raw system evidence (service state, port listeners, disk, memory, CPU) and saved it as a structured, timestamped report without making any changes.

---

**5. Which phase is represented by Claude's explanation?**

The Analyze phase, Claude read the gathered evidence, identified the pattern of failures, determined the most likely cause, and formulated a suggested recovery command for human review.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

![](<screenshots/Assignment 6/Screenshot 16.png>).

---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

![](<screenshots/Assignment 6/Screenshot 17.png>)

---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

![](<screenshots/Assignment 6/Screenshot 18.png>)

---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

![](<screenshots/Assignment 6/Screenshot 19.png>).

---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

I ran sudo systemctl start nginx on the EC2 server after reviewing Claude's suggested recovery command. I made the decision to execute it myself after confirming the evidence supported a simple service restart,no config errors, no resource exhaustion, no port conflicts were detected.

---

**2. What evidence proves that the service recovered?**

hree pieces of evidence confirm recovery: systemctl is-active nginx returned active, curl -I http://localhost returned HTTP/1.1 200 OK, and the second triage script run produced exit code 0 (HEALTHY) with Check 4 and Check 5 both flipping from FAIL back to OK, port 80 listener restored and Nginx service running.

---

**3. Why is the second triage run necessary?**

The second triage run is the VERIFY phase of the Agentic Loop, it provides objective, evidence-based proof that the recovery actually worked. Without it, you only have the recovery command's exit code to go on, which doesn't confirm the application is serving traffic. The second run proves all five checks passed, not just that the systemctl start command completed without error.

---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

Automatic restarts without human review can cause serious harm. If Nginx crashed due to a config syntax error, restarting it blindly would just crash it again, creating a restart loop that masks the real problem. If it crashed due to a security breach or a runaway process consuming resources, restarting it could restore a compromised service or trigger the same resource spike. Automatic restarts also bypass change management processes, making incidents harder to audit and reproduce later.

---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

A chatbot responds to questions based on general knowledge, while this agentic workflow grounds Claude's analysis in real system evidence collected by the Bash script, making every conclusion traceable, auditable, and specific to the actual server state rather than a general guess..

---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Hope Odu

**Date:** 29/07/2026

---

**1. Reported Symptom**

Nginx became unresponsive and the EpicReads website was inaccessible. HTTP requests to port 80 returned connection refused.

---

**2. Evidence Collected**

Check 4 showed systemctl is-active nginx → inactive (FAIL). Check 5 showed no listener bound to port 80 (FAIL). Checks 1-3 confirmed disk at 41%, memory at 34%, and CPU load at 0.00 — all healthy. Exit code 2 (FAIL). Report: triage_report_20260729_091355.txt.

---

**3. Most Likely Cause**

Nginx service was stopped. Resource exhaustion, config errors, and port conflicts were all ruled out by evidence. Single root cause: service not running.

---

**4. Human-Approved Recovery Action**

Hope Odu reviewed Claude's suggested command and executed manually: sudo systemctl start nginx.

---

**5. Verification**

Re-ran linux-triage.sh after recovery. All five checks returned OK. Exit code 0 (HEALTHY). curl -I http://localhost returned HTTP/1.1 200 OK. Two consecutive healthy reports confirmed full recovery..

---

**6. Safety Decision**

Claude analyzed the evidence and suggested the recovery command but did not execute it. Hope Odu reviewed the suggestion, confirmed the evidence supported a simple restart, and executed the command manually. This enforces the principle that AI analyzes and suggests, but humans decide and act.

---

**7. Agentic Loop Mapping**

GATHER: linux-triage.sh collected read-only evidence on the EC2 server and saved a timestamped report
ANALYZE: Claude read the failure report, identified Nginx as stopped, ruled out resource exhaustion, and suggested sudo systemctl start nginx
HUMAN ACT: Hope Odu reviewed Claude's suggestion and executed sudo systemctl start nginx manually
VERIFY: linux-triage.sh re-ran and confirmed HEALTHY (exit code 0) with all five checks passing

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/hope-odu_devops-agenticai-linux-activity-7488203694078050304-JnVy?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFi9QKUB8YFTHgZLGCY97rJ9u0Stdj7th8Q`

---

#### Screenshot — Published LinkedIn post

![](<screenshots/Assignment 6/Screenshot of linkedin post.png>)

---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

https://github.com/mummytwin/linux-triage-project.git

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [ ] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [ ] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [ ] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [ ] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [ ] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [ ] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [ ] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [ ] Incident summary contains all seven required sections
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots and the Bash report
- [ ] Skill does not have Write permission
- [ ] Skill did not execute any recovery commands
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*