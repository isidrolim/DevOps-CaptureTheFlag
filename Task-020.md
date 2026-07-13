# SQS Visibility Timeout Drift

## Scenario

A worker queue is retrying messages immediately because the SQS queue `VisibilityTimeout` attribute drifted to `0`.

This causes messages to become visible again immediately after a worker receives them.

## Requirement

Update the SQS queue so that:

```text
VisibilityTimeout = 30
```

## Initial State

The queue existed and was reachable through the local AWS-compatible endpoint.

However, workers were immediately retrying the same order message because the queue visibility timeout was incorrectly set.

## Troubleshooting Path

```text
Understand SQS queue behavior
↓
Read the queue URL from README.txt
↓
Inspect the current queue VisibilityTimeout
↓
Compare expected value with actual value
↓
Identify attribute drift
↓
Update the queue attribute
↓
Validate the corrected value
```

## Understanding the Architecture

SQS is a queue service used to decouple work between applications and workers.

Example:

```text
Application
↓
SQS Queue
↓
Worker
↓
Process message
```

When a worker receives a message, SQS temporarily hides that message from other workers.

That hidden period is called:

```text
Visibility Timeout
```

Normal behavior:

```text
Worker 1 receives message
↓
SQS hides message for 30 seconds
↓
Worker 1 processes message
↓
Worker 1 deletes message
```

If `VisibilityTimeout` is `0`:

```text
Worker 1 receives message
↓
Message immediately becomes visible again
↓
Worker 2 receives same message
↓
Worker 3 receives same message
```

This can cause duplicate processing.

## Verification Before Fix

Read the challenge README:

```bash
cat /home/devops/README.txt
```

Evidence:

```text
Expected VisibilityTimeout: 30
Current symptom: workers immediately retry the same order message
```

Inspect the current queue attribute:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" sqs get-queue-attributes \
  --queue-url "http://floci:4566/000000000000/dctf-ses-64bbb8e671ead8df2c98d984-orders" \
  --attribute-names VisibilityTimeout
```

Result before the fix:

```text
"VisibilityTimeout": "0"
```

## Systematic Elimination

```text
✓ SQS queue exists
✓ AWS-compatible endpoint is reachable
✓ Queue URL is known
✓ VisibilityTimeout attribute can be queried
✗ VisibilityTimeout is set to 0 instead of 30
```

The first failed verification was:

```text
The SQS queue attribute drifted from 30 to 0.
```

## First Finding

```text
The queue itself was healthy.

The worker behavior was caused by a queue attribute misconfiguration.

VisibilityTimeout was set to 0, causing messages to become immediately visible after being received.
```

## Fix

Update the queue attribute:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" sqs set-queue-attributes \
  --queue-url "http://floci:4566/000000000000/dctf-ses-64bbb8e671ead8df2c98d984-orders" \
  --attributes VisibilityTimeout=30
```

## Validation

Verify the updated attribute:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" sqs get-queue-attributes \
  --queue-url "http://floci:4566/000000000000/dctf-ses-64bbb8e671ead8df2c98d984-orders" \
  --attribute-names VisibilityTimeout
```

Expected result:

```text
"VisibilityTimeout": "30"
```

Run the challenge validation.

Read the generated flag:

```bash
cat /home/devops/flag.txt
```

After validation passes, submit the generated flag.

## Lessons Learned

- SQS is a queue service used to coordinate work between producers and workers.
- A worker receives a message, processes it, then deletes it.
- Visibility Timeout prevents other workers from receiving the same message while it is being processed.
- If Visibility Timeout is `0`, messages immediately become visible again.
- This can cause duplicate message processing.
- `get-queue-attributes` reads queue configuration.
- `set-queue-attributes` updates queue configuration.

## Engineering Insight

This challenge was not only about AWS SQS.

It was about distributed work coordination.

When multiple workers process messages from the same queue, they need a mechanism that prevents duplicate processing.

SQS solves this with Visibility Timeout.

```text
Receive message
↓
Hide message temporarily
↓
Process message
↓
Delete message
```

If the visibility timeout is too low, duplicate processing can happen.

If it is too high, failed messages may take too long to retry.

Good engineers understand that queue settings directly affect business behavior.

For example:

```text
VisibilityTimeout = 0
```

can cause:

```text
Duplicate orders
Duplicate emails
Duplicate payments
Duplicate shipments
```

This challenge reinforces an important production principle:

> Queue configuration is application behavior.

A queue can be healthy and reachable while still being incorrectly configured.

Always compare:

```text
Expected attribute
↓
Actual attribute
↓
Business behavior
```

## Knowledge Check

### Question 1

What does SQS Visibility Timeout do?

A. Deletes messages immediately  
B. Temporarily hides a received message from other workers  
C. Creates a new queue  
D. Changes the queue name  

**Answer:** B

### Question 2

Why was `VisibilityTimeout=0` a problem?

A. It prevented all workers from reading messages  
B. It deleted messages too early  
C. It made messages immediately visible again, allowing duplicate processing  
D. It disabled the queue  

**Answer:** C
