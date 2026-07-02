# Port 8080 Is Taken

## Scenario

Nginx should serve the application on port `8080`, but another process is already occupying the port.

The application health endpoint should be available at:

```text
http://localhost:8080/health
```

## Requirement

Repair the environment so that:

```text
Nginx owns port 8080
curl http://localhost:8080/health returns 200 OK
```

## Initial State

Port `8080` was already in use, but it was not owned by Nginx.

A different process was listening on port `8080`, preventing Nginx from serving the application.

## Troubleshooting Path

```text
Check who owns port 8080
↓
Confirm whether the listener is Nginx
↓
Inspect the process using the port
↓
Stop the wrong process
↓
Verify port 8080 is free
↓
Check whether Nginx is running
↓
Test Nginx configuration
↓
Start Nginx
↓
Verify Nginx owns port 8080
↓
Validate /health returns 200 OK
```

## Verification Before Fix

Check which process is listening on port `8080`:

```bash
ss -tulpn | grep :8080
```

Initial result showed:

```text
0.0.0.0:8080 users:(("python3",pid=7,fd=3))
```

This confirmed that port `8080` was in use by a Python process, not Nginx.

Inspect the process:

```bash
ps -fp 7
```

Result:

```text
UID   PID  PPID  CMD
root    7     1  python3 /usr/local/bin/port-blocker
```

Check whether Nginx was running:

```bash
ps aux | grep nginx
```

Initial result showed only the `grep` process, meaning Nginx was not running.

## First Finding

```text
Port 8080 was occupied by python3 /usr/local/bin/port-blocker.
The process using port 8080 was not the expected Nginx process.
Nginx was not running.
```

The first failed check was:

```text
Port 8080 was owned by the wrong process.
```

## Fix

Stop the wrong process:

```bash
kill 7
```

Verify port `8080` is no longer occupied:

```bash
ss -tulpn | grep :8080
```

Expected result:

```text
No output for port 8080
```

Test the Nginx configuration before starting it:

```bash
nginx -t
```

Expected result:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Start Nginx:

```bash
nginx
```

## Validation

Confirm Nginx is running:

```bash
ps aux | grep nginx
```

Expected result:

```text
nginx: master process nginx
nginx: worker process
```

Confirm Nginx is now listening on port `8080`:

```bash
ss -tulpn | grep :8080
```

Expected result should show Nginx or the Nginx listener using port `8080`.

Validate the health endpoint:

```bash
curl -i http://localhost:8080/health
```

Expected result:

```text
HTTP/1.1 200 OK

OK
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- A port conflict happens when another process is already listening on the port required by the expected service.
- `ss -tulpn` is useful for identifying which process owns a listening port.
- `ps -fp <PID>` helps inspect a process before stopping it.
- Do not kill a process until you confirm it is the wrong process.
- After freeing a port, verify the port is actually free before starting the intended service.
- `nginx -t` should be used to test Nginx configuration before starting or reloading Nginx.
- A running service is not enough; validate the actual endpoint with `curl`.
- Troubleshooting should follow evidence: identify, understand, decide, act, and validate.

## Knowledge Check

### Question 1

When Nginx cannot bind to port `8080`, what should you check first?

A. Which process is currently listening on port `8080`  
B. Whether `/tmp` is empty  
C. Whether the server hostname is correct  
D. Whether the Nginx log directory can be deleted  

**Answer:** A

### Question 2

Which command identifies the process currently listening on port `8080`?

A. `ls -l /var/log/nginx`  
B. `ss -tulpn | grep :8080`  
C. `cat /etc/passwd`  
D. `df -h /`  

**Answer:** B
