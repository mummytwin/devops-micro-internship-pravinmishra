# Assignment 5 — AI-Assisted Sprint Health Report via Jira MCP

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will connect Claude Code to your Jira board through an MCP server, the same way you connected it to GitHub in Week 2, and build a read-only `/sprint-health` skill. The skill reads your current sprint through Jira's API and reports sprint velocity, stories at risk of missing the sprint, and items missing an estimate — but it must never create, edit, comment on, or transition a single ticket itself. You will prove that boundary holds by making a real change on the board yourself and confirming the skill only ever reports, never acts.

---

# Task 1 — Create a Jira API Token

## Goal

Generate an API token from your Atlassian account that the MCP server will use to authenticate with your Jira site. Do not screenshot the token value itself.

### Evidence

#### Screenshot 1 — Jira API token creation confirmation page showing the token name, with the token value not visible

![](<screenshots/Assignment 5/Screenshot 1.png>).

### Notes You Must Write (Very Important):

Why does the MCP server need your site URL and account email in addition to the token?

The API token alone only proves you have a valid Atlassian credential, it doesn't specify which Jira site to connect to or which account identity to use. The site URL tells the MCP server exactly which Atlassian instance to target (since one person can have multiple sites). The email identifies the account associated with the token so Jira can apply the correct permissions and return only the data that account is authorized to see. All three are required together to authenticate a specific user on a specific site.

---

# Task 2 — Create .mcp.json at the Project Root

## Goal

Create or update `.mcp.json` at your project root with a Jira MCP server block, following the same shape as the GitHub MCP server you configured in Week 2.

### Evidence

#### Screenshot 2 — `.mcp.json` open in VS Code showing the Jira server configuration

![](<screenshots/Assignment 5/Screenshot 2.png>).

### Notes You Must Write (Very Important):

Compare this jira block to the github block from Week 2 Assignment 5. The GitHub server ran via npx (a Node.js package); this one runs via uvx (a Python package) — what stays exactly the same shape despite that difference, and why doesn't Claude Code care which language a given MCP server is written in?

The shape stays identical, both have command, args, and env fields in the same structure. Claude Code doesn't care whether the MCP server is written in Node.js (launched via npx) or Python (launched via uvx) because MCP is a protocol, not a language. Claude communicates with every MCP server the same way through standard input/output using the MCP message format. The runtime that starts the server is irrelevant once the connection is established.

---

# Task 3 — Add Your Credentials to settings.local.json

## Goal

Add your Jira site URL, account email, and API token to `.claude/settings.local.json`, and confirm that file is listed in `.gitignore` so it is never committed.

### Evidence

#### Screenshot 3 — `settings.local.json` open in VS Code showing the `env` section, with the actual token value blurred or covered

![](<screenshots/Assignment 5/Screenshot 3.png>).

### Notes You Must Write (Very Important):

Why must JIRA_API_TOKEN live in settings.local.json and never in .mcp.json?

.mcp.json is a project configuration file that gets committed to Git and shared with anyone who has access to the repository. If the token were stored there, it would be exposed in version control history permanently even if deleted later, Git retains the history. settings.local.json is gitignored, meaning it stays only on your local machine and is never pushed to any remote. API tokens grant full account access — exposing one in a public or shared repo would allow anyone to read, modify, or delete your Jira data.

---

# Task 4 — Verify the Connection with /mcp

## Goal

Restart Claude Code and confirm the Jira MCP server shows as connected.

### Evidence

#### Screenshot 4 — `/mcp` output showing `jira: connected`

![](<screenshots/Assignment 5/Screenshot 4.png>).

---

# Task 5 — Run a Live Query to Prove Real Board Data

## Goal

Ask Claude to list the issues in your current active sprint through the Jira MCP connection, and confirm the result matches what you see on your live board in the browser.

### Evidence

#### Screenshot 5 — Claude's response showing the live sprint issue list retrieved via Jira MCP

![](<screenshots/Assignment 5/Screenshot 5.png>).

### Notes You Must Write (Very Important):

How did you confirm this was real board data and not something Claude guessed?

I opened my Jira board in the browser simultaneously and compared the Issue Keys, summaries, statuses, and story points returned by Claude against what was visible on the board. The Issue Keys (e.g. DMIWEB-1, DMIWEB-2) are unique identifiers that Claude cannot fabricate they are assigned by Jira when issues are created. Any mismatch between Claude's output and the live board would immediately expose a hallucination. The data matched exactly, confirming the MCP connection was reading live board state.

---

# Task 6 — Build the /sprint-health Skill

## Goal

Create a `/sprint-health` skill restricted to read-only Jira tools plus `Read`, with no issue-mutating tools and no `Write`. Run it and confirm it produces a report covering sprint velocity, at-risk stories, and items missing an estimate.

### Evidence

#### Screenshot 6 — `SKILL.md` frontmatter showing `allowed-tools` limited to read-only Jira tools plus `Read`, with `disable-model-invocation: true`

![](<screenshots/Assignment 5/Screenshot 6.png>).

#### Screenshot 7 — `/sprint-health` output showing the full triage report against your real sprint

![](<screenshots/Assignment 5/Screenshot 7.png>).

### Notes You Must Write (Very Important):

1. Which Jira MCP tools does this skill's allowed-tools list include, and which mutating tools (create issue, update issue, transition issue, add comment) does it deliberately exclude?

The skill includes only read tools: jira_search, jira_get_issue, jira_get_sprint, jira_get_board, and Read. It deliberately excludes all mutating tools including jira_create_issue, jira_update_issue, jira_transition_issue, jira_add_comment, and Write. These are excluded because the skill's only purpose is to report any write capability would create risk of unintended board changes.

2. Why does a Scrum Master need this restriction more than almost any other role in this course?

The Scrum Master is accountable for the integrity of the board every status transition, every comment, every estimate change is a deliberate team decision that must be traceable to a human. If an AI could silently transition issues or add comments, it would corrupt the audit trail that Jira provides, making it impossible to know whether a board update reflected a real team decision or an automated action. The Scrum Master's authority comes from human judgment an AI that acts on the board without approval doesn't assist the Scrum Master, it replaces them.

---

# Task 7 — Prove the Skill Never Mutates the Board

## Goal

Manually update one ticket on your board in the browser (for example, move a story to "Done" or add a missing estimate), then run `/sprint-health` again and confirm the new report reflects your change — proving the skill only ever reads live state and never wrote to the board itself.

### Evidence

#### Screenshot 8 — Second `/sprint-health` run showing the report now reflects your manual board change

![](<screenshots/Assignment 5/Screenshot 8.png>).

### Notes You Must Write (Very Important):

Map this assignment to Gather → Analyze → Human Act → Verify from Week 3 Assignment 6. Which step did you perform manually in the browser, and why must that step stay human?

Gather: /sprint-health calls Jira MCP read tools to collect live sprint data issue statuses, story points, timestamps
Analyze: Claude reads the gathered data and produces the triage report identifying at-risk stories, missing estimates, and velocity
Human Act: I manually moved the story to Done in the Jira browser UI this step must stay human because status transitions represent team decisions and Scrum accountability. An AI transitioning issues without human approval would break the traceability and trust that Scrum depends on
Verify: Running /sprint-health again confirmed the report now reflects the manual change, proving the skill reads live state without ever having written to the board

---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:
- All 8 required screenshots
- All the required notes

---

# Completion Checklist

- [ ] Task 1: Jira API token created, value never screenshotted (Screenshot 1)
- [ ] Task 2: `.mcp.json` has the Jira server block (Screenshot 2)
- [ ] Task 3: Credentials stored in `settings.local.json`, token blurred, file gitignored (Screenshot 3)
- [ ] Task 4: `/mcp` shows the Jira server connected (Screenshot 4)
- [ ] Task 5: Live query returned real sprint data, verified against the browser (Screenshot 5)
- [ ] Task 6: `/sprint-health` skill created with correct read-only `allowed-tools`, and produced a full report (Screenshots 6–7)
- [ ] Task 7: A manual board change was reflected in a second `/sprint-health` run (Screenshot 8)
- [ ] Skill never created, edited, transitioned, or commented on any issue
- [ ] Reflection answered (Notes)
- [ ] No API token value exposed

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
