# ☁️ DevSecOps CI/CD Pipeline Security Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Use Case:** CI/CD Pipeline Security Review, DevSecOps Maturity Assessment

---

## Source Code Management (SCM) Security

- [ ] Branch protection rules enforced on `main`/`master` branches
- [ ] Direct pushes to protected branches blocked
- [ ] Mandatory code review (minimum 2 approvals) before merge
- [ ] Signed commits enforced (GPG signing)
- [ ] `.gitignore` reviewed — no secrets, credentials, or environment files committed
- [ ] Pre-commit hooks configured (detect-secrets, gitleaks)
- [ ] Git history scanned for previously committed secrets
- [ ] CODEOWNERS file defined for security-sensitive paths

---

## Secrets Management

- [ ] No secrets hardcoded in source code, Dockerfiles, or CI configs
- [ ] Secrets manager used: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault
- [ ] Environment variables set via CI/CD secret variables (not plaintext)
- [ ] Secrets rotated on schedule and on staff offboarding
- [ ] Secrets scanning in CI pipeline (gitleaks, truffleHog, GitGuardian)
- [ ] Short-lived credentials used for cloud access (OIDC-based, not long-lived keys)
- [ ] IAM roles used instead of access key/secret pairs where possible

---

## SAST — Static Application Security Testing

- [ ] SAST tool integrated into CI pipeline (required gate)
- [ ] SAST runs on every pull request
- [ ] High/Critical findings block merge
- [ ] SAST tool appropriate for language/framework:
  - Java/Kotlin: SpotBugs + Find Security Bugs, Checkmarx
  - Python: Bandit, Semgrep
  - JavaScript/TypeScript: ESLint-security, Semgrep
  - Go: gosec, Semgrep
  - Ruby: Brakeman
- [ ] Custom Semgrep rules for business logic and framework-specific patterns
- [ ] SAST findings triaged and false positives managed

---

## SCA — Software Composition Analysis

- [ ] SCA tool integrated in CI pipeline (OWASP Dependency-Check, Snyk, Dependabot)
- [ ] Known CVEs in direct AND transitive dependencies flagged
- [ ] Critical/High CVEs block deployment
- [ ] License compliance checked (GPL, LGPL flags for commercial products)
- [ ] SBOM (Software Bill of Materials) generated for each release
- [ ] Automated dependency update PRs configured (Dependabot, Renovate)

---

## DAST — Dynamic Application Security Testing

- [ ] DAST tool runs against staging environment on each release candidate
- [ ] OWASP ZAP or Burp Enterprise configured for automated scanning
- [ ] API specification (OpenAPI) fed to DAST for comprehensive coverage
- [ ] DAST findings triaged before production release
- [ ] Regression DAST suite covers previously found vulnerabilities
- [ ] Authenticated scans configured (not just unauthenticated)

---

## Container Security

- [ ] Base images pinned to specific digest (not `latest` tag)
- [ ] Minimal base images used (distroless, alpine)
- [ ] Container images scanned for CVEs (Trivy, Grype, Snyk Container)
- [ ] No secrets in container images or layers
- [ ] `USER` directive — containers don't run as root
- [ ] Read-only filesystem where possible
- [ ] Image signing enforced (cosign, Notary)
- [ ] Container registry access control enforced
- [ ] Allowed image registry allowlist enforced in Kubernetes

**Dockerfile Security Template:**
```dockerfile
FROM node:20-alpine@sha256:<digest>   # Pin to digest
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --chown=appuser:appgroup . .
USER appuser                           # Non-root user
EXPOSE 3000
CMD ["node", "server.js"]
```

---

## Infrastructure as Code (IaC) Security

- [ ] IaC security scanning integrated in CI (Checkov, tfsec, KICS)
- [ ] Terraform plans reviewed before apply (Terraform Cloud, Atlantis)
- [ ] No hardcoded secrets in IaC files
- [ ] Least privilege IAM policies in IaC templates
- [ ] Public exposure (S3 bucket ACL, Security Group 0.0.0.0/0) flagged automatically
- [ ] State files encrypted and stored securely (not in git)
- [ ] IaC drift detection alerts configured

---

## Kubernetes Security

- [ ] PodSecurityAdmission / PodSecurityPolicy enforced
- [ ] Pods don't run as root (`runAsNonRoot: true`)
- [ ] Read-only root filesystem (`readOnlyRootFilesystem: true`)
- [ ] Privilege escalation disabled (`allowPrivilegeEscalation: false`)
- [ ] Capabilities dropped (`drop: ["ALL"]`)
- [ ] Network Policies defined (default deny, explicit allow)
- [ ] RBAC least privilege applied — no `cluster-admin` for workloads
- [ ] Secrets mounted via volume or secret manager (not env vars where avoidable)
- [ ] Admission controllers: OPA Gatekeeper or Kyverno policies enforced
- [ ] etcd encrypted at rest
- [ ] Kubernetes API server not publicly exposed

---

## Pipeline Access Control

- [ ] CI/CD system access controlled (SSO + MFA required)
- [ ] Pipeline secrets not accessible to untrusted branches/PRs
- [ ] Third-party Actions/Plugins pinned to commit SHA (not tag)
- [ ] Self-hosted runners isolated from production network
- [ ] Pipeline logs reviewed — no sensitive data (tokens, passwords) in output
- [ ] Separate service accounts per pipeline with minimal permissions

---

## Deployment & Release Security

- [ ] Staging environment mirrors production (security controls included)
- [ ] Production deployments require approval gate
- [ ] Blue/green or canary deployments enable rollback
- [ ] Post-deployment smoke tests include security regression checks
- [ ] Runtime security monitoring enabled post-deployment (Falco, GuardDuty)
- [ ] Change management process documented and followed

---

## DevSecOps Maturity Levels

| Level | Description |
|---|---|
| Level 1 | Secrets scanning + SAST in CI |
| Level 2 | + SCA + Container scanning + IaC scanning |
| Level 3 | + DAST + SBOM + Pipeline security hardening |
| Level 4 | + Runtime protection + Threat modeling in design + Bug bounty |

---

*Last Updated: 2024 | Reference: OWASP DevSecOps Guideline, CNCF Security Whitepaper*
