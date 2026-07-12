# Local AWS Emulator Setup

## Scenario

Prepare the local development environment for AWS-related challenges by starting the local AWS emulator (Floci) and verifying that challenge containers can communicate with it.

The challenge does not require connecting to the real AWS cloud.

Instead, it uses a local AWS-compatible emulator.

## Requirement

Prepare the local environment so that:

```text
Floci is running
Challenge containers can reach Floci
AWS-compatible services report healthy
Default region is ap-southeast-1
```

## Initial State

The challenge container was running.

However, the local AWS emulator (Floci) was not running from the host machine.

Because of this, future AWS-based challenges would not be able to communicate with the expected AWS-compatible endpoint.

## Troubleshooting Path

```text
Understand the challenge architecture
↓
Identify where Floci should run
↓
Locate Docker Compose instructions
↓
Start Floci from the host
↓
Verify Floci container is running
↓
Run challenge validation
↓
Read generated flag
```

## Understanding the Architecture

Floci is a local AWS service emulator.

Instead of communicating with the real AWS cloud:

```text
Application
↓

AWS Cloud
```

the challenge environment communicates with:

```text
Application

↓

Floci

↓

Fake AWS Services
```

Examples include:

```text
Fake S3
Fake SQS
Fake Secrets Manager
Fake DynamoDB
```

The application does not know the difference.

As long as the API behaves like AWS, the application functions normally.

This allows developers to:

- Develop offline
- Test safely
- Avoid AWS costs
- Prevent accidental changes to production AWS resources

## Verification Before Fix

Read the challenge instructions:

```text
Start Floci from your HOST terminal.
```

This is important because Floci is **not** installed inside the challenge container.

Attempts such as:

```bash
floci
```

returned:

```text
command not found
```

Attempts such as:

```bash
systemctl status floci
```

also failed because Floci is not a Linux system service.

Instead, it is managed through Docker Compose.

## Systematic Elimination

```text
✓ Challenge container is running
✓ Docker environment is available
✓ Floci is managed by Docker Compose
✗ Floci emulator is not running
```

The first failed verification was:

```text
The required AWS emulator service was not started on the Docker host.
```

## First Finding

```text
Floci is not installed inside the challenge container.

It is a Docker Compose service running on the Rocky Linux host.

Challenge containers communicate with Floci over the Docker network.
```

## Fix

From the DevOps CTF repository root on the Rocky Linux host:

```bash
docker compose --profile aws up -d floci
```

Verify the service:

```bash
docker compose ps
```

or

```bash
docker ps | grep floci
```

Expected result:

```text
Floci container is running.
```

## Validation

Return to the challenge page.

Click:

```text
Check Solution
```

Expected result:

```text
Floci is reachable.

AWS-compatible services are healthy.

Default region is ap-southeast-1.
```

Read the generated flag:

```bash
cat /home/devops/flag.txt
```

After validation passes, submit the generated flag.

## Lessons Learned

- Floci is a local AWS emulator.
- Development environments often emulate cloud services instead of using real cloud infrastructure.
- Floci is managed by Docker Compose, not by systemd.
- The challenge container consumes the emulator service but does not host it.
- Always identify whether a dependency belongs inside the container or on the host before attempting to start it.
- Read the challenge hints carefully—they often identify the correct execution context.

## Engineering Insight

One of the most important DevOps concepts is separating:

```text
Application Runtime

from

Development Infrastructure
```

The application container was healthy.

The missing dependency was external:

```text
Challenge Container

↓

Floci

↓

AWS-compatible APIs
```

This is identical to many production systems:

```text
Application

↓

Database

↓

Message Queue

↓

Cloud APIs
```

The application may be healthy while one of its external dependencies is unavailable.

Another important lesson is understanding **execution context**.

This challenge specifically instructed:

```text
Run Floci from the HOST terminal.
```

Understanding where a component belongs is just as important as knowing how to start it.

Good engineers first determine:

- What component is failing?
- Where does it run?
- Who manages it?

Only then do they execute the appropriate fix.

## Knowledge Check

### Question 1

Why was Floci started from the Rocky Linux host instead of inside the challenge container?

A. Because Floci is managed as a Docker Compose service on the host.

B. Because the challenge container cannot run Linux commands.

C. Because Docker cannot run inside Linux.

D. Because AWS requires root access.

**Answer:** A

### Question 2

What is Floci's primary purpose in this challenge?

A. To replace Node.js.

B. To emulate AWS services locally for development and testing.

C. To replace Docker.

D. To monitor system performance.

**Answer:** B
