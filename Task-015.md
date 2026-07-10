# Node API Environment Drift

## Scenario

A Node.js API starts successfully, but its health and order endpoints fail because the runtime environment uses a stale or incorrect configuration key.

The application depends on environment variables loaded from:

```text
/srv/api/.env
```

The Node.js process is managed by Supervisor.

## Requirement

Repair the runtime environment so that:

```text
DATABASE_URL is correctly loaded
/health returns HTTP 200
/api/orders returns JSON data
```

## Initial State

The Node.js application started successfully and listened on port `3000`.

However, API requests failed because the expected runtime environment variable was missing.

The application logged runtime errors indicating that the required configuration key could not be found.

## Troubleshooting Path

```text
Verify the application symptom
↓
Read the application log
↓
Inspect runtime environment (.env)
↓
Compare expected configuration with actual configuration
↓
Identify configuration drift
↓
Correct the environment variable
↓
Restart the managed Node.js service
↓
Validate API endpoints
```

## Understanding the Architecture

The Node.js application is managed by **Supervisor**.

Startup sequence:

```text
Supervisor
↓
Runs startup command
↓
Bash loads .env
↓
Environment variables are exported
↓
Node.js starts
↓
Application begins serving requests
```

Supervisor starts the application using:

```text
command=/bin/bash -lc 'set -a; . /srv/api/.env; set +a; exec npm start'
```

Meaning:

```text
Load .env
↓
Export variables
↓
Start Node.js
```

This means environment variables are loaded **only when the application starts**.

Editing `.env` alone does not change the environment of an already-running Node.js process.

## Verification Before Fix

Confirm the runtime error:

```bash
cat /var/log/node-api/app.log
```

Result:

```text
ERROR missing DATABASE_URL
```

Inspect the runtime configuration:

```bash
cat /srv/api/.env
```

Broken configuration:

```text
PORT=3000
DB_URL=file:///srv/api/data/orders.json
```

Inspect the Supervisor configuration:

```bash
cat /etc/supervisor/conf.d/node-api.conf
```

Important startup command:

```text
command=/bin/bash -lc 'set -a; . /srv/api/.env; set +a; exec npm start'
```

## Systematic Elimination

```text
✓ Docker container is running
✓ Node.js process starts successfully
✓ Supervisor is managing the application
✓ Runtime configuration file exists
✗ Expected configuration key is incorrect
```

The first failed verification was:

```text
The application expected:

DATABASE_URL

The runtime configuration provided:

DB_URL
```

## First Finding

```text
The Node.js application expected the environment variable:

DATABASE_URL

The runtime configuration contained:

DB_URL

The configuration file existed, but the required key had drifted.
```

## Fix

Correct the environment variable:

```text
Before:

DB_URL=file:///srv/api/data/orders.json

After:

DATABASE_URL=file:///srv/api/data/orders.json
```

Using:

```bash
sed -i 's/^DB_URL=/DATABASE_URL=/' /srv/api/.env
```

Verify the updated configuration:

```bash
cat /srv/api/.env
```

Restart the managed service:

```bash
supervisorctl restart node-api
```

## Validation

Confirm the health endpoint:

```bash
curl http://127.0.0.1:3000/health
```

Confirm the orders endpoint:

```bash
curl http://127.0.0.1:3000/api/orders
```

Expected result:

```text
Health returns HTTP 200.

Orders endpoint returns JSON data.
```

After validation passes, click **Check Solution** and submit the generated flag.

## Lessons Learned

- Runtime configuration can exist while still containing incorrect configuration keys.
- Reading application logs should always come before editing configuration files.
- Node.js applications commonly receive configuration from environment variables.
- Editing `.env` does not automatically update a running Node.js process.
- Managed services must be restarted to reload updated runtime configuration.
- Read system feedback carefully before retrying commands.

## Engineering Insight

One of the biggest differences between PHP and Node.js runtime configuration is **when configuration is loaded**.

In this challenge:

```text
Supervisor
↓

Loads .env

↓

Starts Node.js
```

Once the process starts, the environment variables are stored in the process memory.

Editing `.env` later does **not** update the already-running application.

This reinforces an important production engineering principle:

> Understand the application startup lifecycle before deciding whether a restart is necessary.

Good engineers do not restart services by habit.

They understand **why** a restart is or is not required.

Another important lesson from this challenge was learning from system feedback.

The first restart attempt was incorrect:

```bash
supervisor reload node-api
```

The system returned an error.

Instead of guessing, the error message was read, understood, and corrected.

Eventually the correct command was used:

```bash
supervisorctl restart node-api
```

Production troubleshooting is an iterative process.

The system itself often tells you what to do next if you are willing to read its feedback.

## Knowledge Check

### Question 1

Why did editing `.env` alone not immediately fix the application?

A. Because `.env` is read only when the Node.js process starts.

B. Because `.env` files cannot be modified.

C. Because Nginx blocks environment variables.

D. Because Docker caches `.env` forever.

**Answer:** A

### Question 2

Which command correctly restarted the managed Node.js application?

A.

```bash
systemctl restart node-api
```

B.

```bash
supervisor reload node-api
```

C.

```bash
supervisorctl restart node-api
```

D.

```bash
npm restart
```

**Answer:** C
