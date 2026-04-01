# 🚨 Incident Response Playbook — Secrets / Credential Exposure

> **Severity:** Critical
> **CWE:** CWE-312, CWE-798, CWE-259
> **Applies To:** API Keys, OAuth Tokens, Database Credentials, Private Keys, JWT Secrets
> **Playbook Version:** 1.0

---

## Overview

Secrets exposure occurs when sensitive credentials (API keys, tokens, passwords, private keys) are accidentally leaked via public GitHub repos, container images, application logs, error messages, or public cloud storage.

**Golden Rule:** Treat every exposed secret as **already compromised**. Rotate first, investigate second.

---

## Detection Sources

- GitHub secret scanning alerts
- GitGuardian / truffleHog scan results
- Researcher disclosure / bug bounty
- Shodan / Censys scan exposure
- Employee accidentally sharing screenshot
- Public Docker Hub image scan
- CloudTrail: API key used from unexpected IP/geo

---

## Phase 1 — IMMEDIATE RESPONSE (< 15 minutes)

> ⚠️ Do NOT spend time investigating before revoking. Revoke immediately.

### Step 1: Identify Secret Type
| Secret Type | Revocation Method |
|---|---|
| AWS Access Key | IAM Console → Delete/Disable Key |
| GitHub Personal Access Token | GitHub Settings → Revoke |
| Google Service Account Key | GCP Console → Disable Key |
| Stripe/Payment API Key | Payment provider dashboard → Roll Key |
| Database Password | DB console → Change Password |
| JWT Secret | Application config → Rotate + redeploy |
| SSH Private Key | Remove from authorized_keys on all servers |
| TLS Private Key | Revoke certificate (CA), reissue |

### Step 2: Revoke the Secret

**AWS IAM Key:**
```bash
# DISABLE (preserves for forensics, stops use)
aws iam update-access-key \
  --access-key-id AKIAXXXXXXXXXXXXXXXX \
  --status Inactive \
  --user-name <username>

# After forensics — DELETE permanently
aws iam delete-access-key \
  --access-key-id AKIAXXXXXXXXXXXXXXXX \
  --user-name <username>
```

**GitHub Token (via API):**
```bash
# Revoke via GitHub API
curl -X DELETE \
  -H "Authorization: token <admin_token>" \
  https://api.github.com/applications/<client_id>/token \
  -d '{"access_token": "<token_to_revoke>"}'
```

**Database Password:**
```sql
-- PostgreSQL
ALTER USER appuser PASSWORD 'new_strong_password_from_vault';

-- MySQL
ALTER USER 'appuser'@'%' IDENTIFIED BY 'new_strong_password_from_vault';
FLUSH PRIVILEGES;
```

### Step 3: Remove from Exposure Source
```bash
# If in Git history — remove with git-filter-repo (NOT git filter-branch)
pip install git-filter-repo
git filter-repo --path-glob '*.env' --invert-paths
git filter-repo --replace-text <(echo "AKIAXXXXXXXX==>REDACTED")

# Force push (coordinate with team)
git push origin --force --all
git push origin --force --tags

# Note: GitHub will still show in cached views — contact GitHub support to purge
```

---

## Phase 2 — Assessment (15–60 minutes)

- [ ] Determine when secret was first exposed (git log, docker history, etc.)
- [ ] Calculate exposure window (first exposure → revocation)
- [ ] Identify all locations where secret was exposed (GitHub, Docker Hub, Pastebin, etc.)
- [ ] Check if secret was already used by unauthorized parties

### AWS Key Investigation
```bash
# Check last use
aws iam get-access-key-last-used --access-key-id AKIAXXXXXXXXXXXXXXXX

# Get all API calls made with this key via CloudTrail
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIAXXXXXXXXXXXXXXXX \
  --max-results 50 | jq '.Events[] | {time: .EventTime, event: .EventName, region: .CloudTrailEvent | fromjson | .awsRegion}'

# Critical: Check for new IAM users, EC2 instances, S3 changes
EVENTS_TO_CHECK="CreateUser CreateAccessKey RunInstances PutBucketPolicy CreateRole AttachRolePolicy"
for EVENT in $EVENTS_TO_CHECK; do
  echo "=== $EVENT ==="
  aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=EventName,AttributeValue=$EVENT \
    --start-time $(date -d '30 days ago' --iso-8601=seconds)
done
```

---

## Phase 3 — Scope of Compromise

Investigate based on secret type:

### If AWS Key Compromised
- [ ] New IAM users or roles created?
- [ ] New EC2 instances launched (cryptomining)?
- [ ] S3 data exfiltrated?
- [ ] CloudTrail disabled / tampered with?
- [ ] New Lambda functions created?
- [ ] SES used for spam?
- [ ] Check all regions (attackers expand beyond initial region)

### If Database Password Compromised
- [ ] DB access logs for unauthorized reads/writes
- [ ] Data tables accessed — are they PII, financial, health data?
- [ ] Any SQL dumps executed?
- [ ] New DB users created?

### If JWT Secret Compromised
- [ ] Forged tokens possible for entire exposure window
- [ ] All tokens from exposure window must be considered invalid
- [ ] Force re-login for ALL users

---

## Phase 4 — Eradication & Hardening

### Replace with Secrets Manager
```python
# BEFORE (hardcoded — vulnerable)
DB_PASSWORD = "MyP@ssw0rd123"
API_KEY = "sk_live_XXXXXXXXXXXX"

# AFTER (AWS Secrets Manager)
import boto3
import json

def get_secret(secret_name: str) -> dict:
    client = boto3.client('secretsmanager', region_name='us-east-1')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

secrets = get_secret('prod/myapp/db')
DB_PASSWORD = secrets['password']
```

### Pre-commit Hook (gitleaks)
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

```bash
# Install and run
pre-commit install
pre-commit run --all-files
```

### GitHub Actions Secret Scanning
```yaml
# Automatically enabled for public repos
# For private repos: Settings → Code security → Secret scanning → Enable
```

---

## Phase 5 — Post-Incident

### Notification Requirements
| Exposure Type | Notify |
|---|---|
| User PII accessed | DPO + Legal + Affected users + Regulator |
| Payment data | PCI DSS incident response, acquiring bank |
| Health data | HIPAA breach notification rules apply |
| Internal secrets only | CISO, Engineering Lead |

### Documentation
- [ ] Timeline: when created → when exposed → when detected → when revoked
- [ ] Full list of resources/actions performed with compromised secret
- [ ] Evidence of unauthorized vs authorized use
- [ ] Impact assessment

### Systemic Changes
- [ ] Pre-commit hooks (gitleaks) mandatory for all repos
- [ ] GitHub secret scanning enabled on all repos
- [ ] Secrets manager adoption tracked as engineering KPI
- [ ] CI/CD secret injection audit (ensure no plaintext in configs)
- [ ] SAST rules for hardcoded secrets (Semgrep)
- [ ] Developer training: secrets hygiene

---

## Quick Reference: Rotation Runbook

| Secret | Rotation Tool | Where to Store After |
|---|---|---|
| AWS Access Keys | IAM Console | Never — use IAM Roles |
| Database passwords | DB console / Secrets Manager rotation | AWS Secrets Manager |
| API Keys (3rd party) | Provider dashboard | AWS Secrets Manager |
| JWT signing secret | App config | AWS Secrets Manager / Parameter Store |
| TLS private keys | CA / Let's Encrypt | AWS Certificate Manager |
| SSH Keys | Generate new keypair | AWS Systems Manager / Vault |

---

*Playbook Version: 1.0 | Reference: OWASP Secrets Management Cheat Sheet*
