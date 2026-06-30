# The Missing Release Symlink

## Scenario

A deployment left `/opt/app/current` pointing at the wrong release, and the application health file is missing.

The application expects `/opt/app/current` to resolve to the correct release directory, and the health file must exist with the correct content.

## Requirement

Repair the deployment symlink so that:

```text
/opt/app/current resolves to the expected release
/opt/app/current/health.txt exists
health.txt contains OK
```

## Initial State

The `/opt/app/current` symlink was pointing to the wrong release directory.

Because of this, the application health file was missing from the active release path.

## Troubleshooting Path

```text
Inspect /opt/app
↓
Check where /opt/app/current points
↓
Inspect available release directories
↓
Find the release that should contain health.txt
↓
Repoint /opt/app/current to the correct release
↓
Create or fix health.txt
↓
Validate symlink target and health file content
```

## Verification Before Fix

Inspect the application directory:

```bash
ls -la /opt/app
```

Check whether `/opt/app/current` is a symlink and where it points:

```bash
ls -l /opt/app/current
```

Resolve the final symlink target:

```bash
readlink -f /opt/app/current
```

List available release directories:

```bash
ls -la /opt/app/releases
```

Search for an existing health file inside the release directories:

```bash
find /opt/app/releases -maxdepth 2 -type f -name "health.txt" -exec ls -l {} \;
```

Check the content of any discovered health file:

```bash
find /opt/app/releases -maxdepth 2 -type f -name "health.txt" -exec sh -c 'echo "---- $1"; cat "$1"' _ {} \;
```

## First Finding

```text
/opt/app/current was a symlink, but it pointed to the wrong release directory.
The expected active release was under /opt/app/releases.
The health file needed to exist at /opt/app/current/health.txt and contain OK.
```

## Fix

Repoint `/opt/app/current` to the correct release directory.

Replace `CORRECT_RELEASE` with the release directory identified during inspection:

```bash
ln -sfn /opt/app/releases/CORRECT_RELEASE /opt/app/current
```

Create or repair the health file through the active symlink path:

```bash
echo OK > /opt/app/current/health.txt
```

If root permission is required, use:

```bash
echo OK | sudo tee /opt/app/current/health.txt
```

## Validation

Confirm `/opt/app/current` points to the correct release:

```bash
ls -l /opt/app/current
```

Resolve the final target:

```bash
readlink -f /opt/app/current
```

Confirm the health file exists:

```bash
ls -l /opt/app/current/health.txt
```

Confirm the health file content:

```bash
cat /opt/app/current/health.txt
```

Expected result:

```text
OK
```

Confirm the real file path through the symlink:

```bash
readlink -f /opt/app/current/health.txt
```

Expected result:

```text
/opt/app/releases/CORRECT_RELEASE/health.txt
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- A symlink is a pointer to another file or directory.
- Deployment systems often use a stable path like `/opt/app/current` to point to the active release.
- `ls -l <symlink>` shows where a symlink points.
- `readlink -f <symlink>` resolves the final real path.
- `ln -sfn <target> <link>` safely repoints an existing symlink.
- Always identify the correct release before changing a deployment symlink.
- Validation should confirm both the symlink target and the required file content.

## Knowledge Check

### Question 1

What command shows the final resolved path of `/opt/app/current`?

A. `pwd /opt/app/current`  
B. `readlink -f /opt/app/current`  
C. `touch /opt/app/current`  
D. `chmod +x /opt/app/current`  

**Answer:** B

### Question 2

Which command safely repoints `/opt/app/current` to a correct release directory?

A. `rm -rf /opt/app/current`  
B. `ln -sfn /opt/app/releases/CORRECT_RELEASE /opt/app/current`  
C. `cat /opt/app/current`  
D. `mkdir /opt/app/current`  

**Answer:** B
