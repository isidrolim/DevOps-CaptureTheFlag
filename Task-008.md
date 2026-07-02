# Find the 500 Spike

## Scenario

A service is returning intermittent HTTP `500` errors.

The logs reveal that one specific endpoint is failing because of a configuration typo.

The application logs are located at:

```text
/var/log/app/access.log
/var/log/app/error.log
```

The application configuration is located at:

```text
/srv/app/settings.env
```

## Requirement

Identify the failing endpoint and repair the configuration so that:

```text
The failing endpoint returns 200 OK
New error log entries stop for that endpoint
```

## Initial State

The service was running, but one endpoint was repeatedly returning HTTP `500`.

The access log showed repeated failed requests, and the error log showed that the failure was related to an invalid application configuration value.

## Troubleshooting Path

```text
Inspect access log
↓
Identify HTTP 500 responses
↓
Find the failing endpoint
↓
Inspect error log
↓
Identify the config value causing the failure
↓
Inspect application config
↓
Fix the bad config value
↓
Test the failing endpoint
↓
Confirm new errors stop
```

## Verification Before Fix

Inspect recent access log entries:

```bash
tail -n 20 /var/log/app/access.log
```

Count status codes and paths from the access log:

```bash
awk '{print $7, $9}' /var/log/app/access.log | sort | uniq -c | sort -nr
```

The access log showed repeated failures for:

```text
/api/report 500
```

Inspect recent error log entries:

```bash
tail -n 30 /var/log/app/error.log
```

The error log showed:

```text
ERROR /api/report invalid REPORT_BACKEND=disabled
```

Inspect the application configuration:

```bash
cat /srv/app/settings.env
```

Search specifically for the report backend setting:

```bash
grep -n REPORT_BACKEND /srv/app/settings.env
```

Broken configuration:

```text
REPORT_BACKEND=disabled
```

## First Finding

```text
The failing endpoint was /api/report.
The endpoint was returning HTTP 500.
The error log showed REPORT_BACKEND=disabled was invalid.
The application config contained REPORT_BACKEND=disabled.
```

The first failed check was:

```text
The /api/report endpoint depended on REPORT_BACKEND, but the config value was invalid.
```

## Fix

Update the application configuration from `disabled` to `enabled`:

```bash
sed -i 's/^REPORT_BACKEND=disabled/REPORT_BACKEND=enabled/' /srv/app/settings.env
```

Verify the updated configuration:

```bash
cat /srv/app/settings.env
```

Expected result:

```text
REPORT_BACKEND=enabled
```

## Validation

Test the failing endpoint:

```bash
curl -i http://localhost:8080/api/report
```

Expected result:

```text
HTTP/1.0 200 OK

report ok
```

Check the error log before and after testing the endpoint:

```bash
tail -n 5 /var/log/app/error.log
curl -i http://localhost:8080/api/report
tail -n 5 /var/log/app/error.log
```

Expected result:

```text
/api/report returns 200 OK.
No new error appears for invalid REPORT_BACKEND=disabled.
```

No restart was required because the endpoint returned `200 OK` immediately after the config change.

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- HTTP `500` means the server received the request but failed while processing it.
- Access logs help identify which endpoint is failing.
- Error logs help explain why the endpoint is failing.
- Do not guess the endpoint; let the access log reveal it.
- Do not guess the root cause; correlate the access log with the error log.
- `awk`, `sort`, and `uniq -c` are useful for summarizing repeated failures in logs.
- Application config issues can cause only one endpoint to fail while other endpoints still work.
- A restart is not always required if the application reloads or reads config dynamically.
- The best validation is to test the exact endpoint that was failing.

## Knowledge Check

### Question 1

What was the failing endpoint in this challenge?

A. `/health`  
B. `/api/report`  
C. `/login`  
D. `/var/log/app/error.log`  

**Answer:** B

### Question 2

Which command was used to summarize paths and status codes from the access log?

A. `awk '{print $7, $9}' /var/log/app/access.log | sort | uniq -c | sort -nr`  
B. `chmod +x /var/log/app/access.log`  
C. `df -h /var/log/app/access.log`  
D. `systemctl restart access.log`  

**Answer:** A
