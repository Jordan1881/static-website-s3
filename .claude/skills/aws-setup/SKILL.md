---
name: aws-setup
description: Step-by-step guide for configuring AWS credentials, creating an S3 static-hosting bucket, and optionally wiring up CloudFront. Run this before /deploy if credentials aren't configured yet.
---

# AWS Static Hosting Setup

Walk the user through each step below in order. Check the current state before suggesting actions — don't redo steps already completed.

## Step 1: AWS credentials

Check: `aws sts get-caller-identity`

If it fails:
1. Ask whether they want to use an IAM user (access key/secret) or SSO.
2. **IAM user**: guide them through `aws configure` — they'll need Access Key ID, Secret Access Key, default region (e.g. `us-east-1`), and output format (`json`).
3. **SSO**: guide them through `aws configure sso`.

Verify credentials work before continuing.

## Step 2: S3 bucket

Ask for:
- Bucket name (must be globally unique; using the domain name is conventional, e.g. `example.com`)
- AWS region (default: `us-east-1`)

Create the bucket:
```bash
aws s3api create-bucket --bucket <name> --region <region> [--create-bucket-configuration LocationConstraint=<region>]
```
Note: omit `--create-bucket-configuration` for `us-east-1`.

Enable static website hosting:
```bash
aws s3 website s3://<name>/ --index-document index.html --error-document 404.html
```

Set a public-read bucket policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::<bucket-name>/*"
  }]
}
```
Apply: `aws s3api put-bucket-policy --bucket <name> --policy file://bucket-policy.json`
Clean up the temp file after applying.

Note the website endpoint: `http://<name>.s3-website-<region>.amazonaws.com`

## Step 3: CloudFront (optional)

Ask if they want CloudFront (HTTPS, custom domain, CDN caching). If yes:

Create a distribution pointing to the S3 website endpoint (not the REST endpoint):
- Origin domain: `<bucket>.s3-website-<region>.amazonaws.com`
- Default root object: `index.html`
- Price class: `PriceClass_100` (US/Europe only) or ask

Provide the CLI command or note that the AWS console is easier for first-time setup.

After creation, note the CloudFront domain (`*.cloudfront.net`) — this is the HTTPS URL until a custom domain is added.

## Step 4: Summary

Print:
- AWS profile/region in use
- S3 bucket name and website endpoint
- CloudFront distribution ID and domain (if created)
- Next step: run `/deploy <bucket-name>` to push the site
