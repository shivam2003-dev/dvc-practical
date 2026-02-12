# DVC with AWS S3 Setup

This guide explains how to configure DVC remote storage on AWS S3 for this project.

## 1. Prerequisites

- AWS account with permission to create/use S3 buckets
- AWS CLI installed (`aws --version`)
- Python + DVC installed

Install DVC with S3 support:

```bash
pip install "dvc[s3]"
```

If DVC is already installed without extras:

```bash
pip install boto3 s3fs
```

## 2. Create S3 Bucket

Use either AWS Console or CLI.

Example (CLI):

```bash
aws s3 mb s3://dvc-practical-storage --region us-east-1
```

Recommended bucket settings:
- Enable versioning
- Enable default encryption (SSE-S3 or SSE-KMS)
- Block public access

## 3. Configure AWS Credentials

Choose one method:

### Option A: AWS CLI profile (recommended for local development)

```bash
aws configure --profile dvc-user
```

Then set:
- AWS Access Key ID
- AWS Secret Access Key
- Default region

### Option B: Environment variables

```bash
export AWS_ACCESS_KEY_ID="<your-access-key>"
export AWS_SECRET_ACCESS_KEY="<your-secret-key>"
export AWS_DEFAULT_REGION="us-east-1"
```

### Option C: IAM Role (recommended for EC2/ECS/EKS)

Attach an IAM role to compute and avoid static keys entirely.

## 4. Add S3 as DVC Remote

From repository root:

```bash
dvc remote add -d myremote s3://dvc-practical-storage/customer-data
```

If using a named AWS profile:

```bash
dvc remote modify myremote profile dvc-user
```

Optional: custom endpoint for S3-compatible storage (MinIO, etc.)

```bash
dvc remote modify myremote endpointurl https://s3.your-endpoint.example
```

## 5. Verify Configuration

Check remote config:

```bash
dvc remote list
cat .dvc/config
```

Push tracked data:

```bash
dvc push
```

Test pull flow (simulate clean workspace):

```bash
rm -f data/customer.csv
dvc pull
```

## 6. Team Workflow

When data changes:

```bash
python3 src/data_ingestion.py
dvc add data/customer.csv
git add data/customer.csv.dvc
git commit -m "Update customer dataset version"
git push origin main
dvc push
```

When teammates sync:

```bash
git pull origin main
dvc pull
```

## 7. Minimal IAM Policy Example

Use least privilege for the exact bucket/prefix:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": "arn:aws:s3:::dvc-practical-storage"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::dvc-practical-storage/customer-data/*"
    }
  ]
}
```

## 8. Troubleshooting

- `Missing credentials`
  - Run `aws sts get-caller-identity` to verify AWS auth.
  - Ensure the correct profile is configured in DVC (`dvc remote modify myremote profile <name>`).
- `AccessDenied`
  - Validate IAM policy bucket ARN and prefix.
  - Confirm bucket policy is not blocking your IAM principal.
- `Region mismatch` or slow uploads
  - Configure `AWS_DEFAULT_REGION` to match bucket region.
- `dvc push` is slow
  - Use a region near compute and avoid unnecessary large-file churn.

## 9. Security Best Practices

- Never commit access keys to Git.
- Prefer IAM roles over long-lived keys.
- Rotate credentials periodically.
- Keep bucket private and encrypted.
- Scope IAM policy to one bucket/prefix used by DVC.
