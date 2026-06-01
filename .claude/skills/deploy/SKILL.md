---
name: deploy
description: Sync the built site to an S3 bucket and optionally invalidate a CloudFront distribution. Usage: /deploy [bucket-name] [--profile aws-profile] [--distribution cloudfront-distribution-id]
disable-model-invocation: true
---

# Deploy to S3

You are executing a deployment of the static site to AWS S3.

## Parse $ARGUMENTS

Extract from $ARGUMENTS:
- `bucket` — S3 bucket name (required; ask if not provided)
- `--profile` — AWS CLI profile to use (optional; omit flag to use default)
- `--distribution` — CloudFront distribution ID for cache invalidation (optional)

## Pre-flight checks

1. Confirm AWS credentials are configured: `aws sts get-caller-identity [--profile <profile>]`
   - If this fails, stop and tell the user to run `/aws-setup` first.
2. Confirm the S3 bucket exists: `aws s3 ls s3://<bucket> [--profile <profile>]`
   - If bucket doesn't exist, ask before creating it.
3. Identify the build output directory (common: `dist/`, `_site/`, `public/`, `out/`, or project root for plain HTML).
   - If unclear, ask the user which directory to sync.

## Deploy

```bash
aws s3 sync <build-dir> s3://<bucket> --delete [--profile <profile>]
```

Report the number of files uploaded/deleted from the command output.

## CloudFront invalidation (if --distribution provided)

```bash
aws cloudfront create-invalidation --distribution-id <distribution-id> --paths "/*" [--profile <profile>]
```

Report the invalidation ID.

## Done

Print a one-line summary: bucket URL or CloudFront domain if known, otherwise the S3 website endpoint.
S3 website endpoint format: `http://<bucket>.s3-website-<region>.amazonaws.com`
