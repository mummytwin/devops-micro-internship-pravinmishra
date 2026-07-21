# Assignment 4 — Building Your AI Team

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build and configure a set of specialized AI subagents inside your project. You will learn how different models and tool permissions define agent behavior, and you will trigger two real agent delegations to analyze security and cost aspects of your Terraform infrastructure.

---

# Task 1 — Create the Agents Folder and Add Files

## Goal

Create the `.claude/agents/` directory and add all required agent files.

### Evidence

#### Screenshot 1 — VS Code sidebar showing `.claude/agents/` with all 3 files

![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 1.png>)

---

# Task 2 — Compare the Agent Configurations

## Goal

Analyze the configuration differences between the three agents and demonstrate understanding of model and tool selection.

### Written Answers

#### 1. Why does the cost optimizer use Haiku instead of Sonnet?

The cost optimizer uses Haiku because Haiku is significantly faster and cheaper than Sonnet, and this task does not require deep reasoning or complex analysis. Speed and cost-efficiency matter here because cost checks may run frequently.

---

#### 2. Why does the security auditor NOT have Write in its tools list?

The security auditor is a read-only agent by design it only needs to read files and grep for patterns to identify security issues. Giving it Write access would violate the principle of least privilege and create unnecessary risk

---

#### 3. Why does the tf-writer use `inherit` instead of a specific model?

The tf-writer uses inherit because it needs to match whatever model the orchestrating agent is already using. Since writing Terraform code requires the same reasoning capability as the parent task that triggered it, inheriting the model ensures consistency and avoids capability mismatches.

---

### Evidence

#### Screenshot 2 — `security-auditor.md` frontmatter showing model and tools configuration

![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 2.png>)

---

#### Screenshot 3 — `cost-optimizer.md` frontmatter showing the model and tools configuration

![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 3.png>)

---

# Task 3 — Run the Security Auditor

## Goal

Trigger the security auditor agent and analyze the generated security report for your Terraform infrastructure.

### Evidence

#### Screenshot 4 — The delegation message showing Claude launched the security-auditor

![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 4.png>)

---

#### Screenshot 5 — Security audit report output

![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 5.png>)

---

# Task 4 — Run the Cost Optimizer

## Goal

Trigger the cost optimizer agent and review the generated cost optimization report.

### Evidence

#### Screenshot 6 — The full cost optimization report

![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 6{1}.png>)
![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 6{2}.png>)
![Assignment 4](<screenshots/Assignment 4 screenshot/Screenshot 6{3}.png>)


---

# Submission Instructions

- Ensure all agent files are committed in `.claude/agents/`
- Complete all written answers in your GitHub Repo
- Push final changes to your forked GitHub repository

---

## GitHub Repository URL

https://github.com/mummytwin/devops-micro-internship-pravinmishra.git

`Add your URL here`

---

# Completion Checklist

- [ ] `.claude/agents/` folder contains all 3 agent files
- [ ] Screenshot 2 shows correct `security-auditor.md` configuration
- [ ] Screenshot 3 shows correct `cost-optimizer.md` configuration
- [ ] All 3 written answers completed 
- [ ] Security auditor executed successfully
- [ ] Cost optimizer executed successfully
- [ ] Security report is visible with findings
- [ ] Cost report is visible with recommendations
- [ ] All required screenshots added
- [ ] GitHub repo updated with agents

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