# Helix Unified Cloud — AWS Real-Time Provisioning Guide

## Overview

This guide explains how to connect the Helix Unified Cloud portal to your real AWS environment for live account provisioning via Terraform and AWS Organizations.

---

## Prerequisites Checklist

- [ ] AWS Organizations enabled on your management account
- [ ] `HelixVendingRunnerRole` IAM role created in management account
- [ ] S3 bucket + DynamoDB table for Terraform state created
- [ ] Terraform installed on runner host/container
- [ ] FastAPI backend deployed and reachable from this portal
- [ ] `TARGET_OU_ID` set to your real OU ID

---

## 1. IAM Role for the Vending Runner

Create this role in your **management account**:

```hcl
resource "aws_iam_role" "vending_runner" {
  name = "HelixVendingRunnerRole"
  assume_role_policy = jsonencode({
    Statement = [{
      Effect    = "Allow"
      Principal = { AWS = "arn:aws:iam::<runner-account-id>:root" }
      Action    = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy" "vending_runner" {
  role = aws_iam_role.vending_runner.id
  policy = jsonencode({
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "organizations:CreateAccount",
          "organizations:DescribeCreateAccountStatus",
          "organizations:MoveAccount",
          "organizations:TagResource"
        ]
        Resource = "*"
      }
    ]
  })
}
```

---

## 2. Terraform State Backend (One-Time Setup)

Run this once to create the shared state infrastructure:

```hcl
resource "aws_s3_bucket" "tf_state" {
  bucket = "helix-tf-state-prod"

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "aws:kms"
      }
    }
  }
}

resource "aws_dynamodb_table" "tf_locks" {
  name         = "helix-tf-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

---

## 3. FastAPI Backend Deployment

The FastAPI service is the real engine that replaces the simulated provisioning in the portal.

### Deployment Options

| Option | How |
|---|---|
| **Local test** | `uvicorn app.main:app --reload` |
| **AWS ECS Fargate** | Push Docker image to ECR, run as Fargate service |
| **AWS Lambda** | Use `Mangum` adapter for FastAPI on Lambda |

### Required Environment Variables

```bash
AWS_MANAGEMENT_ACCOUNT_ID=123456789012
AWS_VENDING_ROLE_ARN=arn:aws:iam::123456789012:role/HelixVendingRunnerRole
TF_STATE_BUCKET=helix-tf-state-prod
TF_LOCK_TABLE=helix-tf-locks
TARGET_OU_ID=ou-xxxx-xxxxxxxx   # from your AWS Org
```

---

## 4. Terraform Account Module

```hcl
# terraform/account/main.tf

variable "team_name"        {}
variable "env"              {}
variable "system_id"        {}
variable "poc"              {}
variable "target_ou_id"     {}
variable "vending_role_arn" {}

terraform {
  backend "s3" {
    bucket         = "helix-tf-state-prod"
    key            = "accounts/${var.team_name}-${var.env}.tfstate"
    region         = "us-east-1"
    dynamodb_table = "helix-tf-locks"
    encrypt        = true
  }
}

provider "aws" {
  assume_role {
    role_arn = var.vending_role_arn
  }
}

resource "aws_organizations_account" "this" {
  name      = "gw-${var.team_name}-${var.env}-001"
  email     = "gw_${var.team_name}_${var.env}_001_aws_acctmgmt@htl.com"
  parent_id = var.target_ou_id
  role_name = "OrganizationAccountAccessRole"

  tags = {
    TeamName  = var.team_name
    Env       = var.env
    SystemID  = var.system_id
    POC       = var.poc
    ManagedBy = "helix-unified-cloud"
  }
}

output "account_id" {
  value = aws_organizations_account.this.id
}
```

---

## 5. FastAPI Terraform Runner Service

```python
# services/terraform.py
import asyncio, os, json

async def provision(request: dict) -> str:
    workdir = f"/tmp/tf/{request['id']}"
    os.makedirs(workdir, exist_ok=True)

    # Write tfvars
    with open(f"{workdir}/terraform.tfvars.json", "w") as f:
        json.dump({
            "team_name":        request["team_name"],
            "env":              request["env"],
            "system_id":        request["system_id"],
            "poc":              request["poc"],
            "target_ou_id":     os.environ["TARGET_OU_ID"],
            "vending_role_arn": os.environ["AWS_VENDING_ROLE_ARN"],
        }, f)

    # Run Terraform steps
    for cmd in [
        ["terraform", "init"],
        ["terraform", "plan", "-out=tfplan"],
        ["terraform", "apply", "-auto-approve", "tfplan"],
    ]:
        proc = await asyncio.create_subprocess_exec(
            *cmd, cwd=workdir,
            stdout=asyncio.subprocess.PIPE,
            stderr=asyncio.subprocess.STDOUT,
        )
        stdout, _ = await proc.communicate()
        log = stdout.decode()

        # Stream log back to DB / WebSocket here
        if proc.returncode != 0:
            raise Exception(f"Terraform failed:\n{log}")

    # Read Terraform output to get account ID
    out = await asyncio.create_subprocess_exec(
        "terraform", "output", "-json", cwd=workdir,
        stdout=asyncio.subprocess.PIPE
    )
    stdout, _ = await out.communicate()
    return json.loads(stdout)["account_id"]["value"]
```

---

## 6. Connecting the Portal to the Real API

The portal currently simulates provisioning. To connect to your deployed FastAPI:

### Step 1 — Deploy FastAPI (ECS or Lambda)

### Step 2 — Replace the simulate call in `pages/Requests.jsx`

```js
// Replace simulation with real API call:
const res = await fetch(
  `https://your-api.internal/api/requests/${req.id}/provision`,
  {
    method: 'POST',
    headers: { Authorization: `Bearer ${token}` }
  }
);
const { aws_account_id } = await res.json();
```

### Step 3 — Poll for status

```js
// Poll every 5 seconds until status is "active" or "failed"
const poll = setInterval(async () => {
  const r = await fetch(`https://your-api.internal/api/requests/${req.id}`);
  const data = await r.json();
  if (['active', 'failed'].includes(data.status)) {
    clearInterval(poll);
    refetch();
  }
}, 5000);
```

---

## 7. Quick Local Test (No ECS Needed)

```bash
# 1. Install dependencies
pip install fastapi uvicorn boto3 python-jose

# 2. Set environment variables
export AWS_PROFILE=your-management-account-profile
export TARGET_OU_ID=ou-xxxx-xxxxxxxx
export AWS_VENDING_ROLE_ARN=arn:aws:iam::123456789012:role/HelixVendingRunnerRole

# 3. Run the API
uvicorn app.main:app --reload --port 8000

# 4. Test with curl
curl -X POST http://localhost:8000/api/requests \
  -H "Content-Type: application/json" \
  -d '{
    "team_name": "platform",
    "env": "dev",
    "system_id": "SYS-001",
    "poc": "admin@htl.com"
  }'
```

---

## 8. Account Naming Rules

| Field | Pattern | Example |
|---|---|---|
| Account Name | `gw-<team>-<env>-001` | `gw-platform-dev-001` |
| Account Email | `gw_<team>_<env>_001_aws_acctmgmt@htl.com` | `gw_platform_dev_001_aws_acctmgmt@htl.com` |

---

## 9. Security Recommendations

- Store all secrets in **AWS Secrets Manager** (DB creds, Entra client secret)
- Use **KMS CMKs** for S3 state bucket, RDS, and DynamoDB encryption
- Place the API in **private subnets** behind an internal ALB + WAF
- Use **VPC endpoints** for S3 and DynamoDB access (no public internet)
- Enforce **DynamoDB state locking** to prevent concurrent Terraform runs
- Require **plan review** before apply for Approver role (add approval step)
- Log every state transition to **CloudWatch Logs** + optional SIEM stream

---

## 10. RBAC Roles

| Role | Permissions |
|---|---|
| **Requester** | Submit requests, view own requests |
| **Approver** | Approve + trigger provisioning, view all requests |
| **Admin** | Everything + audit log, user management |

---

## Architecture Diagram

```
Browser (SSO)
    │  OIDC (Entra ID)
    ▼
React Portal (Helix Unified Cloud)
    │  JWT
    ▼
FastAPI (ECS Fargate / Lambda)
    ├──▶ PostgreSQL RDS        (requests, accounts, audit_log)
    ├──▶ Terraform Runner      (CodeBuild / ECS Task)
    │         ├──▶ S3          (gw-tf-state-prod — tfstate files)
    │         ├──▶ DynamoDB    (helix-tf-locks — state locking)
    │         └──▶ AWS Organizations  (CreateAccount API)
    └──▶ CloudWatch            (audit logs, metrics)
```

---

*Generated by Helix Unified Cloud Portal*
