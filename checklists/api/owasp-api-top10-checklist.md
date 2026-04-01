# 🔌 OWASP API Security Top 10 Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Standard:** OWASP API Security Top 10 (2023)
> **Use Case:** API Security Assessment, Design Review, Pre-release Gate

---

## API1:2023 — Broken Object Level Authorization (BOLA/IDOR)

- [ ] Every API endpoint validates the requesting user owns/can access the requested object
- [ ] Object IDs in paths, query params, and request body all validated
- [ ] GUIDs used instead of sequential integer IDs where possible
- [ ] Authorization checks happen at data access layer, not just controller
- [ ] BOLA tested across all resource types (users, orders, docs, messages)
- [ ] Indirect object references implemented (user-scoped tokens instead of DB IDs)

**Test Pattern:**
```
GET /api/v1/invoices/1001   → User A's invoice
GET /api/v1/invoices/1002   → Can User A access User B's invoice? ← BOLA
```

---

## API2:2023 — Broken Authentication

- [ ] Tokens have short expiry (access: 15min–1hr, refresh: days)
- [ ] Refresh token rotation implemented (old token invalidated on use)
- [ ] Token revocation endpoint exists and works correctly
- [ ] API keys cannot be reused after rotation
- [ ] Brute-force protection on authentication endpoints
- [ ] Credential stuffing protection (device fingerprinting, anomaly detection)
- [ ] JWTs validated: signature, `exp`, `iss`, `aud`, `nbf`
- [ ] `Authorization: Bearer` not accepted in URL parameters (logging risk)

---

## API3:2023 — Broken Object Property Level Authorization

- [ ] API responses only return fields the requester is authorized to see
- [ ] Mass assignment protection: only explicitly allowed fields accepted
- [ ] Sensitive fields (`password`, `role`, `isAdmin`) not accepted in update requests
- [ ] Response filtering enforced server-side (not just via frontend hiding)
- [ ] Field-level access control reviewed for all user roles

**Test Pattern:**
```json
PATCH /api/v1/users/me
{"name": "John", "role": "admin"}  ← Should role be updateable?
```

---

## API4:2023 — Unrestricted Resource Consumption

- [ ] Rate limiting applied per user, per IP, and globally
- [ ] Pagination enforced (no `?limit=99999999`)
- [ ] Max request body size enforced
- [ ] File upload size and count limits enforced
- [ ] Expensive operations (exports, reports) are async and throttled
- [ ] GraphQL query depth and complexity limits set
- [ ] Timeouts enforced on all external calls
- [ ] Cost-per-request tracked for AI/ML endpoints

---

## API5:2023 — Broken Function Level Authorization

- [ ] All administrative/privileged endpoints require elevated role
- [ ] Undocumented or legacy endpoints identified and secured
- [ ] HTTP methods restricted per endpoint (GET-only endpoints reject POST/DELETE)
- [ ] Internal-only APIs not exposed publicly
- [ ] API versioning doesn't leave old insecure versions accessible
- [ ] Swagger/OpenAPI spec matches actual implemented endpoints

**Test Pattern:**
```
DELETE /api/v1/admin/users/123   → Regular user should get 403, not 200
GET /api/v2/users vs GET /api/v1/users → Both versions must enforce auth
```

---

## API6:2023 — Unrestricted Access to Sensitive Business Flows

- [ ] High-risk flows (checkout, account creation, OTP) rate limited
- [ ] Bot/automation detection on business-critical APIs
- [ ] Multi-step workflows validated server-side at each step
- [ ] Inventory/quantity checked atomically (prevent overselling)
- [ ] Referral/promo code abuse protections in place
- [ ] SMS/email OTP endpoints rate limited aggressively

---

## API7:2023 — Server-Side Request Forgery (SSRF)

- [ ] URL parameters validated against strict allowlist
- [ ] DNS resolution of user-supplied hostnames blocked for internal IPs
- [ ] Cloud metadata endpoints blocked (`169.254.169.254`)
- [ ] Webhook/callback URLs validated
- [ ] Response from server-initiated requests sanitized before returning to client
- [ ] Outbound network access from API servers restricted via firewall

---

## API8:2023 — Security Misconfiguration

- [ ] CORS configured with explicit allowed origins (not `*` for credentialed requests)
- [ ] Debug endpoints (`/debug`, `/actuator`, `/metrics`) not exposed publicly
- [ ] HTTP methods restricted appropriately per endpoint
- [ ] Security headers present (`HSTS`, `X-Content-Type-Options`, `CSP`)
- [ ] Stack traces not returned in API error responses
- [ ] API gateway configured with WAF rules
- [ ] Default framework endpoints disabled

---

## API9:2023 — Improper Inventory Management

- [ ] All API versions documented in central registry
- [ ] Deprecated API versions have sunset dates and are decommissioned
- [ ] Third-party/external-facing APIs documented separately from internal
- [ ] Shadow APIs (undocumented, auto-generated) discovered and reviewed
- [ ] API inventory reviewed quarterly
- [ ] OpenAPI/Swagger spec kept up to date and stored in version control

---

## API10:2023 — Unsafe Consumption of APIs

- [ ] Third-party API responses validated before use
- [ ] Data from external APIs not directly reflected to users without sanitization
- [ ] TLS certificates of third-party APIs verified
- [ ] Timeouts and circuit breakers implemented for external API calls
- [ ] Error handling for third-party API failures is secure (no data leakage)
- [ ] Webhook payloads validated with HMAC signatures

---

## API Security Quick Reference

| Control | Recommended Value |
|---|---|
| Access Token Expiry | 15 min – 1 hr |
| Refresh Token Expiry | 7–30 days |
| Rate Limit (auth endpoints) | 5 req/min per IP |
| Rate Limit (general) | 100–1000 req/min per user |
| Max Request Body | 1–10 MB (context-dependent) |
| GraphQL Query Depth | Max 5–10 levels |
| CORS Allowed Origins | Explicit allowlist only |

---

## API Security Testing Tools

| Tool | Purpose |
|---|---|
| Burp Suite Pro | Manual API testing, fuzzing |
| Postman + Newman | Automated API security regression |
| OWASP ZAP | Automated scanning |
| 42Crunch API Security Audit | OpenAPI spec analysis |
| Arjun | Hidden parameter discovery |
| jwt_tool | JWT attack testing |
| kiterunner | API endpoint brute-forcing |

---

*Last Updated: 2024 | Standard: OWASP API Security Top 10 (2023)*
