# 🐳 Container & Kubernetes Security Checklist

> **Level:** Professional / Senior AppSec / Cloud Security Engineer
> **Use Case:** Container Security Review, Kubernetes Cluster Audit

---

## Container Image Security

- [ ] Base images pinned to specific digest (`@sha256:<hash>`, not `:latest`)
- [ ] Minimal base images used (distroless, alpine, scratch)
- [ ] Images scanned for CVEs before push (Trivy, Grype, Snyk)
- [ ] No secrets, credentials, or SSH keys in image layers
- [ ] `.dockerignore` configured to exclude sensitive files
- [ ] Multi-stage builds used to minimize final image size and attack surface
- [ ] Container runs as non-root user
- [ ] Image signing enforced (cosign + policy enforcement)
- [ ] Private registry used (not DockerHub public by default)
- [ ] Image scanning in CI/CD blocks Critical/High CVEs

---

## Dockerfile Hardening

- [ ] `FROM` uses pinned digest
- [ ] `RUN` commands use `--no-cache` for package managers
- [ ] No `ADD` with remote URLs (use `COPY` instead)
- [ ] No `chmod 777` or world-writable permissions
- [ ] No `sudo` installed
- [ ] `HEALTHCHECK` instruction defined
- [ ] Only one process per container
- [ ] `EXPOSE` documents only necessary ports

**Hardened Dockerfile Example:**
```dockerfile
FROM python:3.12-slim@sha256:<digest> AS base

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
COPY --chown=appuser:appgroup requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appgroup . .

USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s CMD curl -f http://localhost:8080/health || exit 1
CMD ["python", "app.py"]
```

---

## Kubernetes Pod Security

- [ ] `runAsNonRoot: true` set in securityContext
- [ ] `runAsUser` set to non-root UID (e.g., 1000)
- [ ] `readOnlyRootFilesystem: true` set
- [ ] `allowPrivilegeEscalation: false` set
- [ ] `privileged: false` explicitly set
- [ ] All Linux capabilities dropped: `drop: ["ALL"]`
- [ ] Only required capabilities added back
- [ ] `hostPID: false`, `hostIPC: false`, `hostNetwork: false`
- [ ] `hostPath` volumes restricted or avoided
- [ ] `seccompProfile` defined (RuntimeDefault or custom)
- [ ] AppArmor profile applied

**Secure Pod SecurityContext:**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  seccompProfile:
    type: RuntimeDefault
containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      privileged: false
      capabilities:
        drop:
          - ALL
```

---

## Kubernetes RBAC

- [ ] `cluster-admin` not assigned to workload service accounts
- [ ] Namespace-scoped roles preferred over cluster-scoped
- [ ] Each workload has its own dedicated service account
- [ ] Default service account token automounting disabled (`automountServiceAccountToken: false`)
- [ ] RBAC reviewed with rbac-police or rakkess
- [ ] No wildcard permissions (`verbs: ["*"]`, `resources: ["*"]`)
- [ ] `system:masters` group not used for non-bootstrap users

---

## Network Policies

- [ ] Default deny NetworkPolicy applied in all namespaces
- [ ] Ingress and egress explicitly allowlisted per workload
- [ ] Pod-to-pod communication restricted to necessary paths
- [ ] Namespace-level isolation enforced
- [ ] External egress restricted to known endpoints

**Default Deny NetworkPolicy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

---

## Secrets Management in Kubernetes

- [ ] Kubernetes Secrets not stored in etcd without encryption
- [ ] etcd encryption at rest configured
- [ ] External secrets manager used (Vault, AWS Secrets Manager via ESO)
- [ ] Secrets mounted as volumes, not environment variables (where possible)
- [ ] RBAC restricts `get`/`list` on Secrets resources
- [ ] Secret values not logged in application or Kubernetes events
- [ ] Secret rotation process documented

---

## API Server Hardening

- [ ] `--anonymous-auth=false` on API server
- [ ] Admission controllers enabled: `NodeRestriction`, `PodSecurity`, `OPA Gatekeeper`
- [ ] Audit logging enabled on API server
- [ ] `--insecure-port=0` (disabled)
- [ ] RBAC authorization mode enabled
- [ ] API server not publicly accessible (private endpoint)
- [ ] kubeconfig files rotated and scoped per user/service

---

## Node Security

- [ ] Worker nodes hardened (CIS Kubernetes Benchmark)
- [ ] kubelet `--anonymous-auth=false`
- [ ] kubelet authorization mode set to `Webhook`
- [ ] Node OS patched and monitored
- [ ] Runtime security tool deployed (Falco)
- [ ] Container runtime: containerd or CRI-O (not Docker daemon)

---

## Runtime Security & Monitoring

- [ ] Falco deployed for runtime threat detection
- [ ] Custom Falco rules for:
  - Shell execution inside containers
  - Sensitive file access (`/etc/shadow`, `/proc/`)
  - Unexpected network connections
  - Privilege escalation attempts
- [ ] Container runtime logs shipped to SIEM
- [ ] Resource quotas and LimitRanges defined per namespace
- [ ] Horizontal Pod Autoscaler configured (DoS mitigation)

---

## Cluster Scanning Tools

| Tool | Purpose |
|---|---|
| kube-bench | CIS Kubernetes Benchmark compliance |
| kube-hunter | Kubernetes penetration testing |
| Trivy | Image + config scanning |
| Checkov | IaC / Kubernetes manifest scanning |
| Falco | Runtime security monitoring |
| rbac-police | RBAC policy analysis |
| Polaris | Kubernetes best practices audit |

---

*Last Updated: 2024 | Reference: CIS Kubernetes Benchmark, NSA Kubernetes Hardening Guide*
