# 🚨 Incident Response Playbook — SQL Injection

> **Severity:** Critical
> **CWE:** CWE-89
> **OWASP:** A03:2021 – Injection
> **Playbook Version:** 1.0

---

## Overview

SQL Injection occurs when untrusted user input is incorporated into database queries without proper sanitization, allowing attackers to manipulate query logic to extract, modify, or delete data.

**Potential Impact:**
- Full database dump (all user data, PII, credentials)
- Authentication bypass
- Data modification or destruction
- Remote code execution (via xp_cmdshell, UDF, etc.)
- Lateral movement via DB server

---

## Detection Signals

### Application / WAF Logs
```
GET /search?q=' OR '1'='1
POST /login: username=admin'--&password=anything
Error: You have an error in your SQL syntax...
Error: ORA-00907: missing right parenthesis
```

### SIEM Alerts
- Spike in 500 errors from single IP
- Database error strings in response bodies
- Unusually large data transfers from DB server
- `UNION SELECT`, `ORDER BY`, `SLEEP()`, `WAITFOR DELAY` in request logs
- SQLi signatures in WAF rules triggered

---

## Phase 1 — Identification & Triage

**⏱ Target Time: 0–30 minutes**

- [ ] Confirm alert is not a false positive — review raw HTTP request
- [ ] Identify affected endpoint(s) and parameter(s)
- [ ] Determine injection type: Error-based / Blind / Time-based / UNION-based
- [ ] Check if payload was blocked by WAF or reached backend
- [ ] Review DB server logs for anomalous queries
- [ ] Assess if data exfiltration occurred (volume of outbound data from DB host)
- [ ] Identify all potentially affected tables/databases
- [ ] Check for privilege escalation (OS commands, file read/write)
- [ ] Assign severity: **Critical** if PII/credentials accessed

**Initial Questions:**
1. Was the injection successful (got a DB error or unexpected response)?
2. What data is in the affected database? (PII, credentials, financial data)
3. How long has this endpoint been exposed?
4. Is this a zero-day or known vulnerability?

---

## Phase 2 — Containment

**⏱ Target Time: 30–60 minutes**

### Immediate Actions
- [ ] **Block attacker IP(s)** at WAF / load balancer / firewall
- [ ] **Enable WAF** in blocking mode for SQLi signatures if in monitor mode
- [ ] **Rate limit or block** the affected endpoint temporarily if business allows
- [ ] **Rotate** database credentials (application DB user password)
- [ ] **Revoke sessions** for all active users if authentication bypass occurred
- [ ] Capture and preserve logs before rotation

### If Data Breach Confirmed
- [ ] Notify Security Lead / CISO immediately
- [ ] Begin incident record in ticketing system
- [ ] Preserve evidence: HTTP logs, DB query logs, network captures
- [ ] Do NOT patch before preserving forensic evidence

### Temporary Mitigation (if fix not ready)
```nginx
# Nginx WAF rule (ModSecurity / NAXSI)
location /search {
    ModSecurityEnabled on;
    SecRuleEngine On;
}
```

---

## Phase 3 — Investigation & Forensics

**⏱ Target Time: 1–4 hours**

- [ ] Pull all requests to affected endpoint from past 30–90 days
- [ ] Identify first occurrence of malicious payload (patient zero)
- [ ] Review attacker IP(s) for pattern (VPN, Tor, residential proxy)
- [ ] Extract all unique payloads used
- [ ] Determine extent of data accessed:
  - [ ] Which tables were queried?
  - [ ] Were all records accessed or specific users?
  - [ ] Were writes/deletes performed?
- [ ] Check for backdoors: web shells uploaded, new DB users created, scheduled jobs
- [ ] Review DB audit logs: `SELECT`, `INSERT`, `UPDATE`, `DROP` statements
- [ ] Check for data exfiltration via outbound DNS, HTTP, email

**DB Forensics Queries (adapt per DB):**
```sql
-- MySQL: Review recent query history (if general_log enabled)
SELECT * FROM mysql.general_log 
WHERE argument LIKE '%UNION%' OR argument LIKE '%SLEEP%'
ORDER BY event_time DESC LIMIT 100;

-- Check for new DB users created
SELECT * FROM mysql.user ORDER BY Create_time DESC;

-- PostgreSQL: pg_stat_activity
SELECT * FROM pg_stat_activity WHERE query LIKE '%UNION%';
```

---

## Phase 4 — Eradication & Remediation

**⏱ Target Time: 4–24 hours for critical fixes**

### Code Fix (Immediate)
- [ ] Replace dynamic query construction with parameterized queries
- [ ] Audit ALL database calls in the affected service
- [ ] Extend audit to similar patterns across entire codebase

**Before (Vulnerable):**
```python
# VULNERABLE
query = f"SELECT * FROM users WHERE username = '{username}'"
cursor.execute(query)
```

**After (Fixed):**
```python
# SECURE — Parameterized Query
query = "SELECT * FROM users WHERE username = %s"
cursor.execute(query, (username,))
```

**ORM Example (SQLAlchemy):**
```python
# SECURE — ORM
user = session.query(User).filter(User.username == username).first()
```

- [ ] Apply WAF virtual patch while code fix is in testing
- [ ] Deploy SAST rule (Semgrep) to catch similar patterns in future PRs
- [ ] Rotate all database credentials used by the application
- [ ] Review DB user permissions — revoke unnecessary privileges (DROP, FILE, EXEC)
- [ ] Enable parameterized query enforcement at ORM / framework level
- [ ] Reset sessions and force re-authentication for all users if breach confirmed

---

## Phase 5 — Recovery

- [ ] Deploy patched code to staging — verify fix with SQLi test cases
- [ ] Perform security regression test (automated SQLi scan via sqlmap in test env)
- [ ] Deploy to production with monitoring heightened
- [ ] Monitor for repeat SQLi attempts post-fix
- [ ] Verify WAF rules active and logging correctly
- [ ] Confirm DB logs clean post-fix

---

## Phase 6 — Post-Incident

### Notifications (if data breach)
- [ ] Legal / Compliance team notified
- [ ] GDPR/CCPA notification timeline reviewed (72 hrs for GDPR)
- [ ] Affected users notified if PII exposed
- [ ] Regulatory body notified if required

### Documentation
- [ ] Incident timeline documented
- [ ] Root cause analysis (RCA) completed
- [ ] Lessons learned documented
- [ ] Remediation actions tracked to completion

### Hardening Actions
- [ ] SAST integrated in CI/CD pipeline with SQLi rules
- [ ] Developer security training on parameterized queries
- [ ] WAF rules reviewed and strengthened
- [ ] DB user permissions reviewed and tightened
- [ ] DB audit logging enabled permanently

---

## Severity Escalation Matrix

| Condition | Action |
|---|---|
| Injection blocked by WAF | Monitor, patch within sprint |
| Injection reached backend but no data accessed | Patch within 24 hrs, notify security lead |
| Data accessed but no exfiltration | Patch immediately, notify CISO |
| Data exfiltrated | Patch immediately, CISO + Legal + DPO notified |
| Auth bypass achieved | Patch + force logout all users + immediate CISO escalation |

---

*Playbook Version: 1.0 | Reference: OWASP SQL Injection Prevention Cheat Sheet*
