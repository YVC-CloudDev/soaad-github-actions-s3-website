# S3 Website Template

A minimal static website with a GitHub Actions workflow that auto-deploys to AWS S3 using OIDC.

Use this repo as a **template** — click **Use this template** on GitHub to create your own copy.

## What's Included

```
├── index.html                     # Single-page website
├── css/style.css                  # Responsive stylesheet
├── js/main.js                     # Scroll-based nav highlighting
└── .github/workflows/
    └── deploy-s3.yml              # GitHub Actions → S3 deploy (OIDC)
```

## Setup

### 1. Create an S3 Bucket

```bash
aws s3api create-bucket --bucket YOUR_BUCKET_NAME --region us-east-1
```

Enable static website hosting:

```bash
aws s3 website s3://YOUR_BUCKET_NAME --index-document index.html
```

Add a **bucket policy** for public read access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

### 2. Set Up OIDC Authentication

1. In the AWS IAM console, create an **Identity Provider** (OpenID Connect):
   - Provider URL: `https://token.actions.githubusercontent.com`
   - Audience: `sts.amazonaws.com`

2. Create an **IAM Role** with a trust policy like:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
        }
      }
    }
  ]
}
```

3. Attach a policy granting S3 access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:DeleteObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    }
  ]
}
```

### 3. Add GitHub Secrets

In your repo, go to **Settings → Secrets and variables → Actions** and add:

| Secret            | Value                                |
| ----------------- | ------------------------------------ |
| `AWS_ROLE_ARN`    | `arn:aws:iam::ACCOUNT_ID:role/ROLE`  |
| `S3_BUCKET_NAME`  | `your-bucket-name`                   |

### 4. Deploy

Push to `main` — the workflow runs automatically and syncs files to your S3 bucket.

Your site will be live at:
```
http://YOUR_BUCKET_NAME.s3-website-us-east-1.amazonaws.com
```

## Making It a Template

On GitHub, go to **Settings → General** and check **Template repository**.

## License

MIT
