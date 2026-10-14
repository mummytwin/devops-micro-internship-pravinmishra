# Assignment 7 — AI-Assisted AWS Security and Cost Audit

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash script that audits the AWS resources you deployed earlier this week — your S3 static site, EC2 instance(s), security groups, RDS database, and EBS volumes — for common security and cost misconfigurations.

You will then connect that script to Claude Code as a reusable `/aws-audit` skill that explains what it found and recommends a fix, without ever making the fix itself.

Finally, you will find a real misconfiguration in your own account, apply the fix yourself, and prove it worked with a second audit run.

---

# Task 1 — Confirm Your AWS Resources and Set Up Your Workspace

## Goal

Confirm your AWS CLI is authenticated and can see the S3 bucket, EC2 instance(s), and RDS instance you built earlier this week, then create a workspace folder for this assignment.

### Evidence

#### Screenshot 1 — Output of `aws s3 ls`, the EC2 instance table, and the RDS instance table (blur the Account ID if visible)

![](<screenshots/Assignment 7/Screenshot 1.png>).

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort`

![](<screenshots/Assignment 7/Screenshot 2.png>).

---

### Notes You Must Write (Very Important)

**1. Which resources from this week's earlier assignments did you see in the listings?**

The listings showed the EC2 instances from the HA deployment (ha-web-asg instances), the RDS MySQL instance (ha-database), and any S3 buckets created during the week. These confirm the resources built across Tasks 1-9 of the HA assignment are still running and accessible via the CLI.

**2. Why must you confirm your resources exist before writing an audit script against them?**

The audit script targets specific resource identifiers S3 bucket names, RDS instance IDs, and security group IDs. If a resource no longer exists or was renamed, the script will return errors or silent failures rather than accurate audit results. Confirming existence first ensures the script runs against real, live resources and produces trustworthy evidence.

---

# Task 2 — Define Safety Rules in CLAUDE.md

## Goal

Create a `CLAUDE.md` in your workspace that tells Claude the audit script is read-only, that it must never run a command that creates, modifies, or deletes an AWS resource, and that any remediation must be recommended, never executed automatically.

### Evidence

#### Screenshot 3 — `CLAUDE.md` open in VS Code showing all four sections

![](<screenshots/Assignment 7/Screenshot 3.png>).

---

### Notes You Must Write (Very Important)

**1. Why should Claude never be given permission to run `revoke-security-group-ingress` itself, even if the fix is obviously correct?**

Security group rules control access to live production resources. An automated revocation based on a misread report, wrong resource ID, or incorrect assumption could lock out legitimate users, break application connectivity, or trigger an outage. The engineer must verify the finding, understand the impact, and consciously approve the change — that accountability cannot be delegated to an AI. A correct fix applied to the wrong security group is still an outage.

**2. Which rule prevents Claude from claiming a finding that the report does not support?**

The rule "Do not claim a finding unless the report contains supporting evidence" prevents unsupported findings. This forces Claude to base every conclusion strictly on the actual script output rather than assumptions, general knowledge, or hallucinations about what the account might contain.

---

# Task 3 — Plan the Audit with Claude Code

## Goal

Ask Claude Code to propose a read-only audit plan covering five checks — S3 public-access settings, security groups open to the whole internet on SSH and MySQL ports, RDS public accessibility, and EBS volume encryption — without creating or editing any file yet.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan

![](<screenshots/Assignment 7/Screenshot 4(a).png>).
![](<screenshots/Assignment 7/Screenshot 4(b).png>).
![](<screenshots/Assignment 7/Screenshot 4(c).png>).
![](<screenshots/Assignment 7/Screenshot 4(d).png>).
![](<screenshots/Assignment 7/Screenshot 4(e).png>).
![](<screenshots/Assignment 7/Screenshot 4(f).png>).
![](<screenshots/Assignment 7/Screenshot 4(g).png>).
![](<screenshots/Assignment 7/Screenshot 4(h).png>).

---

### Notes You Must Write (Very Important)

**1. Which part of this task represents the Gather phase?**

The Gather phase is represented by Claude reading CLAUDE.md and proposing the exact AWS CLI commands that will be used to collect evidence. Claude inspects the project context and safety rules before producing the plan — this is gathering information about what the audit needs to do before any action is taken.

**2. Did every proposed command start with `describe-`, `get-`, or `list-`? Why does that matter?**

Yes — all five proposed commands use read-only prefixes. This matters because AWS CLI commands starting with describe-, get-, and list- are guaranteed to be read-only — they retrieve information without modifying any resource. Commands starting with create-, modify-, delete-, revoke-, or authorize- make changes. Using only read-only prefixes ensures the planning phase cannot accidentally alter the live account regardless of what Claude proposes.

---

# Task 4 — Build the AWS Audit Script

## Goal

Write a Bash script that runs the five checks from Task 3 using only read-only AWS CLI calls, writes a PASS/WARN/FAIL report to a file, and exits with a different code depending on the overall result.

Make it executable and confirm it has no syntax errors.

### Evidence

#### Screenshot 5 — Top section of `aws-audit.sh` showing the variables and the checks array

![](<screenshots/Assignment 7/Screenshot 5.png>).

---

#### Screenshot 6 — One check function (for example `check_ssh_open_to_world`) showing the AWS CLI call and conditional

![](<screenshots/Assignment 7/Screenshot 6.png>).

---

#### Screenshot 7 — Output of `bash -n scripts/aws-audit.sh` and `ls -l scripts/aws-audit.sh`

![](<screenshots/Assignment 7/Screenshot 7.png>).

---

### Notes You Must Write (Very Important)

**1. What is stored in the checks array, and how does the loop use it?**

The checks array stores the names of the five check functions as strings. The for loop iterates through each name and calls it as a function using "$check_function"  this means adding a new check only requires adding the function and its name to the array, without changing the main execution logic.

**2. Why does every AWS CLI call in this script use `--query` and `--output text` instead of parsing raw JSON?**

--query extracts exactly the field needed using JMESPath, and --output text returns a plain string instead of JSON. This makes the result directly comparable in a bash if statement without needing jq or any JSON parser. It also makes the script portable, it runs on any system with AWS CLI installed..

**3. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

Exit codes allow other tools, monitoring systems, and CI/CD pipelines to programmatically read the audit result without parsing text. Exit 0 (HEALTHY) is the universal success signal. Exit 1 (WARN) signals degraded security posture. Exit 2 (FAIL) signals a critical finding requiring immediate attention. This makes the script production-grade — it can trigger alerts or block deployments based on the exit code alone.

---

# Task 5 — Run the Baseline Audit

## Goal

Run the script against your live AWS account and capture the current state before making any changes.

### Evidence

#### Screenshot 8 — Output of `./scripts/aws-audit.sh` showing your Full Name and all five checks

![](<screenshots/Assignment 7/Screenshot 8.png>).

---

#### Screenshot 9 — Output showing the captured exit code and final summary

![](<screenshots/Assignment 7/Screenshot 9.png>).

---

### Notes You Must Write (Very Important)

**1. What is the overall status of your baseline audit?**

Since at least one FAIL result was present, the script exited with code 2 and the overall status was recorded as FAIL in reports/aws-audit-report.txt.

**2. Did any check return FAIL or WARN? If so, which one, and what evidence did it show?**

Yes. Two checks returned FAIL and one returned WARN.

FAIL: SSH open to the world. The report showed 1 security group(s) allow SSH (port 22) from 0.0.0.0/0. This was a real finding: the security group attached to my EC2 instances allowed SSH from any IP address on the internet.

FAIL: S3 public ACLs. The report showed BlockPublicAcls=None, IgnorePublicAcls=None. When I investigated, I found this was a false finding. The script was still pointing at a bucket name from an earlier setup instead of my current bucket (hope-odu-audit-7088). The None values appear when the AWS CLI call errors out, so they didn’t mean the bucket was misconfigured. After I corrected the bucket name in the script, this check passed.


The other two checks passed: no security group allowed MySQL (port 3306) from 0.0.0.0/0, and the RDS instance ha-database was not publicly accessible..

**3. If every check passed, what does that tell you about the security posture of your account so far?**

My baseline did not fully pass, so this is hypothetical. If every check had passed, it would mean the five things the script checks (S3 public ACLs, SSH and MySQL open to the world, RDS public access, EBS encryption) were in good shape. It would not mean the account was fully secure, because the script doesn’t cover IAM, MFA, logging or other ports. A pass can also be empty if there are no resources to check..

---

# Task 6 — Build and Run the /aws-audit Skill

## Goal

Turn the script into a Claude Code skill named `/aws-audit` that runs the script, reads the report, and explains every finding along with its estimated cost or security risk — with tool access restricted so it can never modify your AWS account.

### Evidence

#### Screenshot 10 — `SKILL.md` showing the frontmatter, tool restrictions, and safety rules

![](<screenshots/Assignment 7/Screenshot 10.png>).

---

#### Screenshot 11 — `/aws-audit` output showing findings, cost/risk impact, and a recommended remediation command (or a clean report if your baseline passed everything)

![](<screenshots/Assignment 7/Screenshot 11.png>).

---

### Notes You Must Write (Very Important)

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

The skill only needs to run the audit script, read the report, and search it for findings, which Bash, Read, and Grep cover. Write isn’t needed, so it’s left out as least privilege. Without it, Claude can’t edit or overwrite files, including the report or script, even by mistake..

**2. What part is performed by Bash, and what part is performed by Claude?**

Bash does the evidence gathering. The script runs the same read-only AWS CLI commands every time and records a PASS, WARN, or FAIL for each check in the report. Claude does the analysis. It reads the report, explains each finding, estimates the cost or risk impact, and recommends a remediation command for me to review and run myself..

**3. Why is estimating cost/risk impact something the AI adds on top of a plain PASS/FAIL script?**

A plain script can only report PASS, WARN, or FAIL. It can’t say what a finding means in practice. Claude adds that context. For example, unencrypted EBS volumes cost nothing extra but are a compliance risk, while an open SSH port is a security exposure and an idle EC2 instance is a monthly cost. That helps me decide which finding to fix first.

---

# Task 7 — Fix a Real Finding and Re-Verify

## Goal

Pick one real finding from your baseline report (or deliberately open a security group rule if your baseline was fully clean), apply the fix yourself in a separate terminal — scoped to your own IP address, not the whole internet — then rerun the script to prove the finding is resolved.

### Evidence

#### Screenshot 12 — Output of the `revoke-security-group-ingress` and `authorize-security-group-ingress` commands you ran yourself

![](<screenshots/Assignment 7/Screenshot 12.png>).

---

#### Screenshot 13 — Rerun of `./scripts/aws-audit.sh` showing the finding is now PASS

![](<screenshots/Assignment 7/Screenshot 13.png>).

---

### Notes You Must Write (Very Important)

**1. Which exact finding did you fix, and what command did you run?**

I fixed the SSH open to the world finding: the audit reported that 1 security group allowed port 22 from 0.0.0.0/0. I first added a rule allowing SSH only from my own IP, then removed the open rule.

**2. Why did you scope the new rule to your own IP address instead of leaving it open to `0.0.0.0/0`?**

0.0.0.0/0 lets anyone on the internet try to connect to SSH, and automated scanners find open port 22 within hours. Scoping the rule to my own IP (/32) means only my address can reach SSH, which follows least privilege and shrinks the attack surface while I can still administer the instance.

**3. Did Claude execute the remediation command, or did you? Why does that matter?**

I ran the remediation command myself. Claude only analyzed the report and recommended a fix, and the skill was set up so it can’t change my account. This matters because a wrong reading of the evidence could turn into an outage or a security problem. For example, a bad security group change could lock me out of my own instance. Keeping the human as the one who reviews and runs the command keeps accountability with me.

**4. Which phase of the Agentic Loop does the Bash script represent? Which phase does Claude's explanation represent? Which phase is you running the fix?**

The Bash script is the Gather phase, because it collects evidence with read-only AWS CLI calls. Claude’s explanation is the Analyze phase, where it interprets the findings and estimates cost and risk. Me running the fix is the Human Act phase. Re-running the audit afterward is the Verify phase, which closes the loop.

---

# LinkedIn Post (Required)

## Goal

Create a LinkedIn post including:

- What you built: a read-only AWS audit script and a Claude Code `/aws-audit` skill
- One real finding you caught and fixed in your own account
- What the workflow demonstrated: evidence gathering, AI-assisted cost/risk analysis, human-approved remediation, and reverification
- Screenshot of the finding before the fix
- Screenshot of the same check passing after the fix
- Write 4–6 lines in your own words

Suggested tags:

`#DMIByPravinMishra #AWS #AgenticAI #ClaudeCode #DevOps`

### Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`https://lnkd.in/p/enXkanTC`

---

#### Screenshot of Published LinkedIn Post

![](<screenshots/Assignment 7/Screenshot of linkedin post.png>).

---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:

- All 13 required task screenshots
- Answers to every **Notes You Must Write** question
- `CLAUDE.md`
- `scripts/aws-audit.sh`
- `.claude/skills/aws-audit/SKILL.md`
- `reports/aws-audit-report.txt` baseline report and the reverified report from Task 7
- GitHub folder or repository URL containing the assignment files
- Your Full Name visible in the required outputs
- LinkedIn post URL
- Screenshot of the published LinkedIn post
- GitHub repository URL (containing all assignment files)

---

# Completion Checklist

- [ ] Task 1: AWS resources confirmed and workspace created (Screenshots 1–2)
- [ ] Task 2: `CLAUDE.md` created with project context and safety rules (Screenshot 3)
- [ ] Task 3: Claude produced a read-only five-check audit plan before any script existed (Screenshot 4)
- [ ] Task 4: `aws-audit.sh` built, executable, and passes `bash -n` (Screenshots 5–7)
- [ ] Task 5: Baseline audit captured and saved with Full Name visible (Screenshots 8–9)
- [ ] Task 6: `/aws-audit` skill loads and runs successfully with no Write permission (Screenshots 10–11)
- [ ] Task 7: A real finding was fixed by you and reverified as PASS (Screenshots 12–13)
- [ ] Skill never executed a remediation command
- [ ] New security group rule is scoped to your own IP, not `0.0.0.0/0`
- [ ] All 13 required task screenshots are included
- [ ] All "Notes You Must Write" questions are answered in your own words
- [ ] No AWS credentials or unblurred account IDs exposed
- [ ] LinkedIn post published and URL submitted
- [ ] GitHub repository URL included in submission
- [ ] All assignment files committed and visible in GitHub repository

---

# Final Submission

Submit your GitHub repository URL containing all assignment files, screenshots, reports, and output.

### GitHub Repository URL

Paste your GitHub repository URL here:

`Add your GitHub repository URL here`

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

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*