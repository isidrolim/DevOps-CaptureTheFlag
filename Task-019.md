# SSM Parameter Drift

## Scenario

An application reads its runtime mode from AWS Systems Manager (SSM) Parameter Store.

The parameter exists, but its value has drifted from the expected production configuration.

As a result, the application continues operating in staging mode.

## Requirement

Update the SSM parameter so that:

```text
The runtime parameter value is exactly:

production
```

## Initial State

The application was successfully reading its configuration from AWS Systems Manager Parameter Store.

However, the stored parameter value was:

```text
staging
```

instead of:

```text
production
```

## Troubleshooting Path

```text
Understand SSM Parameter Store
↓
Read session-specific parameter path
↓
Inspect current parameter value
↓
Compare expected value with actual value
↓
Identify configuration drift
↓
Update parameter
↓
Validate updated value
```

## Understanding the Architecture

Unlike previous challenges where configuration was stored locally:

```text
.env

config.yml

settings.json
```

this application retrieves configuration from a centralized configuration service:

```text
AWS Systems Manager

↓

Parameter Store

↓

Application
```

Instead of every application maintaining its own copy of configuration, applications request configuration dynamically from Parameter Store.

Example:

```text
Application

↓

Request:

APP_MODE

↓

SSM Parameter Store

↓

Returns:

production
```

This provides centralized configuration management across multiple applications.

## Verification Before Fix

Read the session instructions:

```bash
cat /home/devops/README.txt
```

Result:

```text
Parameter:

/dctf/.../orders/APP_MODE

Expected value:

production
```

Retrieve the parameter:

```bash
PARAM="/dctf/.../orders/APP_MODE"

aws --endpoint-url "$AWS_ENDPOINT_URL" \
ssm get-parameter \
--name "$PARAM"
```

Result:

```text
Value:

staging
```

## Systematic Elimination

```text
✓ Parameter exists
✓ AWS endpoint reachable
✓ Application reads Parameter Store
✗ Parameter value is incorrect
```

The first failed verification was:

```text
The parameter value drifted from production to staging.
```

## First Finding

```text
The application configuration itself was healthy.

The Parameter Store key existed.

Only the stored value was incorrect.
```

## Fix

Update the parameter:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" \
ssm put-parameter \
--name "$PARAM" \
--type String \
--value "production" \
--overwrite
```

## Validation

Retrieve the parameter again:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" \
ssm get-parameter \
--name "$PARAM"
```

Expected result:

```text
Value:

production
```

Run the challenge validation.

Read the generated flag:

```bash
cat /home/devops/flag.txt
```

After validation passes, submit the generated flag.

## Lessons Learned

- AWS Systems Manager Parameter Store centralizes application configuration.
- Applications can retrieve configuration directly from Parameter Store instead of local files.
- Configuration drift can occur even when the parameter exists.
- Always verify both the existence and the value of a configuration parameter.
- `get-parameter` retrieves the current parameter value.
- `put-parameter --overwrite` updates an existing parameter safely.

## Engineering Insight

One of the biggest advantages of centralized configuration management is eliminating duplicated configuration across multiple servers.

Instead of:

```text
Server 1

↓

.env

Server 2

↓

.env

Server 3

↓

.env
```

Modern cloud applications often use:

```text
Application

↓

AWS Parameter Store

↓

Shared Configuration
```

This allows configuration changes to be managed centrally.

However, centralized configuration introduces a different operational risk:

```text
Configuration Drift
```

The parameter existed.

The application was functioning.

Only the stored value had become incorrect.

This challenge demonstrates another important production engineering principle:

> Configuration can be valid but still be wrong.

Good engineers verify:

1. Does the configuration exist?
2. Is it the expected value?
3. Is the application consuming the correct configuration?

Only after answering all three questions should a configuration change be applied.

## Knowledge Check

### Question 1

What was the root cause of this challenge?

A. The parameter did not exist.

B. The application could not reach AWS.

C. The Parameter Store value drifted from `production` to `staging`.

D. Docker networking failed.

**Answer:** C

### Question 2

Which AWS CLI command retrieves the current value of a Parameter Store entry?

A.

```bash
aws s3 ls
```

B.

```bash
aws ssm get-parameter --name "<parameter>"
```

C.

```bash
aws iam get-user
```

D.

```bash
aws configure
```

**Answer:** B
