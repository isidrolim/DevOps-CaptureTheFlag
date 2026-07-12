# S3 Missing Static Asset

## Scenario

A static website has been deployed to an S3 bucket.

The HTML page was uploaded successfully, but the required JavaScript asset was accidentally omitted during deployment.

As a result, the website shell loads, but the application JavaScript never executes.

## Requirement

Restore the missing static asset so that:

```text
assets/app.js exists in the S3 bucket
The uploaded object has the correct Content-Type
The deployed website is complete
```

## Initial State

The deployment uploaded:

```text
index.html
```

However, the JavaScript asset:

```text
assets/app.js
```

was missing from the S3 bucket.

The local deployment files still existed inside:

```text
/opt/challenge/files
```

## Troubleshooting Path

```text
Understand the deployment architecture
↓
Identify the S3 bucket
↓
Inspect local deployment files
↓
Verify S3 bucket contents
↓
Compare expected files with deployed files
↓
Identify missing deployment artifact
↓
Upload missing object
↓
Validate S3 contents
```

## Understanding the Architecture

A static website deployment usually follows this process:

```text
Source Files

↓

Build

↓

Deployment Artifacts

↓

Upload to S3

↓

Browser
```

The browser first downloads:

```text
index.html
```

The HTML then requests:

```text
assets/app.js
```

If the JavaScript asset is missing from S3:

```text
HTML loads

↓

JavaScript returns 404

↓

Application functionality is broken
```

The deployment itself is incomplete.

## Verification Before Fix

Read the deployment instructions:

```bash
cat /home/devops/README.txt
```

Result:

```text
The deployment uploaded index.html

but

missed assets/app.js
```

Inspect the local JavaScript asset:

```bash
cat /opt/challenge/files/app.js
```

Inspect the HTML:

```bash
cat /opt/challenge/files/index.html
```

Evidence:

```html
<script src="/assets/app.js" defer></script>
```

The website expects:

```text
assets/app.js
```

Inspect the S3 bucket:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" \
s3 ls s3://dctf-ses-2587c4027dd6d8c1bf8477ce-site/ \
--recursive
```

Result before the fix:

```text
index.html
```

The JavaScript asset was missing.

## Systematic Elimination

```text
✓ S3 bucket exists
✓ index.html uploaded successfully
✓ Local app.js exists
✓ HTML references assets/app.js
✗ assets/app.js missing from S3
```

The first failed verification was:

```text
The deployment artifact existed locally but had never been uploaded to S3.
```

## First Finding

```text
The deployment itself was incomplete.

The website HTML referenced:

assets/app.js

The JavaScript file existed locally but was absent from the S3 bucket.
```

## Fix

Upload the missing deployment artifact:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" s3 cp \
/opt/challenge/files/app.js \
s3://dctf-ses-2587c4027dd6d8c1bf8477ce-site/assets/app.js \
--content-type application/javascript
```

## Validation

Verify the upload:

```bash
aws --endpoint-url "$AWS_ENDPOINT_URL" \
s3 ls s3://dctf-ses-2587c4027dd6d8c1bf8477ce-site/ \
--recursive
```

Expected result:

```text
index.html

assets/app.js
```

Run the challenge validation.

Read the generated flag:

```bash
cat /home/devops/flag.txt
```

After validation passes, submit the generated flag.

## Lessons Learned

- Static websites commonly deploy to object storage such as Amazon S3.
- HTML often depends on additional static assets such as CSS and JavaScript.
- A deployment can succeed while still being incomplete.
- Always compare local deployment artifacts with deployed artifacts.
- `aws s3 ls --recursive` is useful for verifying bucket contents.
- `aws s3 cp` uploads individual deployment artifacts.
- Correct `Content-Type` metadata should be specified during uploads.

## Engineering Insight

One of the most common deployment failures is assuming:

```text
Deployment completed

↓

Everything was deployed
```

Production deployments should always verify the deployed artifacts rather than assuming the deployment process copied every file.

A typical deployment flow is:

```text
Build

↓

Generate Artifacts

↓

Upload Artifacts

↓

Validate Deployment
```

This challenge failed during:

```text
Upload Artifacts
```

The application itself was healthy.

The deployment process simply omitted one required object.

Good engineers verify the deployment contents after every release.

Another important lesson is understanding cloud object storage.

Unlike a Linux filesystem:

```text
Directory

↓

File
```

S3 stores:

```text
Bucket

↓

Objects

↓

Object Keys
```

The path:

```text
assets/app.js
```

is an object key inside the bucket.

Understanding this difference becomes important when working with:

- Amazon S3
- MinIO
- Floci
- Object Storage
- CDN deployments

## Knowledge Check

### Question 1

Why did the deployed website fail?

A. The Node.js server stopped.

B. The S3 bucket was deleted.

C. The JavaScript deployment artifact was never uploaded.

D. The HTML file was missing.

**Answer:** C

### Question 2

Which command uploaded the missing JavaScript asset?

A.

```bash
aws s3 sync
```

B.

```bash
aws s3 cp /opt/challenge/files/app.js \
s3://<bucket>/assets/app.js \
--content-type application/javascript
```

C.

```bash
scp app.js
```

D.

```bash
cp app.js
```

**Answer:** B
