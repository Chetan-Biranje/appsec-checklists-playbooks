# 🎯 Web Application Penetration Testing Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Use Case:** Black-box / Grey-box / White-box Pentest Engagements
> **Framework Reference:** OWASP Testing Guide v4.2, PTES, OSSTMM

---

## Phase 1 — Reconnaissance & Information Gathering

- [ ] Subdomain enumeration (amass, subfinder, dnsx)
- [ ] DNS records mapped (A, MX, TXT, CNAME, NS)
- [ ] Technology stack fingerprinted (Wappalyzer, WhatWeb, headers)
- [ ] Web server version, framework, CMS identified
- [ ] Wayback Machine / CommonCrawl checked for historical endpoints
- [ ] Google dorking for exposed files, login pages, config leaks
- [ ] GitHub / GitLab searched for leaked secrets, source code
- [ ] Shodan/Censys checked for open ports and exposed services
- [ ] JavaScript files analyzed for hidden endpoints and API keys
- [ ] Email addresses harvested for social engineering assessment (if in scope)

---

## Phase 2 — Mapping & Crawling

- [ ] Spider/crawl performed (Burp Suite, ZAP)
- [ ] All input vectors identified (forms, headers, cookies, URL params)
- [ ] API endpoints discovered (JS analysis, Swagger/OpenAPI, brute-force)
- [ ] Hidden directories and files discovered (gobuster, feroxbuster, ffuf)
- [ ] Backup files tested (`index.php.bak`, `.DS_Store`, `web.config.bak`)
- [ ] HTTP methods tested on all endpoints (PUT, DELETE, TRACE, OPTIONS)
- [ ] Authenticated and unauthenticated surface mapped separately

---

## Phase 3 — Authentication Testing

- [ ] Default credentials tested
- [ ] Username enumeration tested (login, register, forgot password)
- [ ] Brute-force protection tested (lockout, CAPTCHA bypass, rate limit)
- [ ] MFA bypass tested (code reuse, response manipulation, backup codes)
- [ ] Password reset flow tested (token predictability, reuse, no expiry)
- [ ] OAuth flow tested (redirect_uri manipulation, state parameter, token leakage)
- [ ] JWT attacks tested (alg:none, weak secret, kid injection, jku/x5u manipulation)
- [ ] Session fixation tested
- [ ] "Remember me" token security assessed

---

## Phase 4 — Authorization Testing

- [ ] IDOR tested on all object references (numeric IDs, GUIDs, hashes)
- [ ] Vertical privilege escalation tested (regular → admin endpoints)
- [ ] Horizontal privilege escalation tested (user A → user B resources)
- [ ] Forced browsing tested (direct URL access to protected pages)
- [ ] Parameter tampering tested (role, userId, accountId in requests)
- [ ] HTTP method override tested (`X-HTTP-Method-Override`)
- [ ] API versioning (v1 vs v2) access control parity checked

---

## Phase 5 — Injection Testing

- [ ] SQL Injection — manual + sqlmap on all parameters
- [ ] Blind SQLi tested (time-based, boolean-based)
- [ ] Second-order SQLi tested
- [ ] NoSQL Injection tested (MongoDB, Elasticsearch)
- [ ] OS Command Injection tested
- [ ] LDAP Injection tested
- [ ] XML/XPath Injection tested
- [ ] SSTI (Server-Side Template Injection) tested (Jinja2, Twig, Freemarker)
- [ ] HTML Injection tested

---

## Phase 6 — XSS & Client-Side Testing

- [ ] Reflected XSS tested on all parameters
- [ ] Stored XSS tested on all persistent user-input fields
- [ ] DOM-based XSS tested (JS sources: `location.hash`, `document.referrer`)
- [ ] CSP analyzed and bypass attempted
- [ ] Clickjacking tested (`X-Frame-Options`, frame embedding)
- [ ] CSRF tested on all state-changing operations
- [ ] `postMessage` origin validation tested
- [ ] Open redirect tested (redirect_to, next, url params)
- [ ] Prototype pollution tested (JavaScript)

---

## Phase 7 — Business Logic Testing

- [ ] Workflow bypass tested (skip steps, repeat steps)
- [ ] Negative quantity / price manipulation tested
- [ ] Race conditions tested (concurrent requests on limited resources)
- [ ] Feature flag bypass tested
- [ ] Account linking / merging abuse tested
- [ ] File upload abuse (malicious content, path traversal in filename)
- [ ] Coupon/voucher reuse or stacking abuse tested

---

## Phase 8 — Infrastructure & Configuration

- [ ] SSRF tested on all URL/IP input fields
- [ ] XXE tested on all XML-accepting endpoints
- [ ] Path/directory traversal tested
- [ ] Security headers audited (CSP, HSTS, CORS, etc.)
- [ ] TLS configuration tested (SSL Labs / testssl.sh)
- [ ] HTTP to HTTPS redirect verified
- [ ] Error pages reviewed for information disclosure
- [ ] Admin interfaces identified and tested

---

## Phase 9 — Reporting Milestones

- [ ] All findings documented with: Title, Severity, CVSS Score, Steps to Reproduce, Impact, Recommendation
- [ ] PoC screenshots / HTTP request-response captured
- [ ] CWE and OWASP category mapped for each finding
- [ ] False positives eliminated through manual verification
- [ ] Executive summary written for non-technical stakeholders
- [ ] Remediation guidance tailored to tech stack
- [ ] Re-test plan proposed for critical/high findings

---

## CVSS Severity Reference

| Score | Severity |
|---|---|
| 9.0 – 10.0 | Critical |
| 7.0 – 8.9 | High |
| 4.0 – 6.9 | Medium |
| 0.1 – 3.9 | Low |
| 0.0 | Informational |

---

## Recommended Tools

| Category | Tools |
|---|---|
| Proxy | Burp Suite Pro, OWASP ZAP |
| Recon | Amass, Subfinder, theHarvester |
| Fuzzing | ffuf, feroxbuster, wfuzz |
| SQLi | sqlmap |
| Exploitation | Metasploit (where authorized) |
| TLS | testssl.sh, SSL Labs |
| Secrets | truffleHog, gitleaks |

---

*Last Updated: 2024 | Reference: OWASP Testing Guide v4.2*
