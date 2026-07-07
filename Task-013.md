# PHP App Missing Runtime Config

## Scenario

A PHP application returns a fatal runtime error because an expected environment variable is missing or incorrectly named.

The application reads its runtime configuration from:

```text
/var/www/html/.env
```

The runtime error is logged in:

```text
/var/log/php/error.log
```

## Requirement

Repair the application runtime configuration so that:

```text
The PHP endpoint returns HTTP 200
The application returns the expected runtime output
The fatal runtime error no longer appears
```

## Initial State

Nginx, PHP-FPM, and PHP were all functioning correctly.

However, requesting:

```text
http://localhost/index.php
```

returned a PHP fatal runtime error.

The application could not find the expected runtime configuration.

## Troubleshooting Path

```text
Confirm the runtime error
↓
Read the PHP error log
↓
Inspect the application .env file
↓
Compare the expected configuration key with the actual configuration
↓
Identify the incorrect environment variable
↓
Correct the runtime configuration
↓
Validate the PHP endpoint
```

## Verification Before Fix

Confirm the runtime error:

```bash
curl http://localhost/index.php
```

Initial result:

```text
Fatal error:
missing APP_MODE=production runtime config
```

Inspect the PHP runtime error log:

```bash
cat /var/log/php/error.log
```

Result:

```text
Fatal error:
missing APP_MODE=production runtime config
```

Inspect the runtime configuration:

```bash
cat /var/www/html/.env
```

Broken configuration:

```text
APP_NAME=runtime-config-demo
AP_MODE=production
```

Notice the typo:

```text
Expected:

APP_MODE

Actual:

AP_MODE
```

## Systematic Elimination

```text
✓ Browser reaches application
✓ Nginx is working
✓ PHP-FPM is working
✓ PHP interpreter executes
✓ .env file exists
✗ Runtime configuration key is incorrect
```

The first failed verification was:

```text
The runtime configuration key was misspelled.
```

## First Finding

```text
The application was expecting:

APP_MODE

The runtime configuration contained:

AP_MODE

The .env file existed, but the required configuration key was incorrectly named.
```

## Fix

Correct the environment variable:

```text
Before:

AP_MODE=production

After:

APP_MODE=production
```

One possible command:

```bash
sed -i 's/^AP_MODE=/APP_MODE=/' /var/www/html/.env
```

Or edit manually:

```bash
nano /var/www/html/.env
```

## Validation

Verify the corrected configuration:

```bash
cat /var/www/html/.env
```

Expected result:

```text
APP_NAME=runtime-config-demo
APP_MODE=production
```

Test the application:

```bash
curl http://localhost/index.php
```

Expected result:

```text
PHP runtime config OK
```

The challenge passed immediately after updating the configuration.

No PHP-FPM or Nginx reload was required because the application picked up the updated runtime configuration automatically.

After validation passes, click **Check Solution** and submit the generated flag.

## Lessons Learned

- PHP runtime configuration is commonly stored in `.env` files.
- A configuration file can exist while still containing incorrect or misspelled keys.
- Always read the application error before editing configuration files.
- Runtime errors often tell you exactly which configuration key is missing.
- Verify configuration changes by testing the application, not by assuming a restart is required.

## Engineering Insight

One of the most common production mistakes is assuming that every configuration change requires a service restart.

In this challenge, evidence proved otherwise.

The application immediately returned:

```text
PHP runtime config OK
```

This demonstrated that the application reloaded or reread the updated runtime configuration without restarting PHP-FPM.

A key production principle is:

> Never restart a service simply because "that's what we usually do."

Instead:

1. Apply the smallest safe change.
2. Test the application.
3. Restart only if the evidence shows it is necessary.

Good engineers minimize operational risk by avoiding unnecessary service interruptions.

## Knowledge Check

### Question 1

What was the root cause of the runtime error?

A. PHP-FPM was stopped.

B. The `.env` file was missing.

C. The runtime configuration key `APP_MODE` was misspelled as `AP_MODE`.

D. Nginx was listening on the wrong port.

**Answer:** C

### Question 2

After correcting the `.env` file, why was a PHP-FPM restart not required?

A. Because PHP automatically restarted itself.

B. Because the application immediately read the corrected configuration and returned the expected output.

C. Because Nginx restarted PHP automatically.

D. Because PHP ignores `.env` files.

**Answer:** B
