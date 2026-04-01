# ⚡ GraphQL Security Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Use Case:** GraphQL API Security Review, Penetration Testing

---

## Introspection & Schema Exposure

- [ ] Introspection disabled in production environments
- [ ] GraphQL Playground / GraphiQL disabled in production
- [ ] Schema not served via `/__schema` or `/graphql?query={__schema{types{name}}}`
- [ ] Field suggestions (did you mean?) disabled in production (prevents enumeration)
- [ ] Error messages do not expose schema details

---

## Authentication & Authorization

- [ ] Authentication required on all non-public queries/mutations
- [ ] Authorization checked at resolver level (not just middleware)
- [ ] Field-level authorization implemented for sensitive data
- [ ] Object-level authorization enforced (BOLA equivalent)
- [ ] Batched queries don't bypass per-operation authorization checks

---

## Query Complexity & Depth Limiting

- [ ] Query depth limit set (recommended: max 5–7 levels)
- [ ] Query complexity scoring implemented
- [ ] Maximum query complexity limit enforced
- [ ] Aliases limited (prevent query batching abuse)
- [ ] Circular query protection in place

**Example Malicious Deep Query:**
```graphql
query {
  user {
    friends {
      friends {
        friends {
          friends { name }  # Deep nesting — can cause DoS
        }
      }
    }
  }
}
```

---

## Batching & Rate Limiting

- [ ] Query batching rate limited
- [ ] Per-operation rate limiting implemented (not just per HTTP request)
- [ ] Expensive resolvers tracked and throttled (N+1 protection via DataLoader)
- [ ] Mutation rate limiting stricter than query limits

---

## Injection Prevention

- [ ] All resolver inputs validated and sanitized
- [ ] No direct string interpolation into SQL/NoSQL queries from GraphQL args
- [ ] File upload mutations: file type, size, content validated
- [ ] SSRF possible via URL arguments in mutations — restricted

---

## Information Disclosure

- [ ] Error messages sanitized — no internal stack traces, query errors exposed
- [ ] Verbose error mode disabled in production
- [ ] Production errors return generic `Internal Server Error`
- [ ] Resolver errors logged server-side without exposing to client

---

## DoS Protection

- [ ] Large query rejection implemented (character/token limit)
- [ ] Timeout per query set
- [ ] Request body size limit enforced at gateway
- [ ] Field count per query limited
- [ ] Subscription connection limits enforced

---

## Persisted Queries

- [ ] Automatic Persisted Queries (APQ) used where possible
- [ ] Only allowlisted queries accepted in high-security mode
- [ ] Ad-hoc queries disabled in production (if applicable)

---

## Testing Checklist

- [ ] Introspection attempted — returns 400/403 in prod
- [ ] Deeply nested queries sent — rejected above depth limit
- [ ] Complexity bomb attempted — rejected
- [ ] Batch query abuse tested
- [ ] IDOR tested via resolver object fetching
- [ ] Field-level auth bypass tested (request admin fields as regular user)

---

## Recommended Tools

| Tool | Purpose |
|---|---|
| InQL (Burp Extension) | GraphQL security testing |
| Clairvoyance | Schema enumeration without introspection |
| graphql-cop | Automated GraphQL security checks |
| BatchQL | Batch query abuse testing |

---

*Last Updated: 2024 | Reference: OWASP GraphQL Security Cheat Sheet*
