# The Cron Job That Never Runs

## Scenario

A health report should be generated every minute, but cron never writes the output file.

The health report script should create:

```text
/var/reports/health.txt
```

The challenge requires the file to exist with a fresh timestamp, and cron must be running.

## Requirement

Repair the cron job so that:

```text
/var/reports/health.txt exists
/var/reports/health.txt has a fresh timestamp
cron is running
```

## Initial State

Cron was running inside the challenge container, but the scheduled health report was not being generated.

The cron definition existed in:

```text
/etc/cron.d/health-report
```

The script existed in:

```text
/usr/local/bin/write-health-report
```

However, the script did not have execute permission, so cron could not run it.

## Troubleshooting Path

```text
Check whether cron is running
↓
Inspect the cron definition
↓
Verify cron syntax and schedule
↓
Inspect the health report script
↓
Verify the script shebang and logic
↓
Check script permissions
↓
Run the script manually
↓
Fix the first failure
↓
Validate output file creation
↓
Validate cron updates the file automatically
```

## Verification Before Fix

Check the process running as PID 1:

```bash
ps -p 1 -f
```

In this challenge container, cron was running as PID 1:

```text
/usr/sbin/cron -f
```

Check for cron processes:

```bash
ps aux | grep cron
```

Inspect the cron job definition:

```bash
cat /etc/cron.d/health-report
```

Expected cron entry:

```text
* * * * * root /usr/local/bin/write-health-report >> /var/log/health-report-cron.log 2>&1
```

Meaning:

```text
* * * * *    Run every minute
root         Run as the root user
command      Run /usr/local/bin/write-health-report
>>           Append output to the log file
2>&1         Send errors to the same log file
```

Inspect the script:

```bash
cat /usr/local/bin/write-health-report
```

The script content was valid:

```bash
#!/usr/bin/env bash
set -euo pipefail

mkdir -p /var/reports

printf 'OK health-report generated_at=%s\n' "$(date -Iseconds)" > /var/reports/health.txt
```

Check the script permissions:

```bash
ls -l /usr/local/bin/write-health-report
```

Broken permission state:

```text
-rw-r--r-- 1 root root ... /usr/local/bin/write-health-report
```

## First Finding

```text
Cron was running.
The cron job existed.
The cron schedule and user field were valid.
The script content and shebang were valid.
The script was not executable.
```

The first failed check was:

```text
/usr/local/bin/write-health-report did not have execute permission.
```

Because the cron job calls the script directly, Linux requires the script to have the execute bit set.

## Fix

Add execute permission to the script:

```bash
chmod +x /usr/local/bin/write-health-report
```

Validate the permission changed:

```bash
ls -l /usr/local/bin/write-health-report
```

Expected result:

```text
-rwxr-xr-x 1 root root ... /usr/local/bin/write-health-report
```

## Validation

Run the script manually first:

```bash
/usr/local/bin/write-health-report
```

Check that the health report was created:

```bash
cat /var/reports/health.txt
```

Expected result:

```text
OK health-report generated_at=...
```

Check the file timestamp:

```bash
stat /var/reports/health.txt
```

Wait for cron to run again:

```bash
sleep 70
```

Check the timestamp again:

```bash
stat /var/reports/health.txt
```

The modification time should become newer, proving cron is running the script automatically.

Optional validation:

```bash
ls -l /var/reports/health.txt
cat /var/reports/health.txt
```

Expected result:

```text
/var/reports/health.txt exists and contains a fresh health report.
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- Cron problems should be troubleshot one layer at a time.
- Do not assume cron is broken before checking the script.
- In containers, `systemctl` may not work because systemd is not PID 1.
- Use `ps aux | grep cron` or `ps -p 1 -f` to check cron in a container.
- Files in `/etc/cron.d/` require a user field after the schedule.
- A valid script still will not run if it does not have execute permission.
- `chmod +x <script>` adds execute permission.
- Always run the script manually before blaming cron.
- The best validation is a fresh output file timestamp after cron runs.

## Knowledge Check

### Question 1

When a cron job does not create its expected output file, what should you verify before changing the cron schedule?

A. Whether the script exists, is valid, and is executable  
B. Whether `/tmp` is empty  
C. Whether the server has a GUI installed  
D. Whether the root password is expired  

**Answer:** A

### Question 2

Which command fixes a script that exists but cannot be executed by cron because it is missing the execute bit?

A. `chmod +x /usr/local/bin/write-health-report`  
B. `cat /etc/cron.d/health-report`  
C. `mkdir /var/reports`  
D. `ps aux | grep cron`  

**Answer:** A
