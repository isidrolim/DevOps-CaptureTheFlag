# Logrotate Misfire

## Scenario

Application logs are growing without proper rotation because the logrotate configuration points to the wrong log file.

The application writes to:

```text
/var/log/app/app.log
```

But logrotate was configured to rotate a different file.

## Requirement

Repair the logrotate configuration so that:

```text
A rotated file exists
The current log is below the configured threshold after rotation
The application still writes to the current log
```

## Initial State

The real application log file existed and was growing:

```text
/var/log/app/app.log
```

However, the logrotate configuration was pointing to the wrong file:

```text
/var/log/app/wrong.log
```

Because of this, logrotate was not rotating the real application log.

## Troubleshooting Path

```text
Confirm actual application log exists
↓
Inspect logrotate configuration
↓
Compare configured log path with real log path
↓
Identify wrong log path
↓
Fix logrotate configuration
↓
Run logrotate in debug mode
↓
Force log rotation
↓
Validate rotated file exists
↓
Validate current log is small
↓
Validate app still writes to current log
```

## Verification Before Fix

List the application log directory:

```bash
ls -l /var/log/app/
```

Check the real application log:

```bash
ls -lh /var/log/app/app.log
```

Inspect the logrotate configuration:

```bash
cat /etc/logrotate.d/app
```

Broken configuration:

```text
/var/log/app/wrong.log {
    size 1k
    rotate 2
    missingok
    notifempty
    copytruncate
}
```

Compare the configured path with the real log path:

```text
Configured path: /var/log/app/wrong.log
Actual log path: /var/log/app/app.log
```

Run logrotate in debug mode to see what it would rotate:

```bash
logrotate -d /etc/logrotate.d/app
```

Before the fix, logrotate would check the wrong file path.

## First Finding

```text
The application was writing to /var/log/app/app.log.
logrotate was configured to rotate /var/log/app/wrong.log.
The rotation rule was valid, but it was applied to the wrong file.
```

The first failed check was:

```text
logrotate configuration did not point to the real application log.
```

## Fix

Update the logrotate configuration so it points to the correct log file:

```bash
sed -i 's|/var/log/app/wrong.log|/var/log/app/app.log|' /etc/logrotate.d/app
```

Verify the corrected configuration:

```bash
cat /etc/logrotate.d/app
```

Expected configuration:

```text
/var/log/app/app.log {
    size 1k
    rotate 2
    missingok
    notifempty
    copytruncate
}
```

Run logrotate in debug mode:

```bash
logrotate -d /etc/logrotate.d/app
```

Expected signs in the debug output:

```text
rotating pattern: /var/log/app/app.log 1024 bytes
considering log /var/log/app/app.log
log needs rotating
copying /var/log/app/app.log to /var/log/app/app.log.1
truncating /var/log/app/app.log
```

Force logrotate to apply the corrected rule immediately:

```bash
logrotate -f /etc/logrotate.d/app
```

## Validation

Confirm the rotated file exists:

```bash
ls -lah /var/log/app/
```

Expected files:

```text
app.log
app.log.1
```

Confirm the current log is small after rotation:

```bash
ls -lah /var/log/app/app.log /var/log/app/app.log.1
```

Expected result:

```text
app.log    small or empty after rotation
app.log.1  rotated copy of the previous log
```

Confirm the application still writes to the current log:

```bash
ls -lh /var/log/app/app.log
sleep 5
ls -lh /var/log/app/app.log
```

If the size increases, the application is still writing to the current log.

Check recent log lines:

```bash
tail -n 5 /var/log/app/app.log
```

Expected result:

```text
The current app.log receives new log entries after rotation.
```

If the application writes quickly, run logrotate again and check the solution immediately:

```bash
logrotate -f /etc/logrotate.d/app && ls -lah /var/log/app/app.log /var/log/app/app.log.1
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- `logrotate` manages log growth by rotating, truncating, compressing, or removing old logs.
- A correct logrotate rule is useless if it points to the wrong file.
- Always compare the actual log path with the path configured in `/etc/logrotate.d/`.
- `logrotate -d <config>` runs in debug mode and shows what would happen without making changes.
- `logrotate -f <config>` forces rotation immediately.
- `copytruncate` copies the current log to a rotated file and then truncates the original log in place.
- `copytruncate` is useful when an application keeps writing to the same open log file.
- The correct fix was to repair the wrong path, not to change the `size 1k` threshold.
- Validation must confirm that the rotated file exists, the current log is small, and the app still writes to the current log.

## Knowledge Check

### Question 1

What was the root cause of the Logrotate Misfire challenge?

A. The `size 1k` threshold was too small  
B. logrotate was pointing to `/var/log/app/wrong.log` instead of `/var/log/app/app.log`  
C. The application was not running  
D. The `/var/log/app` directory did not exist  

**Answer:** B

### Question 2

Which command forces logrotate to run using the corrected app configuration?

A. `logrotate -f /etc/logrotate.d/app`  
B. `systemctl restart logrotate`  
C. `chmod +x /var/log/app/app.log`  
D. `tail -f /etc/logrotate.d/app`  

**Answer:** A
