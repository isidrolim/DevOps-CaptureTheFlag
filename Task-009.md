# Disk Full from Old Deploy Artifacts

## Scenario

Disk usage is high because old application releases and archived logs have accumulated.

The application needs enough free space to write temporary files, but cleanup must be done safely so the current release is not deleted.

Important paths:

```text
/opt/app/releases
/opt/app/current
/var/log/app
```

## Requirement

Free disk space safely so that:

```text
Free space threshold is met
The current release still exists
The application health endpoint returns 200 OK
```

## Initial State

Old deployment artifacts existed under:

```text
/opt/app/releases
```

Archived application logs existed under:

```text
/var/log/app/archive
```

The active application release was controlled by the symlink:

```text
/opt/app/current
```

Before deleting anything, the current release needed to be identified to avoid removing the active application version.

## Troubleshooting Path

```text
Check filesystem usage
↓
Identify large application directories
↓
Inspect old releases
↓
Identify current release symlink target
↓
Confirm which releases are safe to remove
↓
Remove only old releases and archived logs
↓
Validate current release still exists
↓
Validate application health endpoint
```

## Verification Before Fix

Check filesystem usage:

```bash
df -h
df -h /
```

Check application directory usage:

```bash
du -sh /opt/app/releases /var/log/app /opt/app/current 2>/dev/null
```

Check release directory usage:

```bash
du -h --max-depth=1 /opt/app/releases 2>/dev/null | sort -h
```

Check application log directory usage:

```bash
du -h --max-depth=1 /var/log/app 2>/dev/null | sort -h
```

The large cleanup candidates were:

```text
/opt/app/releases/2026-04-01
/opt/app/releases/2026-05-01
/var/log/app/archive
```

Before removing old releases, identify the current release:

```bash
ls -l /opt/app/current
readlink -f /opt/app/current
ls -la /opt/app/releases
```

Result:

```text
/opt/app/current -> /opt/app/releases/2026-06-12
```

## First Finding

```text
The current active release was /opt/app/releases/2026-06-12.
The old releases were /opt/app/releases/2026-04-01 and /opt/app/releases/2026-05-01.
The archived logs were stored under /var/log/app/archive.
```

The first important safety finding was:

```text
Do not delete /opt/app/current or /opt/app/releases/2026-06-12.
```

Only old releases and archived logs should be removed.

## Fix

Remove the old release directories:

```bash
rm -rf /opt/app/releases/2026-04-01 /opt/app/releases/2026-05-01
```

Remove archived logs:

```bash
rm -rf /var/log/app/archive
```

Do not remove:

```text
/opt/app/current
/opt/app/releases/2026-06-12
```

## Validation

Confirm the current release symlink still points to the active release:

```bash
readlink -f /opt/app/current
```

Expected result:

```text
/opt/app/releases/2026-06-12
```

Confirm the current release still exists:

```bash
ls -lah /opt/app/releases/
```

Expected result:

```text
2026-06-12
```

Confirm old releases were removed:

```bash
ls -lah /opt/app/releases/
```

Expected result:

```text
2026-04-01 and 2026-05-01 are no longer present.
```

Confirm archived logs were removed:

```bash
du -sh /var/log/app 2>/dev/null
ls -lah /var/log/app
```

Check filesystem usage:

```bash
df -h /
```

If the percentage does not visibly change, check in MB:

```bash
df -m /
```

Test the application health endpoint:

```bash
curl -i http://localhost:8080/health
```

Expected result:

```text
HTTP/1.0 200 OK

OK
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- Use `df` to check filesystem usage.
- Use `du` to identify which directories are consuming space.
- `df` shows usage for the entire filesystem.
- `du` shows usage for specific files or directories.
- A small cleanup may not visibly change the rounded `df -h` percentage on a large filesystem.
- Never delete deployment releases before identifying the active release.
- `/opt/app/current` is a symlink that points to the active release.
- `readlink -f /opt/app/current` confirms the real active release path.
- Safe cleanup means removing old artifacts while preserving the current release.
- Always validate application health after cleanup.

## Knowledge Check

### Question 1

Before deleting old application releases, what should you verify first?

A. Which release `/opt/app/current` points to  
B. Whether `/tmp` is empty  
C. Whether the server hostname changed  
D. Whether Nginx logs can be deleted  

**Answer:** A

### Question 2

Which command shows the real directory that `/opt/app/current` points to?

A. `df -h /opt/app/current`  
B. `readlink -f /opt/app/current`  
C. `rm -rf /opt/app/current`  
D. `chmod +x /opt/app/current`  

**Answer:** B
