# PHP-FPM Socket Mismatch

## Scenario

Nginx returns `502 Bad Gateway` because the configured FastCGI target does not match where PHP-FPM is actually listening.

Nginx successfully receives HTTP requests, but it cannot forward PHP requests to the correct PHP runtime.

## Requirement

Repair the FastCGI configuration so that:

```text
/index.php returns the expected dynamic PHP output
Nginx and PHP-FPM are healthy
```

## Initial State

Nginx was running and responding to HTTP requests.

Static pages worked, but requesting:

```text
http://localhost/index.php
```

returned:

```text
HTTP/1.1 502 Bad Gateway
```

The PHP application was running, but Nginx was forwarding PHP requests to the wrong FastCGI socket.

## Troubleshooting Path

```text
Verify the symptom
↓
Understand the PHP request flow
↓
Inspect Nginx FastCGI target
↓
Identify where PHP-FPM is actually listening
↓
Compare both endpoints
↓
Identify the mismatch
↓
Update FastCGI target
↓
Validate Nginx configuration
↓
Reload Nginx
↓
Validate dynamic PHP output
```

## Understanding the Architecture

Unlike static HTML files, Nginx does **not** execute PHP files.

Instead, the request flows like this:

```text
Browser
↓
Nginx
↓
FastCGI (fastcgi_pass)
↓
PHP-FPM
↓
PHP executes index.php
↓
Response returned to Nginx
↓
Browser
```

PHP-FPM (PHP FastCGI Process Manager) is the service responsible for executing PHP code.

Nginx only forwards PHP requests to PHP-FPM.

Communication usually happens using either:

```text
Unix Socket

/run/php/php8.x-fpm.sock
```

or

```text
TCP

127.0.0.1:9000
```

Both are valid, but **Nginx and PHP-FPM must agree on the same communication endpoint.**

## Verification Before Fix

Confirm the failure:

```bash
curl http://localhost/index.php
```

Initial result:

```text
HTTP/1.1 502 Bad Gateway
```

Inspect the configured FastCGI target:

```bash
grep -R "fastcgi_pass" -n /etc/nginx
```

Result:

```text
fastcgi_pass unix:/run/php/php8.2-fpm.sock;
```

Check where PHP-FPM is actually listening:

```bash
ss -xlpn | grep php
```

Result:

```text
/run/php/php8.3-fpm.sock
```

Inspect the socket directory:

```bash
ls -lah /run/php
```

Result:

```text
php8.3-fpm.sock
php8.3-fpm.pid
```

## Systematic Elimination

```text
✓ Browser reaches Nginx
✓ Nginx is running
✓ PHP-FPM service is running
✓ PHP-FPM is listening
✗ Nginx points to php8.2-fpm.sock
✗ PHP-FPM is listening on php8.3-fpm.sock
```

The first failed verification was:

```text
Nginx FastCGI target did not match the active PHP-FPM socket.
```

## First Finding

```text
Nginx expected:

/run/php/php8.2-fpm.sock

PHP-FPM was actually listening on:

/run/php/php8.3-fpm.sock
```

Because the socket names did not match, Nginx could not forward PHP requests, resulting in HTTP 502.

## Fix

Update the FastCGI socket:

```bash
sed -i 's|/run/php/php8.2-fpm.sock|/run/php/php8.3-fpm.sock|' /etc/nginx/conf.d/default.conf
```

Verify the change:

```bash
grep -R "fastcgi_pass" -n /etc/nginx/conf.d/default.conf
```

Expected result:

```text
fastcgi_pass unix:/run/php/php8.3-fpm.sock;
```

## Validation

Before reloading Nginx, validate the configuration:

```bash
nginx -t
```

Expected result:

```text
syntax is ok
test is successful
```

Reload Nginx:

```bash
nginx -s reload
```

Test the PHP page:

```bash
curl http://localhost/index.php
```

Expected result:

```text
PHP-FPM OK from 8.3.6
```

After validation passes, click **Check Solution** and submit the generated flag.

## Lessons Learned

- Nginx does **not** execute PHP.
- PHP files are executed by PHP-FPM.
- `fastcgi_pass` tells Nginx where PHP-FPM is listening.
- PHP-FPM may listen on a Unix socket or a TCP port.
- Both Nginx and PHP-FPM must use the same communication endpoint.
- `ss -xlpn` is useful for inspecting Unix socket listeners.
- `grep` is useful for locating the configured FastCGI target.
- Always validate Nginx configuration before reloading.

## Engineering Insight

The first instinct when seeing `502 Bad Gateway` is often:

> "PHP is broken."

However, the evidence showed something different.

Nginx was healthy.

PHP-FPM was healthy.

The failure was the communication layer between them.

Good troubleshooting isolates each layer independently before changing configuration.

This challenge reinforces an important production principle:

> Never assume two services are communicating correctly.

Prove both sides independently.

1. Verify what the client expects.
2. Verify what the backend is actually providing.
3. Compare the two.
4. Fix only the mismatch.

Before applying configuration changes to a production web server, always validate them first.

```bash
nginx -t
```

This command reduces operational risk by preventing invalid configurations from being loaded into a running production service.

Good engineers don't simply fix systems.

They reduce the probability of creating a second incident while fixing the first.

## Knowledge Check

### Question 1

Why did Nginx return `502 Bad Gateway`?

A. PHP was not installed.

B. Nginx was configured to use a different PHP-FPM socket than the one PHP-FPM was actually listening on.

C. Apache was running.

D. Port 80 was closed.

**Answer:** B

### Question 2

Before reloading Nginx after changing the FastCGI configuration, which command should be run?

A. `systemctl restart php-fpm`

B. `nginx -t`

C. `chmod +x /run/php/php8.3-fpm.sock`

D. `tail -f /var/log/nginx/error.log`

**Answer:** B
