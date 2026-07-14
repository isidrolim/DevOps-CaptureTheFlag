# DynamoDB TTL Disabled

## Scenario

A DynamoDB table stores user session records with expiration timestamps.

Although each session contains an expiration time (`expires_at`), DynamoDB Time To Live (TTL) is disabled.

As a result, expired session records remain in the table indefinitely.

## Requirement

Enable DynamoDB Time To Live (TTL) using the:

```text
expires_at
```

attribute.

## Initial State

The DynamoDB sessions table existed and was reachable.

However, expired session records were never automatically removed because TTL was disabled.

## Troubleshooting Path

```text
Understand DynamoDB TTL
↓
Read the session-specific table name
↓
Inspect current TTL configuration
↓
Compare expected state with actual state
↓
Identify TTL drift
↓
Enable TTL
↓
Validate TTL configuration
```

## Understanding the Architecture

A typical web application stores user sessions.

Example:

```text
Session ID

User

Created Time

expires_at
```

Example record:

```text
SessionID: abc123

User: sid

expires_at: 1720872000
```

Without TTL:

```text
Session expires

↓

Record remains forever
```

With TTL enabled:

```text
Session expires

↓

DynamoDB automatically deletes the record

↓

No manual cleanup required
```

TTL allows DynamoDB to automatically remove expired records based on a timestamp attribute.

## Verification Before Fix

Read the challenge README:

```bash
cat /home/devops/README.txt
```

Evidence:

```text
Table:

dctf-ses-a77709d870d68c322acd78-sessions

TTL attribute:

expires_at

Expected:

TTL Enabled
```

Inspect the current TTL configuration:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" \
dynamodb describe-time-to-live \
--table-name "dctf-ses-a77709d870d68c322acd78-sessions"
```

Result before the fix:

```text
TimeToLiveStatus:

DISABLED
```

## Systematic Elimination

```text
✓ DynamoDB table exists
✓ AWS endpoint reachable
✓ TTL attribute is known
✗ DynamoDB TTL is disabled
```

The first failed verification was:

```text
Time To Live was disabled for the sessions table.
```

## First Finding

```text
Session expiration timestamps already existed.

The database was not automatically deleting expired records because the TTL feature was disabled.
```

## Fix

Enable TTL:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" \
dynamodb update-time-to-live \
--table-name "dctf-ses-a77709d870d68c322acd78-sessions" \
--time-to-live-specification \
"Enabled=true,AttributeName=expires_at"
```

## Validation

Verify the updated configuration:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" \
dynamodb describe-time-to-live \
--table-name "dctf-ses-a77709d870d68c322acd78-sessions"
```

Expected result:

```text
TimeToLiveStatus:

ENABLED

AttributeName:

expires_at
```

Run the challenge validation.

Read the generated flag:

```bash
cat /home/devops/flag.txt
```

After validation passes, submit the generated flag.

## Lessons Learned

- DynamoDB TTL automatically removes expired records.
- TTL requires a timestamp attribute (for example `expires_at`).
- The expiration timestamp alone is not enough.
- DynamoDB must have TTL explicitly enabled.
- `describe-time-to-live` checks the current TTL configuration.
- `update-time-to-live` enables or updates TTL settings.

## Engineering Insight

This challenge introduces another important cloud engineering concept:

```text
Data Lifecycle Management
```

Applications often create temporary data:

- Login sessions
- Password reset tokens
- Temporary authentication tokens
- Shopping carts
- Cache records

Without automatic cleanup:

```text
Expired records accumulate

↓

Database grows

↓

Queries slow down

↓

Storage costs increase
```

TTL allows the database itself to manage record expiration automatically.

Instead of writing scheduled cleanup scripts:

```text
Cron Job

↓

Delete expired sessions
```

DynamoDB performs the cleanup internally.

This challenge reinforces another important engineering principle:

> Configuration controls system behavior.

The session records already contained expiration timestamps.

The problem was not the application.

The problem was the database configuration.

Good engineers always verify:

```text
Data

↓

Configuration

↓

Expected behavior
```

before assuming the application code is faulty.

## Knowledge Check

### Question 1

What does DynamoDB TTL do?

A. Encrypts database records.

B. Automatically deletes expired records based on a timestamp attribute.

C. Creates new tables automatically.

D. Compresses database storage.

**Answer:** B

### Question 2

Which command checks whether DynamoDB TTL is enabled?

A.

```bash
aws dynamodb list-tables
```

B.

```bash
aws dynamodb describe-time-to-live
```

C.

```bash
aws dynamodb scan
```

D.

```bash
aws dynamodb describe-table
```

**Answer:** B
