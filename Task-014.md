# Static Assets 403

## Scenario

The HTML page loads successfully, but static assets such as CSS return an error because Nginx cannot access the required directory path.

The website references:

```text
/assets/app.css
```

The application shell loads, but the browser cannot retrieve the stylesheet.

## Requirement

Repair the directory permissions so that:

```text
/assets/app.css returns HTTP 200
Relevant paths are not world-writable
```

## Initial State

The application homepage loaded successfully.

However, the referenced CSS asset could not be served because Nginx was denied permission while traversing the directory path.

The CSS file itself existed.

## Troubleshooting Path

```text
Confirm HTML loads
↓
Verify CSS request
↓
Inspect Nginx logs
↓
Verify CSS file exists
↓
Inspect directory permissions
↓
Identify traversal permission issue
↓
Fix directory permissions
↓
Validate CSS returns HTTP 200
```

## Verification Before Fix

Confirm the application homepage loads:

```bash
curl http://localhost
```

Result:

```text
HTTP 200 OK
```

The HTML referenced:

```html
<link rel="stylesheet" href="/assets/app.css">
```

Test the CSS file directly:

```bash
curl -I http://localhost/assets/app.css
```

Result:

```text
HTTP/1.1 404 Not Found
```

Inspect the Nginx access log:

```bash
cat /var/log/nginx/access.log
```

Inspect the Nginx error log:

```bash
cat /var/log/nginx/error.log
```

Important evidence:

```text
stat() "/var/www/html/assets/app.css" failed (13: Permission denied)
```

Confirm the CSS file exists:

```bash
ls -lah /var/www/html/assets/
```

Result:

```text
app.css exists
```

Inspect the directory permissions:

```bash
ls -ld /var/www/html/assets
```

Broken permissions:

```text
drw-------
```

## Systematic Elimination

```text
✓ Browser reaches Nginx
✓ HTML loads successfully
✓ CSS file exists
✓ Nginx knows the correct file path
✗ Nginx cannot traverse the assets directory
```

The first failed verification was:

```text
The assets directory was missing execute (traverse) permission.
```

## First Finding

```text
The CSS file was present.

The problem was not the file itself.

Nginx was denied access because it could not traverse the assets directory.

The directory lacked execute permission.
```

## Fix

Grant the directory the correct traversal permissions:

```bash
chmod 755 /var/www/html/assets
```

Verify the new permissions:

```bash
ls -ld /var/www/html/assets
```

Expected result:

```text
drwxr-xr-x
```

## Validation

Test the CSS endpoint again:

```bash
curl -I http://localhost/assets/app.css
```

Expected result:

```text
HTTP/1.1 200 OK
```

Verify the CSS file is still readable:

```bash
ls -lah /var/www/html/assets
```

Confirm the directory is **not** world-writable:

```bash
stat -c "%A %a" /var/www/html/assets
```

Expected result:

```text
drwxr-xr-x
755
```

After validation passes, click **Check Solution** and submit the generated flag.

## Lessons Learned

- A `404` from the browser does not always mean the file is missing.
- Nginx error logs provide the real reason behind many HTTP errors.
- Directories require execute (`x`) permission for processes to traverse them.
- Files can have correct permissions while access still fails because of the parent directory.
- `chmod 755` allows traversal without allowing unauthorized modification.
- Always inspect both access and error logs before changing permissions.

## Engineering Insight

One of the biggest misconceptions in Linux is believing that execute permission only applies to executable files.

For directories, execute permission means:

```text
Permission to traverse or enter the directory.
```

Think of a directory as a hallway.

```text
Building
│
├── assets
│      │
│      └── app.css
```

Without execute permission on the hallway:

```text
Nginx cannot walk into the directory.

Therefore it cannot reach:

app.css
```

Even though the file exists.

This challenge reinforces another production engineering principle:

> Gather evidence before changing permissions.

The browser showed:

```text
404 Not Found
```

But the Nginx error log revealed:

```text
Permission denied
```

Those are two very different root causes.

Good engineers trust evidence, not assumptions.

Finally, avoid using:

```bash
chmod 777
```

Simply because it "works."

Use the **least privilege principle**:

- Owner: full control
- Group: read and traverse
- Others: read and traverse

Only grant the permissions required for the service to function.

## Knowledge Check

### Question 1

Why was Nginx unable to serve `app.css`?

A. The CSS file was missing.

B. The CSS file was corrupted.

C. Nginx could not traverse the `assets` directory because execute permission was missing.

D. PHP-FPM was stopped.

**Answer:** C

### Question 2

Why is `755` preferred over `777` for the assets directory?

A. Because `777` prevents reading files.

B. Because `755` allows users to traverse and read files while preventing unauthorized modification by non-owners.

C. Because `755` makes the directory invisible.

D. Because Nginx only works with `755`.

**Answer:** B
