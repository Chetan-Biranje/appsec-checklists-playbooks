# 🛡️ AppSec Checklists & Playbooks

> Application Security checklists, playbooks and response guides that I collect and refine while learning AppSec. Useful for code reviews, assessments and incident response practice.

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Maintained: Yes](https://img.shields.io/badge/Maintained-Yes-green.svg)
![Topics: AppSec](https://img.shields.io/badge/Topics-Web%20%7C%20API%20%7C%20Mobile%20%7C%20Cloud-orange.svg)

---

## 📌 Overview

This repository provides structured, actionable security checklists and incident response playbooks across four critical domains:

| Domain | Checklists | Playbooks |
|---|---|---|
| 🌐 Web Application Security | [View](./checklists/web/) | [View](./playbooks/web/) |
| 🔌 API Security | [View](./checklists/api/) | [View](./playbooks/api/) |
| 📱 Mobile App Security | [View](./checklists/mobile/) | [View](./playbooks/mobile/) |
| ☁️ Cloud Security / DevSecOps | [View](./checklists/cloud/) | [View](./playbooks/cloud/) |

---

## 📂 Repository Structure

```
appsec-checklists-playbooks/
│
├── checklists/
│   ├── web/
│   │   ├── owasp-top10-checklist.md
│   │   ├── secure-code-review.md
│   │   └── penetration-testing-checklist.md
│   ├── api/
│   │   ├── owasp-api-top10-checklist.md
│   │   ├── rest-api-security-checklist.md
│   │   └── graphql-security-checklist.md
│   ├── mobile/
│   │   ├── owasp-masvs-checklist.md
│   │   ├── android-security-checklist.md
│   │   └── ios-security-checklist.md
│   └── cloud/
│       ├── devsecops-pipeline-checklist.md
│       ├── aws-security-checklist.md
│       └── container-kubernetes-checklist.md
│
├── playbooks/
│   ├── web/
│   │   ├── sqli-response-playbook.md
│   │   ├── xss-response-playbook.md
│   │   └── broken-auth-playbook.md
│   ├── api/
│   │   ├── api-abuse-playbook.md
│   │   └── broken-object-level-auth-playbook.md
│   ├── mobile/
│   │   ├── reverse-engineering-response-playbook.md
│   │   └── data-leakage-playbook.md
│   └── cloud/
│       ├── cloud-misconfiguration-playbook.md
│       └── secrets-exposure-playbook.md
│
├── templates/
│   ├── checklist-template.md
│   ├── playbook-template.md
│   └── vuln-report-template.md
│
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## 🚀 How to Use

### For Security Engineers
- Use **checklists** during code reviews, security assessments, and pre-deployment gates.
- Use **playbooks** during incident response and threat hunting activities.

### For DevSecOps Teams
- Integrate checklist items into CI/CD pipeline gates.
- Refer to cloud checklists for IaC (Infrastructure as Code) reviews.

### For Pentesters / Red Teams
- Use domain-specific checklists as structured test plans.
- Track coverage per OWASP category.

---

## 🗺️ Standards & Frameworks Covered

- [OWASP Top 10 (Web)](https://owasp.org/www-project-top-ten/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [OWASP MASVS (Mobile)](https://owasp.org/www-project-mobile-app-security/)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [CWE/CVE References](https://cwe.mitre.org/)
- [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before submitting PRs.

---

## 📄 License

This project is licensed under the [MIT License](./LICENSE).

---

> Maintained by the AppSec Community. Star ⭐ this repo if it helped you!
