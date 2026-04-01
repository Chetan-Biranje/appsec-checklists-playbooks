# 🔐 REST API Security Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Use Case:** API Design Review, Pre-deployment Security Gate, Code Review

---

## Transport Security

- [ ] All API traffic served over HTTPS/TLS 1.2+
- [ ] HTTP requests rejected or redirected to HTTPS
- [ ] HSTS header set with sufficient `max-age`
- [ ] Certificate chain valid and not expired
- [ ] Weak cipher suites disabled
- [ ] Certificate pinning implemented for high-risk mobile clients

---

## Authentication

- [ ] Authentication required on all non-public endpoints
- [ ] Bearer token passed in `Authorization` header only (not URL)
- [ ] API key rotation supported and documented
- [ ] Service-to-service auth uses mTLS or signed JWTs (not shared secrets)
- [ ] OAuth 2.0 scopes are minimal and granular
- [ ] Token introspection endpoint secured
- [ ] Machine-to-machine tokens have short TTL and auto-rotate

---

## Authorization

- [ ] Every endpoint has explicit authorization policy defined
- [ ] Claims/scope validated on every request (not just at login)
- [ ] Resource-level access validated (not just endpoint-level)
- [ ] Privilege escalation via parameter tampering not possible
- [ ] Service accounts have least-privilege access
- [ ] Authorization logic centralized and not duplicated across handlers

---

## Input Validation

- [ ] All request inputs validated: path params, query params, headers, body
- [ ] JSON schema validation enforced (strict mode)
- [ ] Content-Type header validated before parsing body
- [ ] Unexpected fields rejected (or ignored) — no mass assignment
- [ ] Integer fields checked for min/max range
- [ ] String fields have max length limits
- [ ] Enum fields validated against defined values

---

## Output / Response Security

- [ ] Only necessary fields returned in response (minimize data exposure)
- [ ] Sensitive fields excluded from responses (password hashes, internal IDs)
- [ ] Error responses use generic messages (no stack traces, SQL errors)
- [ ] Content-Type response header set correctly (`application/json`)
- [ ] `X-Content-Type-Options: nosniff` header present
- [ ] Pagination implemented for list endpoints
- [ ] Large responses compressed (gzip) to prevent amplification

---

## Rate Limiting & Abuse Prevention

- [ ] Rate limiting applied at API gateway level
- [ ] Per-user and per-IP rate limits defined
- [ ] Rate limit headers returned (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`)
- [ ] Costly operations (search, export) have separate stricter limits
- [ ] 429 response returned gracefully with `Retry-After` header
- [ ] Distributed rate limiting (Redis-backed) for multi-instance APIs

---

## CORS Configuration

- [ ] `Access-Control-Allow-Origin` set to explicit allowlist (not `*`)
- [ ] Credentialed requests (`withCredentials: true`) never paired with `*` origin
- [ ] `Access-Control-Allow-Methods` lists only necessary methods
- [ ] `Access-Control-Allow-Headers` limited to required headers
- [ ] CORS preflight caching configured (`Access-Control-Max-Age`)

---

## HTTP Methods & Status Codes

- [ ] Only necessary HTTP methods enabled per endpoint
- [ ] `OPTIONS` method handled for CORS preflight
- [ ] `TRACE` and `CONNECT` methods disabled
- [ ] Correct status codes used (401 vs 403, 404 vs 403 for security)
- [ ] `405 Method Not Allowed` returned for unsupported methods

**Status Code Security Guide:**

| Scenario | Correct Code | Avoid |
|---|---|---|
| Unauthenticated | 401 | 200, 403 |
| Unauthorized (authenticated, no permission) | 403 | 401, 200 |
| Resource not found AND user unauthorized | 403 | 404 (prevents enumeration) |
| Successful delete | 204 | 200 with body |

---

## API Versioning & Lifecycle

- [ ] API version included in URL path (`/api/v1/`) or header
- [ ] Deprecated versions have announced sunset dates
- [ ] Breaking changes handled via new version (not silent changes)
- [ ] Old versions removed after deprecation period
- [ ] Changelog maintained for each API version

---

## Secrets & Configuration

- [ ] No API keys, DB credentials, or secrets in codebase
- [ ] Secrets managed via vault (HashiCorp Vault, AWS Secrets Manager, etc.)
- [ ] Environment-specific configs separated
- [ ] Secrets rotation automated
- [ ] Access logs don't capture Authorization header values

---

## Logging & Monitoring

- [ ] All requests logged: method, path, user ID, IP, response status, latency
- [ ] Authentication failures alerted in real-time
- [ ] Anomalous patterns trigger automated alerts (spike in 401s, 403s, 404s)
- [ ] Logs stored in append-only, tamper-resistant storage
- [ ] PII masked/redacted in logs
- [ ] Distributed tracing implemented (correlation IDs in requests)

---

## API Documentation Security

- [ ] Swagger/OpenAPI documentation not publicly exposed in production
- [ ] Internal API docs protected with authentication
- [ ] Sensitive endpoints not documented in public-facing specs
- [ ] Schema examples don't contain real data

---

*Last Updated: 2024 | Tools: Burp Suite, Postman, 42Crunch, OWASP ZAP*
