# Nginx 502: The Upstream That Moved

## Scenario

Nginx returns `502 Bad Gateway` because `proxy_pass` points to the wrong upstream host or port.

The application backend moved to a different port, but the Nginx reverse proxy configuration was still pointing to the old upstream port.

## Requirement

Repair the Nginx upstream configuration so that:

```text
GET /health through Nginx returns HTTP 200
The response body contains OK
```

## Initial State

Nginx was running and listening on port `80`.

However, requests through Nginx returned:

```text
HTTP/1.1 502 Bad Gateway
```

The backend application was running, but Nginx was proxying traffic to the wrong backend port.

## Troubleshooting Path

```text
Confirm /health returns 502 through Nginx
↓
Check listening ports
↓
Identify the actual backend application port
↓
Inspect Nginx proxy_pass configuration
↓
Compare proxy_pass target with actual backend port
↓
Fix the Nginx upstream port
↓
Test Nginx configuration
↓
Reload Nginx
↓
Validate /health returns 200 OK
```

## Verification Before Fix

Confirm the failure through Nginx:

```bash
curl -i http://localhost/health
```

Initial result:

```text
HTTP/1.1 502 Bad Gateway
Server: nginx/1.24.0
```

Check listening ports and processes:

```bash
ss -tulpn
```

Result showed Nginx listening on port `80`:

```text
0.0.0.0:80 users:(("nginx",pid=15,fd=5))
```

It also showed the backend application listening on port `5001`:

```text
127.0.0.1:5001 users:(("python3",pid=8,fd=3))
```

Inspect the Nginx proxy configuration:

```bash
grep -R "proxy_pass" -n /etc/nginx
```

Broken configuration:

```text
/etc/nginx/conf.d/default.conf:12: proxy_pass http://127.0.0.1:5000;
```

## First Finding

```text
Nginx was reachable and listening on port 80.
The backend application was running on 127.0.0.1:5001.
Nginx proxy_pass was configured for 127.0.0.1:5000.
```

The first failed check was:

```text
Nginx proxy_pass pointed to the wrong backend port.
```

The broken request path was:

```text
Client
↓
Nginx port 80
↓
proxy_pass http://127.0.0.1:5000
↓
Backend actually listening on 127.0.0.1:5001
```

## Fix

Update the Nginx `proxy_pass` target from port `5000` to port `5001`:

```bash
sed -i 's|http://127.0.0.1:5000|http://127.0.0.1:5001|' /etc/nginx/conf.d/default.conf
```

Verify the updated configuration:

```bash
grep -R "proxy_pass" -n /etc/nginx
```

Expected result:

```text
proxy_pass http://127.0.0.1:5001;
```

Test the Nginx configuration before reloading:

```bash
nginx -t
```

Expected result:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Reload Nginx:

```bash
nginx -s reload
```

## Validation

Test the health endpoint through Nginx:

```bash
curl -i http://localhost/health
```

Expected result:

```text
HTTP/1.1 200 OK

OK
```

Confirm that Nginx is still listening:

```bash
ss -tulpn | grep :80
```

Confirm the backend application is still listening on the expected upstream port:

```bash
ss -tulpn | grep :5001
```

Expected final result:

```text
Nginx proxies to the moved upstream successfully.
GET /health through Nginx returns HTTP 200 and body OK.
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- `502 Bad Gateway` usually means Nginx is reachable but cannot successfully reach the upstream backend.
- A reverse proxy depends on both Nginx and the backend application being reachable.
- `ss -tulpn` shows which ports are listening and which process owns them.
- `grep -R "proxy_pass" -n /etc/nginx` helps locate Nginx upstream proxy configuration.
- Always compare the configured `proxy_pass` target with the actual backend listening port.
- `nginx -t` should be used before reloading Nginx.
- `nginx -s reload` applies configuration changes without fully stopping Nginx.
- The best validation is testing the same endpoint required by the challenge: `curl -i http://localhost/health`.

## Knowledge Check

### Question 1

What does an Nginx `502 Bad Gateway` usually indicate in a reverse proxy setup?

A. The client cannot reach Nginx  
B. Nginx is reachable, but it cannot successfully reach the upstream backend  
C. The disk is full  
D. The file permissions on `/tmp` are wrong  

**Answer:** B

### Question 2

Which command finds the Nginx `proxy_pass` configuration?

A. `grep -R "proxy_pass" -n /etc/nginx`  
B. `df -h /etc/nginx`  
C. `chmod +x /etc/nginx/nginx.conf`  
D. `cat /var/log/app/access.log`  

**Answer:** A
