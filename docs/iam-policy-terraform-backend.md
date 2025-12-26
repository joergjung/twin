# IAM Policy for Terraform Backend

If your IAM user needs to create the DynamoDB table and S3 bucket for Terraform state management, attach this policy:

## Minimal Policy (if table/bucket already exist)

This policy allows Terraform to use existing backend resources:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "TerraformStateS3",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::twin-terraform-state-*",
        "arn:aws:s3:::twin-terraform-state-*/*"
      ]
    },
    {
      "Sid": "TerraformStateDynamoDB",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:eu-central-1:*:table/twin-terraform-locks"
      ]
    }
  ]
}
```

## Full Policy (to create backend resources)

If you need to create the backend resources (S3 bucket and DynamoDB table), use this expanded policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "TerraformStateS3",
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:ListBucket",
        "s3:GetBucketVersioning",
        "s3:PutBucketVersioning",
        "s3:GetBucketEncryption",
        "s3:PutBucketEncryption",
        "s3:GetBucketPublicAccessBlock",
        "s3:PutBucketPublicAccessBlock",
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::twin-terraform-state-*",
        "arn:aws:s3:::twin-terraform-state-*/*"
      ]
    },
    {
      "Sid": "TerraformStateDynamoDB",
      "Effect": "Allow",
      "Action": [
        "dynamodb:CreateTable",
        "dynamodb:DescribeTable",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:DeleteItem",
        "dynamodb:TagResource"
      ],
      "Resource": [
        "arn:aws:dynamodb:eu-central-1:*:table/twin-terraform-locks"
      ]
    }
  ]
}
```

## How to Apply

### Option 1: Using AWS Console
1. Go to IAM → Users → `aiengineer`
2. Click "Add permissions" → "Create inline policy"
3. Paste the JSON policy above
4. Review and create

### Option 2: Using AWS CLI (requires admin access)
```bash
aws iam put-user-policy \
  --user-name aiengineer \
  --policy-name TerraformBackendPolicy \
  --policy-document file://terraform-backend-policy.json
```

## Recommended Approach

**Best practice**: Have an AWS administrator create the S3 bucket and DynamoDB table once using the `create-dynamodb-lock-table.sh` script, then use the minimal policy for your IAM user. This follows the principle of least privilege.

