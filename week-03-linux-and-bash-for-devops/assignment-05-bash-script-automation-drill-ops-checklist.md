# Assignment 5 — Bash Script Automation Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will practice Bash scripting by building a series of small automation scripts covering environment setup, variables, arrays, loops, file conditionals, if-else logic, and functions. These scripts form the foundation of real-world Linux automation used in DevOps, cloud, and production support environments.

---

# Task 1 — Bash Environment & Workspace Setup

## Goal

Verify that Bash is available on your system and create a clean workspace for this assignment.

### Evidence

#### Screenshot 1 — Output of `echo $SHELL` and `bash --version`

![](<screenshots/Assignment 5/Screenshot 1(task 1).png>)

---

#### Screenshot 2 — Output of `pwd` and `ls -lah` showing the scripts directory

![](<screenshots/Assignment 5/Screenshot 2(task 1).png>)

---

### Notes

Answer the following in your own words:

**1. What is Bash?**

Bash (Bourne Again SHell) is a command-line interpreter for Linux/Unix systems. It lets you interact with the operating system by typing commands, and also allows you to write scripts files containing sequences of commands that can be executed automatically.

---

**2. What is the difference between shell and Bash?**

A shell is a general term for any program that provides a command-line interface between the user and the operating system. Bash is one specific type of shell the most widely used on Linux systems. So Bash is a shell, but not all shells are Bash.

---

**3. Why is it important to confirm the Bash version before writing scripts?**

Confirming the version ensures your scripts will run as expected in your environment.

---

# Task 2 — Your First Bash Script

## Goal

Create your first Bash script, make it executable, and run it from the terminal.

### Evidence

#### Screenshot 1 — Content of `first-script.sh`

![](<screenshots/Assignment 5/Screenshot 1(task 2).png>)

---

#### Screenshot 2 — Output of `./first-script.sh`

![](<screenshots/Assignment 5/Screenshot 2 task 2.png>).

---

#### Screenshot 3 — Output of `ls -l first-script.sh` showing executable permission

![](<screenshots/Assignment 5/Screenshot 3(task 2).png>)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of `#!/bin/bash`?**

#!/bin/bash specifically instructs the system to use Bash to execute the file, rather than another shell like sh or zsh

---

**2. Why do we use `chmod +x` before running a script?**

By default, newly created files on Linux do not have execute permission for security reasons. chmod +x adds execute permission, which allows the file to be run directly as a program. Without it, the system will return a "Permission denied" error.

---

**3. What is the difference between running a script using `./script.sh` and `bash script.sh`?**

./script.sh runs the script as an executable using the interpreter defined in the shebang line (#!/bin/bash). It requires execute permission. bash script.sh explicitly calls Bash to interpret the file, bypassing the shebang line and not requiring execute permission — Bash reads it as a text file directly.

---

# Task 3 — Variables: User Information Script

## Goal

Use variables to store and display user-related information.

### Evidence

#### Screenshot 1 — Content of `user-info.sh`

![](<screenshots/Assignment 5/Screenshot 1{task 3}.png>)

---

#### Screenshot 2 — Output of `./user-info.sh`

![](<screenshots/Assignment 5/Screenshot 2(task 3).png>)

---

### Notes

Answer the following in your own words:

**1. What is a variable in Bash?**

A variable is a named container that stores a value — such as text, a number, or a file path that can be referenced and reused throughout a script. In Bash, variables are defined without a $ sign but accessed with one (e.g. NAME="Hope" to set, $NAME to use).

---

**2. Why should we avoid spaces around the `=` sign when creating variables?**

Bash is strict about syntax. If you write NAME = "Hope", Bash interprets NAME as a command and = as an argument, causing an error. The correct syntax NAME="Hope" with no spaces tells Bash this is a variable assignment, not a command.

---

**3. How do you access the value stored inside a Bash variable?**

By prefixing the variable name with a $ sign. For example, if you set NAME="Hope", you access it with $NAME or ${NAME}. The curly braces ${} are optional for simple variables but useful when combining a variable with other text, like ${NAME}s to avoid ambiguity.

---

# Task 4 — Arrays & Loops: Tools Checklist Script

## Goal

Use arrays and loops to print a checklist of tools used in Bash scripting.

### Evidence

#### Screenshot 1 — Content of `tools-checklist.sh`

![](<screenshots/Assignment 5/Screenshot 1(task4).png>)

---

#### Screenshot 2 — Output of `./tools-checklist.sh`

![](<screenshots/Assignment 5/Screenshot 2(task 4).png>)

---

### Notes

Answer the following in your own words:

**1. What is an array in Bash?**

An array is a variable that holds multiple values under a single name. Instead of creating separate variables for each item.

---

**2. Why are arrays useful in scripts?**

Arrays let you group related items together and process them all with a single loop, making scripts cleaner, shorter, and easier to maintain. For example, instead of writing a separate echo for each tool, you loop through the array and handle all of them in one block of code

---

**3. What does `"${tools[@]}"` mean?**

tools is the array name, [@] means "all elements," and the ${} syntax expands the variable. So "${tools[@]}" expands to every item in the array as a separate quoted string.

---

**4. What is the purpose of the `for` loop in this script?**

The for loop iterates through every item in the tools array one at a time, assigning each value to the variable tool, then executing the echo command for it. This means instead of writing 7 separate echo statements, one loop handles all 7 tools automatically.

---

# Task 5 — Loops: Number Counter Script

## Goal

Use loops to repeat a task multiple times.

### Evidence

#### Screenshot 1 — Content of `counter.sh`

![](<screenshots/Assignment 5/Screenshot 1(task 5).png>).

---

#### Screenshot 2 — Output of `./counter.sh`

![](<screenshots/Assignment 5/Screenshot 2(task 5).png>)

---

### Notes

Answer the following in your own words:

**1. What is a loop?**

A loop is a programming construct that repeats a block of commands multiple times until a condition is met or a set of items is exhausted. Instead of writing the same command over and over, a loop executes it automatically for each iteration.

---

**2. Why do we use loops in Bash scripting?**

Loops eliminate repetition and make scripts more efficient and scalable. For example, instead of writing echo "Count: 1", echo "Count: 2" ... echo "Count: 100" manually, a single loop handles all 100 iterations in 3 lines of code. They are essential for processing files, iterating over lists, and automating repetitive tasks..

---

**3. How many times did the loop run in your script?**

The loop ran 5 times, once for each number in the range {1..5}, printing Count: 1 through Count: 5.

---

**4. What would you change if you wanted the loop to run 10 times?**

Change {1..5} to {1..10} on the for line. The script would then count from 1 to 10, running the loop body 10 times instead of 5.

---

# Task 6 — Files & Conditionals: File Validation Script

## Goal

Use file checks and conditionals to verify whether files and directories exist.

### Evidence

#### Screenshot 1 — Output of `ls -lah ../test-folder`

![](<screenshots/Assignment 5/Screenshot 1(task 6).png>)

---

#### Screenshot 2 — Content of `file-check.sh`

![](<screenshots/Assignment 5/Screenshot 2(task 6).png>)

---

#### Screenshot 3 — Output of `./file-check.sh`

![](<screenshots/Assignment 5/Screenshot 3(task 6).png>)

---

### Notes

Answer the following in your own words:

**1. What does `-d` check in Bash?**

-d checks whether a given path exists and is a directory. In an if statement, [ -d "$DIR" ] returns true if the directory exists, and false if it doesn't exist or if the path points to a file instead of a directory.

---

**2. What does `-f` check in Bash?**

-f checks whether a given path exists and is a regular file. [ -f "$FILE" ] returns true only if the path exists and is a file (not a directory, not a symlink to a directory). It returns false if the file doesn't exist.

---

**3. Why should file and directory paths be stored in variables?**

Storing paths in variables at the top of the script makes them easy to update in one place without hunting through the entire script. It also makes the code more readable $FILE is clearer than a long hardcoded path repeated multiple times and reduces the risk of typos when the same path is used in multiple places.

---

**4. What happens if the file does not exist?**

The -f condition returns false, so the else block executes and prints ✘ File NOT FOUND with the path. The script doesn't crash, it handles the missing file gracefully and continues running. This is exactly how production scripts should behave: check first, then act, never assume

---

# Task 7 — Conditionals: Pass or Retry Script

## Goal

Use if-else conditionals to make decisions based on a variable value.

### Evidence

#### Screenshot 1 — Content of `score-check.sh` with `score=85`

![](<screenshots/Assignment 5/Screenshot 1(task 7).png>).

---

#### Screenshot 2 — Output showing `Result: Pass`

![](<screenshots/Assignment 5/Screenshot 2(task 7).png>).

---

#### Screenshot 3 — Content of `score-check.sh` with `score=55`

![](<screenshots/Assignment 5/Screenshot 3(task 7).png>)

---

#### Screenshot 4 — Output showing `Result: Retry`

![](<screenshots/Assignment 5/Screenshot 4(task 7).png>)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of if-else in Bash?**

if-else allows a script to make decisions based on conditions. The if block runs when a condition is true, and the else block runs when it is false.

---

**2. What does `-ge` mean?**

-ge stands for "greater than or equal to." It is a numeric comparison operator used inside Bash conditionals. For example, [ $score -ge 70 ] checks whether the value of score is greater than or equal to 70.

---

**3. Why should conditions be tested with different values?**

Testing with different values confirms that both the if and else branches work correctly. A script might appear to work when only tested with one value, but have a bug in the other branch that goes unnoticed. Testing with a passing score (85) and a failing score (55) proves the logic handles both cases as expected.

---

**4. How can conditionals help in automation scripts?**

Conditionals make automation scripts self-managing. For example, a DevOps script can check if a service is running before trying to restart it, verify a file exists before processing it, or check an exit code before continuing a deployment. Without conditionals, scripts run blindly and can fail silently or cause unintended damage with conditionals, they respond intelligently to real system states.

---

# Task 8 — Functions: Final Bash Automation Script

## Goal

Create a final Bash script using functions to organize reusable code.

### Evidence

#### Screenshot 1 — Content of `final-automation.sh`

![](<screenshots/Assignment 5/Screenshot 1(task 8).png>)

---

#### Screenshot 2 — Output of `./final-automation.sh`

![](<screenshots/Assignment 5/Screenshot 2(task 8).png>)

---

#### Screenshot 3 — Output of `ls -lah` showing all created scripts

![](<screenshots/Assignment 5/Screenshot 3(task 8).png>)

---

### Notes

Answer the following in your own words:

**1. What is a function in Bash?**

A function is a named block of reusable code defined once and called multiple times anywhere in the script. It groups related commands under a single name, so instead of repeating the same lines, you call the function name and it executes the block.

---

**2. Why are functions useful in scripts?**

Functions make scripts modular, readable, and maintainable. Each function handles one specific task, so the script is easy to understand at a glance. If you need to change how a task works, you update it in one function rather than finding and editing every place the code appears. They also make debugging easier since you can test each function independently.

---

**3. Which functions did you create in this script?**

Six functions were created: print_header (displays the script title and user info), check_system (prints hostname, user, and shell), tools_checklist (loops through a DevOps tools array), score_check (validates a score using a conditional), file_check (verifies a file exists and writes a log entry), and print_footer (displays a completion message).

---

**4. How does this final script combine variables, arrays, loops, conditionals, files, and functions?**

Variables store the name, cohort, date, and log file path used throughout. An array holds the list of DevOps tools, and a for loop iterates through it in tools_checklist. A conditional (-ge) checks the score in score_check. A file conditional (-f) checks whether the log file exists in file_check, then writes to it using >>. All of this logic is organized inside six functions that are called sequentially at the bottom under the main section making the script a complete demonstration of every Bash concept covered in this assignment.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/hope-odu_bash-scripting-automation-activity-7487998089279131648-BUAM?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFi9QKUB8YFTHgZLGCY97rJ9u0Stdj7th8Q`

---

#### Screenshot — Published LinkedIn post

![](<screenshots/Assignment 5/Screenshot of linkedin post.png>)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- All script files must be created and run successfully
- Required notes must be answered clearly for every task
- Do not expose sensitive information (keys, passwords, credentials)

---

# Completion Checklist

- [ ] Task 1: Environment setup verified, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: First script created, executed, permissions verified (Screenshots 1–3, Notes answered)
- [ ] Task 3: Variables script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 4: Arrays and loops script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 5: Counter loop script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 6: File validation script created and run (Screenshots 1–3, Notes answered)
- [ ] Task 7: Pass/Retry conditional script tested with both values (Screenshots 1–4, Notes answered)
- [ ] Task 8: Final automation script created and run (Screenshots 1–3, Notes answered)
- [ ] All scripts run without errors
- [ ] Full Name visible in all required screenshots
- [ ] LinkedIn post published and URL submitted
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