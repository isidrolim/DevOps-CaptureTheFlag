# The Runaway Logger

## Scenario

A forgotten process is continuously writing to `/var/log/bad.log` and consuming disk space.

The log file must remain available as evidence, but the process writing to it must be stopped.

## Requirement

Stop the runaway writer process while keeping the log file intact.

The final state must be:

```text
/var/log/bad.log exists
The writer process is gone
The log size remains stable during the check interval
```

## Initial State

The file `/var/log/bad.log` existed and was continuously growing.

A background process was actively writing new lines to the log file.

## Troubleshooting Path

```text
Confirm the log file exists
↓
Check whether the log file is growing
↓
Identify which process has the log file open
↓
Inspect the process before stopping it
↓
Stop the writer process
↓
Verify the process is gone
↓
Verify the log file still exists
↓
Verify the log file size remains stable
```

## Verification Before Fix

Confirm the log file exists:

```bash
ls -lh /var/log/bad.log
```

Check whether the file size is increasing:

```bash
ls -lh /var/log/bad.log
sleep 5
ls -lh /var/log/bad.log
```

If the size increases, something is still writing to the file.

Watch the log content if needed:

```bash
tail -f /var/log/bad.log
```

Stop `tail -f` with:

```text
Ctrl+C
```

Identify which process has the file open:

```bash
sudo lsof /var/log/bad.log
```

Example output:

```text
COMMAND     PID USER   FD   TYPE NAME
badlogger  1234 root   3w   REG  /var/log/bad.log
```

The important fields are:

```text
PID = process ID
FD  = file descriptor
w   = opened for writing
```

Inspect the process before stopping it:

```bash
ps -fp <PID>
```

Optional process search:

```bash
pgrep -a -f bad
pgrep -a -f log
```

## First Finding

```text
/var/log/bad.log existed and was growing.
A running process had /var/log/bad.log open for writing.
The log growth was caused by the writer process, not by the file itself.
```

## Fix

Stop the writer process gracefully first:

```bash
sudo kill <PID>
```

Replace `<PID>` with the process ID found from `lsof`.

If the process does not stop after a normal kill, use SIGKILL as a last resort:

```bash
sudo kill -9 <PID>
```

## Validation

Verify the process is gone:

```bash
ps -p <PID>
```

Expected result:

```text
Only the header appears, or no process is listed.
```

Verify no process still has the log file open for writing:

```bash
sudo lsof /var/log/bad.log
```

Expected result:

```text
No writer process is listed.
```

Verify the log file still exists:

```bash
ls -l /var/log/bad.log
```

Verify the log size remains stable:

```bash
ls -lh /var/log/bad.log
sleep 10
ls -lh /var/log/bad.log
```

Expected result:

```text
The file size stays the same between checks.
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- Do not delete logs blindly when troubleshooting disk growth.
- A growing log file is usually a symptom, not the root cause.
- The first troubleshooting question should be: “Who is writing to this file?”
- `lsof <file>` shows which process has a file open.
- `ps -fp <PID>` helps inspect a process before stopping it.
- Use normal `kill <PID>` first to allow graceful shutdown.
- Use `kill -9 <PID>` only when the process does not stop normally.
- Always validate that the log file still exists after stopping the writer.
- Always confirm the file size remains stable after the fix.

## Knowledge Check

### Question 1

When a log file is growing unexpectedly, what should you identify first?

A. Which process is writing to the file  
B. Whether the file can be deleted  
C. Whether the server can be rebooted  
D. Whether the file permissions can be changed to `777`  

**Answer:** A

### Question 2

Which command shows which process has `/var/log/bad.log` open?

A. `df -h /var/log/bad.log`  
B. `lsof /var/log/bad.log`  
C. `chmod /var/log/bad.log`  
D. `pwd /var/log/bad.log`  

**Answer:** B
