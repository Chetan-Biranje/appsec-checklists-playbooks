# 🔍 Secure Code Review Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Use Case:** Manual Code Review, Pull Request Security Gate, Security Audit

---

## General Code Quality & Security Hygiene

- [ ] No hardcoded secrets, API keys, or passwords in source code
- [ ] No commented-out debug code, TODOs exposing sensitive logic
- [ ] Sensitive data not logged (passwords, tokens, PII, card numbers)
- [ ] No use of deprecated or insecure functions (`eval()`, `exec()`, `strcpy()`)
- [ ] Dead code and unused imports removed
- [ ] Third-party libraries are pinned to specific versions

---

## Input Validation

- [ ] All user-supplied input validated on the server side
- [ ] Input length limits enforced (prevent buffer overflows, DoS)
- [ ] Allowlist validation used where possible (not just blocklist)
- [ ] File uploads: type, size, and content validated (not just extension)
- [ ] Numeric inputs checked for integer overflow/underflow
- [ ] JSON/XML input validated against strict schemas
- [ ] Regex patterns reviewed for ReDoS vulnerability

---

## Authentication & Session Management

- [ ] Passwords hashed with strong algorithm (bcrypt/Argon2/scrypt)
- [ ] Authentication logic centralized, not scattered across codebase
- [ ] Session tokens generated using cryptographically secure RNG
- [ ] Session invalidation logic is correct and complete on logout
- [ ] Password reset tokens expire within short window (15–60 min)
- [ ] MFA implementation reviewed for bypass vulnerabilities
- [ ] Account enumeration not possible via timing differences or error messages

---

## Authorization

- [ ] Authorization checks present on every sensitive function/endpoint
- [ ] No reliance on client-supplied values for access control decisions
- [ ] Object-level authorization: IDs validated against current user's context
- [ ] Role checks cannot be bypassed via HTTP method tampering
- [ ] Function-level access control (admin-only routes protected)

---

## Cryptography

- [ ] No use of MD5/SHA1 for security-sensitive operations
- [ ] Symmetric encryption uses AES-256 (GCM mode preferred)
- [ ] IV/Nonce is randomly generated per operation (never reused)
- [ ] Private keys not stored in code or config files
- [ ] Random number generation uses `SecureRandom` / `os.urandom()` equivalent
- [ ] JWT signing uses RS256 or ES256 (avoid HS256 with weak keys)
- [ ] `alg: none` JWT attack mitigated

---

## Error Handling & Logging

- [ ] Generic error messages returned to user; details logged server-side only
- [ ] Exception handlers do not expose internal paths or stack traces
- [ ] Errors caught and handled — no unhandled promise rejections / null pointers
- [ ] Security events logged with sufficient context (user, IP, timestamp, action)
- [ ] Logs use structured format (JSON) for SIEM ingestion

---

## Database / Data Access

- [ ] ORM used correctly — no `.raw()` or string concatenation for queries
- [ ] Stored procedures use parameterized inputs
- [ ] DB connection strings not hardcoded — fetched from secrets manager
- [ ] Least privilege DB accounts used (read-only where applicable)
- [ ] Mass assignment / over-posting vulnerabilities reviewed

---

## Dependency & Third-Party Code

- [ ] All imported libraries reviewed for known CVEs
- [ ] No untrusted or unverified NPM/PyPI packages used
- [ ] `package-lock.json` / `poetry.lock` committed and verified
- [ ] Dynamic code loading reviewed (e.g., `require()` with user input)

---

## Business Logic

- [ ] Workflow steps validated server-side (cannot skip steps)
- [ ] Monetary/quantity values validated (negative amounts, overflow)
- [ ] Race conditions reviewed in concurrent operations (TOCTOU)
- [ ] Critical operations require re-authentication or confirmation
- [ ] Rate limits enforced programmatically, not just at gateway

---

## Frontend / Client-Side (if applicable)

- [ ] No sensitive data stored in `localStorage` or `sessionStorage`
- [ ] `innerHTML` / `document.write()` not used with user input (XSS risk)
- [ ] `postMessage` origin validated
- [ ] CSRF tokens present on all state-changing requests
- [ ] SameSite=Strict/Lax on all sensitive cookies
- [ ] Content Security Policy (CSP) reviewed and tightened

---

## Code Review Severity Rating

| Finding | Severity |
|---|---|
| Hardcoded credentials | Critical |
| SQL Injection | Critical |
| Auth bypass | Critical |
| Missing authorization check | High |
| Weak cryptography | High |
| Missing CSRF protection | Medium |
| Verbose error messages | Low |
| Missing rate limiting | Medium |

---

*Last Updated: 2024 | Use with: SAST tools (Semgrep, CodeQL, Checkmarx)*
