# Permissions Drift

## Scenario

A service user cannot read its configuration file after ownership and file permission bits drifted.

The application depends on a config file located at:

```text
/srv/app/config.env
```

The service user must be able to read this file, but the file must not be world-writable.

## Requirement

Repair the ownership and permissions so that:

```text
appuser can read /srv/app/config.env
/srv/app/config.env is not world-writable
```

## Initial State

The service user `appuser` could not read `/srv/app/config.env`.

The file ownership and mode bits were not in the expected state, causing a permission problem for the application.

## Troubleshooting Path

```text
Confirm config file exists
↓
Check file owner, group, and permissions
↓
Confirm appuser exists
↓
Test whether appuser can read the file
↓
Fix ownership and permissions
↓
Validate appuser can read the file
↓
Validate the file is not world-writable
```

## Verification Before Fix

Confirm the config file exists:

```bash
ls -l /srv/app/config.env
```

Check the service user exists:

```bash
id appuser
```

Test whether `appuser` can read the config file:

```bash
sudo -u appuser cat /srv/app/config.env
```

If permissions are broken, this may return:

```text
Permission denied
```

Check detailed file metadata:

```bash
stat -c '%A %a %U %G %n' /srv/app/config.env
```

## First Finding

```text
The config file existed, but appuser could not read it.
The file ownership and permissions needed to be corrected.
The file also needed to remain not world-writable.
```

## Fix

Set the file owner and group to `appuser`:

```bash
sudo chown appuser:appuser /srv/app/config.env
```

Set secure file permissions:

```bash
sudo chmod 640 /srv/app/config.env
```

This results in:

```text
Owner: appuser
Group: appuser
Mode: 640
```

Permission meaning:

```text
owner  = rw-  read and write
group  = r--  read only
others = ---  no access
```

## Validation

Check the final ownership and permissions:

```bash
ls -l /srv/app/config.env
```

Expected result:

```text
-rw-r----- 1 appuser appuser ... /srv/app/config.env
```

Confirm using `stat`:

```bash
stat -c '%A %a %U %G %n' /srv/app/config.env
```

Expected result:

```text
-rw-r----- 640 appuser appuser /srv/app/config.env
```

Confirm `appuser` can read the file:

```bash
sudo -u appuser cat /srv/app/config.env
```

Expected result:

```text
The file contents print successfully without Permission denied.
```

Confirm the file is not world-writable:

```bash
stat -c '%a' /srv/app/config.env
```

Expected result:

```text
640
```

The last digit controls permissions for `others`.

In this case:

```text
0 = no permission for others
```

So the file is not world-writable.

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- Linux file access is controlled by ownership, group membership, and permission bits.
- Ownership decides which permission column Linux checks.
- Permissions decide what actions are allowed.
- Linux checks permissions in this order: owner, group, others.
- `ls -l` shows file permissions, owner, and group.
- `id <user>` confirms whether a user exists and which groups they belong to.
- `sudo -u <user> cat <file>` is useful for testing file access as a specific user.
- `chmod 640` allows the owner to read/write, the group to read, and others to have no access.
- Avoid using `chmod 777` because it gives unnecessary and unsafe access.
- A config file usually needs read permission, not execute permission.

## Knowledge Check

### Question 1

When troubleshooting a file permission issue, what should you check first?

A. Whether the file owner, group, and permissions allow the required access  
B. Whether the server can access the internet  
C. Whether the file can be deleted  
D. Whether the user has a password  

**Answer:** A

### Question 2

What does permission mode `640` mean for a regular file?

A. Owner can read/write, group can read, others have no access  
B. Everyone can read/write/execute  
C. Owner can execute only, group can write only, others can read  
D. Owner has no access, group has no access, others can write  

**Answer:** A
