# 🌐 OWASP Top 10 Web Application Security Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Standard:** OWASP Top 10 (2021)
> **Use Case:** Security Assessment, Code Review, Pre-deployment Gate

---

## How to Use This Checklist

| Symbol | Meaning |
|---|---|
| ✅ | Verified / Implemented |
| ❌ | Not Implemented / Vulnerable |
| ⚠️ | Partial / Needs Review |
| N/A | Not Applicable |

---

## A01 — Broken Access Control

> CWE-284, CWE-285, CWE-639

- [ ] Deny by default — all resources require explicit access grants
- [ ] Access control checks enforced server-side, not client-side
- [ ] Vertical privilege escalation tested (user → admin)
- [ ] Horizontal privilege escalation tested (user A → user B data)
- [ ] IDOR (Insecure Direct Object References) tested on all object IDs
- [ ] JWT/session tokens validated for authorization on every request
- [ ] Directory listing disabled on web server
- [ ] Sensitive file access restricted (`.git`, `.env`, `backup.zip`, etc.)
- [ ] CORS policy is restrictive — `Access-Control-Allow-Origin` not set to `*` for sensitive APIs
- [ ] Admin panels not publicly accessible

**References:** [CWE-284](https://cwe.mitre.org/data/definitions/284.html) | [OWASP A01:2021](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)

---

## A02 — Cryptographic Failures

> CWE-311, CWE-327, CWE-328

- [ ] Sensitive data (PII, passwords, credit cards) not stored in plaintext
- [ ] Passwords hashed using bcrypt / Argon2 / scrypt (NOT MD5/SHA1)
- [ ] TLS 1.2+ enforced; TLS 1.0/1.1 and SSL disabled
- [ ] HSTS header set (`Strict-Transport-Security`)
- [ ] HTTP → HTTPS redirect enforced
- [ ] No sensitive data in URL parameters (tokens, passwords)
- [ ] Encryption keys stored securely (not hardcoded or in source code)
- [ ] Database encryption at rest enabled for sensitive tables
- [ ] API keys, secrets not exposed in HTML source / JS bundles
- [ ] Certificate pinning implemented (for high-risk mobile/API clients)

**References:** [CWE-311](https://cwe.mitre.org/data/definitions/311.html) | [OWASP A02:2021](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)

---

## A03 — Injection

> CWE-79, CWE-89, CWE-917

- [ ] All user input validated and sanitized server-side
- [ ] Parameterized queries / prepared statements used for all DB operations
- [ ] ORM used correctly — no raw query concatenation with user input
- [ ] SQL injection tested on all input fields, headers, and cookies
- [ ] NoSQL injection tested (MongoDB `$where`, `$gt` operators)
- [ ] LDAP injection tested on authentication modules
- [ ] OS command injection tested (`ping`, `curl`, `system()` calls)
- [ ] XML/XPath injection tested on XML parsers
- [ ] Template injection (SSTI) tested on server-side rendering
- [ ] Output encoding applied before rendering user-controlled data in HTML/JS/CSS

**References:** [CWE-89](https://cwe.mitre.org/data/definitions/89.html) | [OWASP A03:2021](https://owasp.org/Top10/A03_2021-Injection/)

---

## A04 — Insecure Design

> CWE-209, CWE-656

- [ ] Threat modeling performed during design phase (STRIDE/PASTA)
- [ ] Business logic flaws tested (price manipulation, workflow bypass)
- [ ] Rate limiting applied to sensitive operations (login, OTP, checkout)
- [ ] Anti-automation controls in place for high-value endpoints
- [ ] Principle of least privilege applied in architecture
- [ ] Sensitive operations require re-authentication
- [ ] Multi-step processes validated at each step server-side
- [ ] Data validation rules documented and enforced

**References:** [OWASP A04:2021](https://owasp.org/Top10/A04_2021-Insecure_Design/)

---

## A05 — Security Misconfiguration

> CWE-16, CWE-611

- [ ] Security hardening applied across all stack layers (OS, web server, framework, DB)
- [ ] Default credentials changed (admin/admin, root/root, etc.)
- [ ] Unnecessary features/services/ports disabled
- [ ] Error messages do not expose stack traces or internal paths to users
- [ ] Security headers configured: `X-Content-Type-Options`, `X-Frame-Options`, `CSP`, `Referrer-Policy`
- [ ] Cloud storage buckets not publicly accessible
- [ ] XML external entity processing (XXE) disabled
- [ ] Debug mode disabled in production
- [ ] Server version not disclosed in response headers (`Server`, `X-Powered-By`)
- [ ] Directory traversal tested

**Security Headers Quick Reference:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: no-referrer
Permissions-Policy: geolocation=(), microphone=()
```

**References:** [OWASP A05:2021](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)

---

## A06 — Vulnerable and Outdated Components

> CWE-1035, CWE-937

- [ ] Inventory of all third-party libraries and components maintained
- [ ] Software Composition Analysis (SCA) tool integrated in CI/CD (Snyk, Dependabot, OWASP Dependency-Check)
- [ ] No known CVEs in production dependencies
- [ ] OS and runtime packages patched regularly
- [ ] End-of-life (EOL) frameworks/runtimes identified and scheduled for upgrade
- [ ] Container base images scanned for vulnerabilities
- [ ] License compliance checked for open-source components

**References:** [OWASP A06:2021](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/)

---

## A07 — Identification and Authentication Failures

> CWE-287, CWE-297, CWE-384

- [ ] Multi-factor authentication (MFA) enforced for privileged accounts
- [ ] Brute-force protection on login (lockout / rate limiting / CAPTCHA)
- [ ] Password complexity and minimum length enforced
- [ ] "Forgot password" flow does not leak account existence
- [ ] Session tokens have sufficient entropy (128-bit minimum)
- [ ] Session invalidated on logout (server-side)
- [ ] Session timeout enforced (idle and absolute)
- [ ] Session fixation attacks mitigated (new token on login)
- [ ] JWTs validated for signature, expiry (`exp`), and audience (`aud`)
- [ ] "Remember me" tokens stored securely (HttpOnly, Secure, SameSite cookies)

**References:** [OWASP A07:2021](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)

---

## A08 — Software and Data Integrity Failures

> CWE-494, CWE-829

- [ ] CI/CD pipeline secured — no untrusted plugins or scripts
- [ ] Code signing / artifact verification in deployment pipeline
- [ ] Subresource Integrity (SRI) used for CDN-loaded scripts/styles
- [ ] Deserialization of untrusted data avoided; or strictly validated
- [ ] Software updates verified via signed checksums
- [ ] Third-party integrations reviewed for supply chain risk
- [ ] SBOM (Software Bill of Materials) generated for releases

**References:** [OWASP A08:2021](https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/)

---

## A09 — Security Logging and Monitoring Failures

> CWE-117, CWE-223, CWE-778

- [ ] Authentication events logged (success, failure, lockout)
- [ ] Authorization failures logged
- [ ] Input validation failures logged
- [ ] High-value transactions logged with immutable audit trail
- [ ] Logs do not contain sensitive data (passwords, tokens, PII)
- [ ] Centralized logging (SIEM) in place
- [ ] Alerts configured for suspicious activity patterns
- [ ] Log tampering protected (append-only, offsite storage)
- [ ] Mean Time to Detect (MTTD) baseline established

**References:** [OWASP A09:2021](https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/)

---

## A10 — Server-Side Request Forgery (SSRF)

> CWE-918

- [ ] URL/IP inputs validated against allowlist (not blocklist)
- [ ] Internal metadata endpoints blocked (`169.254.169.254`, `fd00::/8`)
- [ ] DNS rebinding attacks mitigated
- [ ] Outbound requests from server limited to necessary domains/IPs
- [ ] Response from server-initiated requests not reflected directly to user
- [ ] Cloud provider IMDS (Instance Metadata Service) access restricted
- [ ] SSRF tested on file upload, webhook, URL preview, PDF generation features

**References:** [CWE-918](https://cwe.mitre.org/data/definitions/918.html) | [OWASP A10:2021](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)

---

## 📋 Summary Scorecard

| Category | Status | Severity if Failed | Notes |
|---|---|---|---|
| A01 - Broken Access Control | | Critical | |
| A02 - Cryptographic Failures | | High | |
| A03 - Injection | | Critical | |
| A04 - Insecure Design | | High | |
| A05 - Security Misconfiguration | | Medium–High | |
| A06 - Vulnerable Components | | Medium–High | |
| A07 - Auth Failures | | Critical | |
| A08 - Integrity Failures | | High | |
| A09 - Logging Failures | | Medium | |
| A10 - SSRF | | High | |

---

*Last Updated: 2024 | Standard: OWASP Top 10 (2021) | Maintained by AppSec Team*
