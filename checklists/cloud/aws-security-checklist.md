# ☁️ AWS Security Checklist

> **Level:** Professional / Senior AppSec / Cloud Security Engineer
> **Use Case:** AWS Account Security Review, Cloud Pentest, Compliance Audit
> **Reference:** CIS AWS Foundations Benchmark v2.0, AWS Well-Architected Security Pillar

---

## Identity & Access Management (IAM)

- [ ] Root account MFA enabled
- [ ] Root account not used for daily operations
- [ ] Root account access keys deleted
- [ ] All IAM users have MFA enabled
- [ ] IAM password policy enforced (14+ chars, complexity, rotation)
- [ ] No long-lived access keys for human users (use SSO/Federation instead)
- [ ] Access keys rotated within 90 days
- [ ] IAM roles used for EC2/Lambda/ECS instead of embedded credentials
- [ ] `sts:AssumeRole` restricted to necessary principals
- [ ] Inline policies avoided; managed policies attached
- [ ] Permissions reviewed with IAM Access Analyzer
- [ ] AWS Organizations SCPs deployed to restrict dangerous actions
- [ ] `NotAction` / `*` resource in IAM policies flagged and reviewed

---

## Logging & Monitoring

- [ ] CloudTrail enabled in all regions
- [ ] CloudTrail log file validation enabled
- [ ] CloudTrail logs delivered to S3 bucket with Object Lock (WORM)
- [ ] S3 bucket for CloudTrail logs is not publicly accessible
- [ ] CloudTrail logs shipped to centralized SIEM
- [ ] VPC Flow Logs enabled for all VPCs
- [ ] AWS Config enabled and recording all resource changes
- [ ] GuardDuty enabled in all accounts and regions
- [ ] Security Hub enabled and aggregating findings
- [ ] CloudWatch alarms for: root account usage, MFA disable, security group changes, IAM policy changes, unauthorized API calls
- [ ] Access Analyzer findings reviewed and resolved

---

## Networking

- [ ] Default VPC deleted or unused (create custom VPC)
- [ ] Security Groups follow least-privilege (no `0.0.0.0/0` inbound on sensitive ports)
- [ ] SSH (22) and RDP (3389) not open to `0.0.0.0/0`
- [ ] NACLs as defense-in-depth layer
- [ ] VPC endpoints used for AWS services (S3, DynamoDB, SSM) — avoid internet exposure
- [ ] Resources in private subnets; NAT Gateway used for outbound
- [ ] WAF attached to ALB / CloudFront / API Gateway
- [ ] AWS Shield Standard active; AWS Shield Advanced for DDoS-critical workloads
- [ ] VPC Peering connections reviewed and necessary only

---

## S3 Bucket Security

- [ ] S3 Block Public Access enabled at account level and bucket level
- [ ] Bucket ACLs not granting public access
- [ ] S3 bucket policies reviewed — no `Principal: "*"` without conditions
- [ ] Server-side encryption enabled (SSE-S3 or SSE-KMS)
- [ ] S3 versioning enabled for critical buckets
- [ ] S3 Object Lock configured for audit/compliance buckets
- [ ] S3 Access Logging enabled for sensitive buckets
- [ ] Lifecycle policies defined for data retention/deletion
- [ ] Cross-account access reviewed and justified

---

## Compute Security (EC2)

- [ ] EC2 instances use IMDSv2 only (`HttpTokens: required`)
- [ ] No EC2 key pairs in use for production (use SSM Session Manager)
- [ ] EC2 instances managed via AWS Systems Manager (SSM)
- [ ] Security patches applied (AWS Patch Manager)
- [ ] EBS volumes encrypted at rest
- [ ] Instance profiles assigned (not hardcoded credentials)
- [ ] Unnecessary ports not open in security groups

---

## Serverless / Lambda Security

- [ ] Lambda execution role follows least privilege
- [ ] Lambda environment variables don't contain plaintext secrets (use SSM Parameter Store or Secrets Manager)
- [ ] Lambda VPC configuration if accessing private resources
- [ ] Lambda function URLs authenticated (not publicly open without auth)
- [ ] Lambda layers reviewed for third-party code supply chain risk
- [ ] Lambda code signed (CodeSigning config)

---

## Database Security (RDS / DynamoDB)

- [ ] RDS instances not publicly accessible
- [ ] RDS encryption at rest enabled (KMS)
- [ ] RDS encryption in transit enforced (SSL/TLS)
- [ ] Automated backups enabled
- [ ] RDS Multi-AZ for production databases
- [ ] Database master credentials in Secrets Manager with rotation
- [ ] RDS Enhanced Monitoring enabled
- [ ] DynamoDB encryption at rest enabled
- [ ] DynamoDB Point-in-Time Recovery (PITR) enabled

---

## KMS & Secrets Management

- [ ] AWS KMS used for encryption key management
- [ ] KMS key rotation enabled (annual for customer-managed keys)
- [ ] KMS key policies restrict access to least privilege
- [ ] Secrets Manager used for application secrets (not SSM Parameter Store plain strings)
- [ ] Secrets Manager automatic rotation configured
- [ ] No secrets in CloudFormation templates, Lambda env vars, or EC2 user data

---

## Incident Response Readiness

- [ ] AWS Config rules cover compliance baselines
- [ ] Automated remediation Lambda functions deployed for common misconfigurations
- [ ] AWS Security Hub automated response playbooks configured
- [ ] IR runbooks documented for: compromised IAM key, EC2 breach, S3 data exposure
- [ ] AWS Backup plan in place for critical resources
- [ ] Recovery Time Objective (RTO) and Recovery Point Objective (RPO) defined

---

## AWS Security Tooling Reference

| Tool | Purpose |
|---|---|
| AWS Security Hub | Centralized findings aggregation |
| AWS GuardDuty | Threat detection (CloudTrail, DNS, VPC Flow Logs) |
| AWS Config | Resource configuration compliance |
| AWS Access Analyzer | IAM policy analysis |
| AWS Inspector | EC2/Container vulnerability scanning |
| Prowler | CIS benchmark audit CLI tool |
| ScoutSuite | Multi-cloud security auditing |
| CloudMapper | AWS network visualization |

---

*Last Updated: 2024 | Reference: CIS AWS Foundations Benchmark v2.0*
