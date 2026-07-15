# IAM Access Key Rotation

## Scenario

A CI/CD deployment IAM user has undergone an access key rotation.

A new access key has been created successfully, but the previous access key was never removed.

Both access keys remain active, creating an unnecessary security risk.

## Requirement

Ensure the IAM deployment user has:

```text
Exactly one active access key
```

## Initial State

The IAM user contained two active access keys.

The newer key had already been created, but the previous deployment key was still active.

## Troubleshooting Path

```text
Understand IAM Access Keys
↓
Inspect IAM user
↓
List active access keys
↓
Compare creation timestamps
↓
Identify stale credential
↓
Remove old access key
↓
Validate only one active key remains
```

## Understanding the Architecture

Applications, CI/CD pipelines, and automation tools authenticate to AWS using IAM Access Keys.

Unlike interactive users, applications normally authenticate using:

```text
Access Key ID

+

Secret Access Key
```

Example:

```text
GitLab CI

↓

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

↓

AWS APIs
```

Examples include:

- S3 uploads
- DynamoDB access
- SQS messaging
- Lambda deployments
- Terraform
- Jenkins pipelines

## Understanding Access Key Rotation

A proper rotation follows this lifecycle:

```text
Old Key

↓

Create New Key

↓

Update Applications

↓

Validate Everything

↓

Delete Old Key
```

For a short period, having two active keys is normal.

However, after validation completes, the previous key must be removed.

Leaving the previous key active creates an unnecessary security risk.

## Verification Before Fix

Read the challenge README:

```bash
cat /home/devops/README.txt
```

Evidence:

```text
IAM User:

dctf-ses-562c788876026727577c6edd-ci-deploy

Expected Active Keys:

1
```

Inspect the active access keys:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" iam list-access-keys \
--user-name "dctf-ses-562c788876026727577c6edd-ci-deploy"
```

Result:

```text
Two active access keys
```

Compare the creation timestamps:

```text
Key 1

CreateDate:
2026-07-15T00:30:37...

↓

Older

Key 2

CreateDate:
2026-07-15T00:30:38...

↓

Newer
```

The oldest active key was identified as the stale deployment credential.

## Systematic Elimination

```text
✓ IAM user exists
✓ AWS endpoint reachable
✓ Access keys listed successfully
✓ New access key exists
✗ Previous access key still active
```

The first failed verification was:

```text
The old deployment access key remained active after rotation.
```

## First Finding

```text
Credential rotation had already occurred.

However, the final cleanup step was never completed.

Both access keys remained active.
```

## Fix

Delete the stale access key:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" iam delete-access-key \
--user-name "dctf-ses-562c788876026727577c6edd-ci-deploy" \
--access-key-id "AKIA2LQGMYVZG6G083N8"
```

## Validation

Verify the remaining access keys:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" iam list-access-keys \
--user-name "dctf-ses-562c788876026727577c6edd-ci-deploy"
```

Expected result:

```text
Exactly one active access key remains.
```

Run the challenge validation.

Read the generated flag:

```bash
cat /home/devops/flag.txt
```

After validation passes, submit the generated flag.

## Docker Investigation Habit

Before troubleshooting any containerized application, perform a quick container profile.

Instead of reading the full JSON from:

```bash
docker inspect <container>
```

use Docker's Go template formatting to extract only the information you need.

Container image:

```bash
docker inspect --format '{{.Config.Image}}' <container>
```

Startup command:

```bash
docker inspect --format '{{.Path}}' <container>
```

Container command arguments:

```bash
docker inspect --format '{{json .Config.Cmd}}' <container>
```

Environment variables:

```bash
docker inspect --format '{{range .Config.Env}}{{println .}}{{end}}' <container>
```

Network name:

```bash
docker inspect --format '{{range $name, $_ := .NetworkSettings.Networks}}{{println $name}}{{end}}' <container>
```

Container IP:

```bash
docker inspect --format '{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' <container>
```

Labels:

```bash
docker inspect --format '{{range $k, $v := .Config.Labels}}{{println $k "=" $v}}{{end}}' <container>
```

These commands help quickly answer:

```text
What image?

What starts the container?

What environment variables are injected?

Which network is it attached to?

What IP address does it have?

What labels identify it?
```

This investigation becomes useful during production incidents before changing anything.

## Lessons Learned

- IAM Access Keys authenticate applications to AWS.
- Access key rotation should follow a controlled lifecycle.
- Temporary overlap between old and new keys is expected.
- Old keys should only be removed after all applications have successfully switched to the new key.
- Leaving unused access keys active increases security risk.
- `list-access-keys` is useful for auditing credentials.
- `delete-access-key` permanently removes stale credentials.
- `docker inspect --format` provides fast, targeted container information without reading hundreds of JSON lines.

## Engineering Insight

This challenge demonstrates another important DevOps responsibility:

```text
Credential Lifecycle Management
```

Credentials should never remain active indefinitely.

A proper rotation is not complete until:

```text
New credential deployed

↓

Applications validated

↓

Old credential removed
```

Many production outages occur because engineers delete credentials before validating that applications are using the replacement.

Conversely, many security incidents occur because obsolete credentials are never removed.

Good DevOps engineers balance:

- Operational continuity
- Security
- Validation
- Safe rollback

before completing a credential rotation.

## Knowledge Check

### Question 1

Why should an old IAM access key be deleted after successful rotation?

A. To reduce AWS billing.

B. To eliminate unnecessary active credentials that could be abused.

C. To improve Docker networking.

D. To increase API performance.

**Answer:** B

### Question 2

Which AWS CLI command lists all access keys for an IAM user?

A.

```bash
aws iam describe-user
```

B.

```bash
aws iam list-access-keys
```

C.

```bash
aws iam get-user
```

D.

```bash
aws iam list-users
```

**Answer:** B
