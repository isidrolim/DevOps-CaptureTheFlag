# SET-000 – Local Lab Readiness

## Scenario

The local DevOps CTF platform must confirm that the lab environment is ready before starting the real troubleshooting challenges.

This challenge walks through the full CTF loop:

```text
Start a server
Use the browser terminal
Run a readiness script
Check the solution
Read the generated flag
Submit the flag
```

## Requirement

Complete the local lab readiness check.

The platform must confirm that:

```text
Terminal readiness marker exists
Local preview service returns OK
Platform-injected local AWS defaults are present for future AWS challenges
Generated flag can be read and submitted after the check passes
```

## Initial State

The challenge server was started from the platform UI.

The browser terminal connected successfully to the managed challenge container, but the readiness marker still needed to be created by running the provided readiness script.

## Troubleshooting Path

```text
Start challenge server
↓
Confirm browser terminal access
↓
Inspect readiness script
↓
Run readiness script
↓
Verify readiness marker
↓
Verify local health check
↓
Run Check Solution
↓
Read and submit generated flag
```

## Verification Before Fix

Confirm that the terminal is connected and commands can run:

```bash
whoami
hostname
pwd
```

Inspect the provided readiness script:

```bash
ls -l /opt/challenge/scripts/ready.sh
cat /opt/challenge/scripts/ready.sh
```

Check whether the readiness marker already exists:

```bash
ls -l /tmp/ksg-lab-ready/terminal-ok
```

Check the readiness health file:

```bash
ls -l /srv/readiness/health
```

## First Finding

```text
The challenge container was running.
The browser terminal was connected.
The readiness marker still needed to be created.
```

## Fix

Run the provided readiness script:

```bash
/opt/challenge/scripts/ready.sh
```

The script prepares the local challenge environment and writes the readiness marker required by the platform validation.

## Validation

Confirm the readiness marker exists:

```bash
ls -l /tmp/ksg-lab-ready/terminal-ok
```

Confirm the local readiness health file exists:

```bash
ls -l /srv/readiness/health
```

Read the health file:

```bash
cat /srv/readiness/health
```

Expected result:

```text
OK
```

After the local checks pass, click **Check Solution** in the challenge UI.

Then read the generated flag:

```bash
cat /srv/readiness/flag
```

If the flag location is not obvious, search for it:

```bash
find / -iname "*flag*" 2>/dev/null
```

Expected result:

```text
The readiness marker exists, the health check returns OK, and the generated flag can be read and submitted.
```

## Lessons Learned

- A readiness challenge verifies that the lab platform itself is working.
- The browser terminal must be able to run commands inside the managed challenge container.
- Provided challenge scripts should be inspected before execution when possible.
- Validation should match the challenge requirement instead of assuming the environment is ready.
- The flag confirms that the platform check passed successfully.
- `find / -iname "*flag*" 2>/dev/null` is useful when the flag location is not obvious.

## Knowledge Check

### Question 1

What is the main purpose of the Local Lab Readiness challenge?

A. To install Docker manually  
B. To verify that the challenge platform, terminal, health check, and flag flow work  
C. To configure Nginx on port 8080  
D. To rotate application logs  

**Answer:** B

### Question 2

Which command runs the provided readiness script?

A. `systemctl start readiness`  
B. `/opt/challenge/scripts/ready.sh`  
C. `chmod 777 /srv/readiness/health`  
D. `docker compose down`  

**Answer:** B
