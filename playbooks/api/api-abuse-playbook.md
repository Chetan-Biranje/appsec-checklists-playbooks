# 🚨 Incident Response Playbook — API Abuse & Automated Attack

> **Severity:** High
> **CWE:** CWE-307, CWE-770, CWE-799
> **OWASP API:** API4:2023 – Unrestricted Resource Consumption, API6:2023 – Business Flow Abuse
> **Playbook Version:** 1.0

---

## Overview

API abuse encompasses automated attacks against APIs including credential stuffing, scraping, inventory hoarding, promo abuse, account takeover via brute-force, and resource exhaustion attacks. These attacks often appear as legitimate traffic at the network layer.

---

## Detection Signals

### Traffic Anomalies
- Request rate far exceeding normal per-user baselines
- High volume of 401/403/429 responses from single IP/ASN
- Unusual geographic distribution of requests
- High ratio of failed-to-successful login attempts (>10:1)
- Sequential enumeration of IDs or usernames

### Business Logic Anomalies
- Mass account creation from same IP/device fingerprint
- Same promo code redeemed by hundreds of accounts
- Rapid inventory depletion followed by marketplace resale
- OTP/verification endpoint receiving unusual traffic
- API response time spikes (resource exhaustion)

---

## Phase 1 — Classification

**Determine attack type before response:**

| Attack Type | Indicators |
|---|---|
| Credential Stuffing | High login failures, valid username:password pairs from breach lists |
| Brute Force | Sequential password attempts, single username |
| Scraping | High GET volume, sequential ID traversal, full catalog requests |
| Account Takeover | Successful logins from new geo, immediate profile/password changes |
| Inventory Hoarding | Mass add-to-cart, low checkout rate |
| Promo Abuse | Same promo code, multiple accounts, same payment method |
| Resource Exhaustion | API response degradation, expensive endpoint hammering |

---

## Phase 2 — Containment

**⏱ Target Time: 0–30 minutes**

### Immediate Rate Limiting
- [ ] Tighten rate limits at API gateway for affected endpoint
- [ ] Apply per-IP rate limiting (aggressive: 5 req/min for auth endpoints)
- [ ] Block top attacking IPs/ASNs at WAF/firewall
- [ ] Enable CAPTCHA challenge for suspicious clients (JS challenge at CDN)
- [ ] Geo-block if attack concentrated in unexpected region (if business allows)

### Credential Stuffing Specific
- [ ] Enable CAPTCHA on login immediately
- [ ] Implement progressive delays on failed logins
- [ ] Send "unusual login" alerts to affected users
- [ ] If breach data identified as source: force password reset for affected accounts

### Account Takeover Confirmed
- [ ] Suspend compromised accounts pending user verification
- [ ] Invalidate all sessions for affected accounts
- [ ] Notify affected users via secondary channel (email)
- [ ] Initiate account recovery flow

---

## Phase 3 — Investigation

- [ ] Pull full traffic logs for attacking IPs — timeline, endpoints, user agents
- [ ] Identify total accounts successfully compromised
- [ ] Check if attacker used residential proxies / rotating IPs
- [ ] Identify attacker tooling from User-Agent patterns
- [ ] Correlate with known breach data (HaveIBeenPwned API)
- [ ] Assess business impact: financial loss, data exposure, reputation
- [ ] Identify persistence (did attacker plant backdoors in compromised accounts)
- [ ] Review access logs for data exfiltration from compromised sessions

---

## Phase 4 — Eradication & Hardening

### Rate Limiting Architecture
```
API Gateway Layer:
  - Per-IP: 100 req/min (general), 5 req/min (auth)
  - Per-user: 1000 req/min (general), 10 req/min (sensitive)
  - Per endpoint: custom limits on high-cost operations

Response: 429 Too Many Requests
Headers: 
  Retry-After: 60
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 0
  X-RateLimit-Reset: 1735689600
```

### Authentication Hardening
```
Failed Login Lockout Policy:
  - 5 failed attempts → 1 minute lockout
  - 10 failed attempts → 15 minute lockout  
  - 20 failed attempts → Account lock + email notification
  - Use exponential backoff with jitter
```

### Bot Detection Layers
- [ ] **Layer 1:** IP reputation (Cloudflare, AWS WAF managed rules)
- [ ] **Layer 2:** Behavioral analysis (request timing, mouse movement, JS fingerprint)
- [ ] **Layer 3:** Device fingerprinting (TLS fingerprint, HTTP/2 fingerprint)
- [ ] **Layer 4:** CAPTCHA for challenge-response on suspicious clients
- [ ] **Layer 5:** Business logic (limit accounts per IP, device, payment method)

### Credential Stuffing Prevention
```python
# Check credentials against known breach databases
import hashlib
import requests

def check_pwned_password(password: str) -> int:
    """Returns number of times password appeared in breaches (0 = safe)"""
    sha1 = hashlib.sha1(password.encode()).hexdigest().upper()
    prefix, suffix = sha1[:5], sha1[5:]
    
    resp = requests.get(f"https://api.pwnedpasswords.com/range/{prefix}")
    hashes = {line.split(':')[0]: int(line.split(':')[1]) 
              for line in resp.text.splitlines()}
    return hashes.get(suffix, 0)
```

---

## Phase 5 — Recovery

- [ ] Notify affected users with clear action steps
- [ ] Provide account recovery mechanism
- [ ] Monitor for repeat attacks post-mitigation
- [ ] Review and refine rate limit thresholds based on attack data
- [ ] Update bot signature blocklists

---

## Phase 6 — Post-Incident Actions

- [ ] Document: attack timeline, volume, impact, TTR
- [ ] Review rate limiting strategy across all API endpoints
- [ ] Implement device fingerprinting if not present
- [ ] Evaluate third-party bot management solution (Cloudflare Bot Management, DataDome, PerimeterX)
- [ ] Run tabletop exercise for next API abuse scenario

---

*Playbook Version: 1.0 | Reference: OWASP Credential Stuffing Prevention Cheat Sheet*
