# 🚨 Incident Response Playbook — Cloud Misconfiguration

> **Severity:** Critical–High
> **CWE:** CWE-732, CWE-284, CWE-522
> **Applies To:** AWS, Azure, GCP
> **Playbook Version:** 1.0

---

## Overview

Cloud misconfigurations are the leading cause of cloud security breaches. Common scenarios include publicly exposed S3 buckets, overly permissive IAM policies, exposed databases, and open security groups allowing unauthorized access.

---

## Common Misconfiguration Scenarios

| Scenario | Risk |
|---|---|
| Public S3 bucket with sensitive data | Data breach |
| Security group 0.0.0.0/0 on SSH/RDP | Unauthorized server access |
| IAM user with AdministratorAccess | Full cloud compromise |
| RDS instance publicly accessible | Database breach |
| Lambda function URL without auth | Unauthorized function execution |
| EC2 with IMDSv1 enabled | SSRF → credential theft |
| Secrets in EC2 user data | Credential exposure |
| Public ECR/DockerHub image with secrets | Supply chain + credential exposure |

---

## Phase 1 — Detection & Triage

**Sources of Detection:**
- AWS Security Hub / GuardDuty finding
- AWS Config rule violation
- Prowler / ScoutSuite scan
- External researcher disclosure
- Bug bounty report
- News report / Twitter disclosure

**Triage Questions:**
1. What resource is misconfigured?
2. Is it publicly accessible right now?
3. Does it contain sensitive data?
4. Has it been accessed by unauthorized parties?
5. How long has it been misconfigured? (Check creation date vs config change)

---

## Phase 2 — Containment (By Type)

**⏱ Target Time: 0–15 minutes for critical (public data exposure)**

### Exposed S3 Bucket
```bash
# Step 1: Remove public access IMMEDIATELY
aws s3api put-public-access-block \
  --bucket BUCKET-NAME \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Step 2: Remove bucket policy if granting public access
aws s3api delete-bucket-policy --bucket BUCKET-NAME

# Step 3: Check what was accessed
aws s3api get-bucket-logging --bucket BUCKET-NAME
```

### Open Security Group (SSH/RDP to 0.0.0.0/0)
```bash
# Remove inbound rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-XXXXXXXX \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Add restrictive rule (corporate IP only)
aws ec2 authorize-security-group-ingress \
  --group-id sg-XXXXXXXX \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/32
```

### Compromised IAM Credentials
```bash
# Disable access key immediately
aws iam update-access-key \
  --access-key-id AKIAXXXXXXXXXXXXXXXX \
  --status Inactive \
  --user-name username

# Attach deny-all policy to user while investigating
aws iam put-user-policy \
  --user-name username \
  --policy-name EMERGENCY-DENY-ALL \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Deny","Action":"*","Resource":"*"}]}'
```

### Exposed RDS Instance
```bash
# Modify to not publicly accessible
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --no-publicly-accessible \
  --apply-immediately
```

---

## Phase 3 — Investigation

### S3 Data Access Investigation
```bash
# Check S3 server access logs (if enabled)
aws s3 cp s3://logging-bucket/access-logs/ ./logs/ --recursive

# Check CloudTrail for GetObject events on bucket
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceName,AttributeValue=BUCKET-NAME \
  --start-time 2024-01-01 \
  --end-time 2024-01-31 \
  | jq '.Events[] | select(.EventName == "GetObject")'

# Check if bucket was listed in public bucket scanners (GrayhatWarfare, etc.)
```

### IAM Credential Compromise Investigation
```bash
# Get last activity for access key
aws iam get-access-key-last-used --access-key-id AKIAXXXXXXXXXXXXXXXX

# Get CloudTrail events for this key
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIAXXXXXXXXXXXXXXXX

# Check for IAM actions (new users, policy changes, new keys created)
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser

# Check for new resources created
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances
```

### Key Investigation Questions
- [ ] What data was accessed and by whom?
- [ ] Were any new IAM users, roles, or access keys created?
- [ ] Were any new EC2 instances, Lambda functions, or other resources created?
- [ ] Was any data exfiltrated (unusual S3 GetObject, large data transfers)?
- [ ] Were any credentials or secrets exposed in the misconfigured resource?
- [ ] Was there lateral movement to other AWS accounts/services?

---

## Phase 4 — Eradication & Remediation

### Preventive Controls
- [ ] Enable S3 Block Public Access at **account level** (not just bucket)
- [ ] Enable AWS Config rules for public bucket detection
- [ ] Configure GuardDuty (catches anomalous S3 access)
- [ ] Enforce IMDSv2 via SCP or instance launch template
- [ ] Enable AWS Security Hub with CIS AWS benchmark

### IaC Prevention (Terraform)
```hcl
# Enforce S3 Block Public Access via Terraform
resource "aws_s3_bucket_public_access_block" "example" {
  bucket = aws_s3_bucket.example.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Checkov rule: CKV_AWS_53, CKV_AWS_54, CKV_AWS_55, CKV_AWS_56
```

### Service Control Policy (SCP) — Prevent Public S3
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyPublicS3",
      "Effect": "Deny",
      "Action": [
        "s3:PutBucketAcl",
        "s3:PutObjectAcl"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": ["public-read", "public-read-write", "authenticated-read"]
        }
      }
    }
  ]
}
```

---

## Phase 5 — Post-Incident

### Notifications
- [ ] CISO notified
- [ ] Legal / DPO if PII was exposed
- [ ] Regulatory notification if required (GDPR 72hr, HIPAA 60 days)
- [ ] Affected users if their data was exposed

### Documentation
- [ ] Timeline of misconfiguration and discovery
- [ ] Data inventory of what was exposed
- [ ] Evidence of unauthorized access (or confirmation of no access)
- [ ] Remediation actions taken
- [ ] RCA: How did this misconfiguration occur? (Manual change, IaC drift, no guardrails)

### Systemic Hardening
- [ ] Implement IaC scanning (Checkov, tfsec) in CI/CD
- [ ] Enable AWS Config rules across all accounts
- [ ] Implement automated remediation Lambdas for common misconfigs
- [ ] Schedule quarterly Prowler / ScoutSuite scans
- [ ] Add cloud security training for engineering teams

---

*Playbook Version: 1.0 | Reference: AWS Security Incident Response Guide, CIS AWS Benchmark*
