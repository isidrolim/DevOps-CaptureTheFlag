# Apache Is Serving the Wrong Site

## Scenario

Apache is serving the default website instead of the intended application website.

The application Virtual Host already exists, but Apache is currently serving the default site.

## Requirement

Repair the Apache Virtual Host configuration so that:

```text
The root URL returns the application marker string
Apache configuration test passes
```

## Initial State

Apache was running and responding to HTTP requests.

However, browsing to the root URL displayed the default Apache page instead of the application website.

The application Virtual Host configuration already existed, but it was not enabled.

## Troubleshooting Path

```text
Confirm the symptom
↓
Verify Apache is responding
↓
Inspect enabled Virtual Hosts
↓
Inspect available Virtual Hosts
↓
Compare enabled site with application site
↓
Enable application Virtual Host
↓
Disable default Virtual Host
↓
Validate Apache configuration
↓
Reload Apache
↓
Validate application website
```

## Verification Before Fix

Confirm what Apache is serving:

```bash
curl http://localhost
```

Initial result:

```text
Apache Default Page
```

Inspect enabled Virtual Hosts:

```bash
ls -lah /etc/apache2/sites-enabled/
```

Result:

```text
000-default.conf
```

Inspect available Virtual Hosts:

```bash
ls -lah /etc/apache2/sites-available/
```

Result:

```text
000-default.conf
app.conf
default-ssl.conf
```

Inspect the application Virtual Host:

```bash
cat /etc/apache2/sites-available/app.conf
```

Application configuration:

```text
DocumentRoot /var/www/app
ServerName localhost
```

## Systematic Elimination

```text
✓ Browser can reach Apache
✓ Apache is running
✓ Apache is serving HTTP requests
✓ Application Virtual Host exists
✗ Wrong Virtual Host is enabled
```

The first failed verification was:

```text
Apache was serving the default Virtual Host instead of the application Virtual Host.
```

## First Finding

```text
Apache itself was healthy.

The application Virtual Host already existed in sites-available.

Only the default Virtual Host was enabled, causing Apache to serve the wrong website.
```

## Fix

Enable the application Virtual Host:

```bash
a2ensite app.conf
```

Reload Apache:

```bash
service apache2 reload
```

Disable the default Virtual Host:

```bash
a2dissite 000-default.conf
```

Reload Apache again:

```bash
service apache2 reload
```

## Validation

Confirm the application website is now served:

```bash
curl http://localhost
```

Expected result:

```text
WEB-002 app marker: Apache is serving the intended site.
```

Validate Apache configuration:

```bash
apachectl configtest
```

Expected result:

```text
Syntax OK
```

After validation passes, click **Check Solution** in the challenge UI and submit the generated flag.

## Lessons Learned

- Apache selects websites using Virtual Hosts.
- `sites-available` stores all Virtual Host configurations.
- `sites-enabled` contains the Virtual Hosts Apache actively serves.
- A Virtual Host must be enabled before Apache can use it.
- `a2ensite` enables a Virtual Host.
- `a2dissite` disables a Virtual Host.
- Always validate the application using the same URL reported by the user.

## Engineering Insight

Before applying configuration changes to a production web server, always prove the configuration is valid.

Apache provides:

```bash
apachectl configtest
```

This is not simply a syntax checker.

It is a risk-reduction step.

A broken configuration can prevent Apache from reloading successfully, making the website unavailable even though the original problem has already been identified.

Good engineers do not only fix problems.

They minimize operational risk while applying the fix.

Another important operational principle is change sequencing.

When replacing one Virtual Host with another:

1. Enable the new site first.
2. Validate the configuration.
3. Disable the old site.
4. Reload Apache.
5. Validate the application.

This minimizes the risk of leaving Apache without a working Virtual Host during the change.

## Knowledge Check

### Question 1

Apache is serving the default page. What does this immediately prove?

A. Apache is running and responding to requests.

B. Apache is stopped.

C. The network is disconnected.

D. The application has crashed.

**Answer:** A

### Question 2

Which helper command enables an Apache Virtual Host?

A. `a2dissite app.conf`

B. `a2ensite app.conf`

C. `systemctl enable apache2`

D. `apachectl start`

**Answer:** B
