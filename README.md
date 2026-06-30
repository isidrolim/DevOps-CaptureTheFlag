# DevOps CTF – Learn by Fixing

This folder documents my hands-on troubleshooting practice from a local DevOps CTF-style training platform.

CTF means **Capture The Flag**. In this context, each challenge provides a broken Linux, application, Docker, logging, or deployment environment. The goal is to investigate the issue, repair the system, run the validation check, and capture the generated flag.

This is not just a collection of commands or answers. The purpose of this documentation is to build and reinforce a systematic troubleshooting mindset that can be applied to real DevOps, Infrastructure, CI/CD, and production support work.

---

## Why I Am Doing This

I currently come from an Infrastructure and Operations background, and I am building the skills needed to transition into DevOps.

These challenges reflect problems that commonly happen in real environments, including:

* Broken application releases
* Incorrect symlinks
* Permission drift
* Runaway processes
* Cron and scheduled job failures
* Port conflicts
* Nginx and application health issues
* Log rotation failures
* HTTP 500 errors caused by config issues
* Disk pressure from old deployment artifacts
* CI/CD-style release and runtime problems

My goal is to practice not only how to fix these issues, but how to investigate them properly.

---

## Troubleshooting Framework

For every challenge, I follow the same process:

1. **Identify the symptom**
   Understand what is actually broken.

2. **Build the dependency path**
   Identify what components must work for the system to be healthy.

3. **Verify each layer**
   Check one layer at a time using evidence.

4. **Find the first failure**
   Stop when the first failed check is found.

5. **Fix the first failure**
   Apply the smallest safe fix needed.

6. **Validate the result**
   Confirm the fix using commands or checks that match the original requirement.

---

## Repository Structure

```text
DevOps-CTF/
├── README.md
├── Set-000-Foundations/
├── Logs/
├── Linux/
├── Docker/
├── CI-CD/
└── AWS-Emulation/
```

The structure may grow as more challenges are completed.

---

## Documentation Format

Each challenge write-up uses this format:

```text
Challenge:
Scenario:
Symptom:
Dependency Path:
Verification:
First Failure:
Fix:
Validation:
Lessons Learned:
Knowledge Check:
```

This helps me practice structured troubleshooting instead of memorizing random commands.

---

## Learning Principle

The goal is not to ask:

```text
What command fixes this?
```

The goal is to ask:

```text
What is the symptom?
What must be true for this to work?
What should I verify next?
Where is the first failure?
How do I prove the fix worked?
```

This is the troubleshooting mindset I want to build for Linux, DevOps, CI/CD, Docker, application support, and production operations.
