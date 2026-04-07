
# Standards Reference

Compliance standards and penetration testing frameworks. Referenced during scoping, reporting, and when assessing environments subject to specific regulatory requirements.

---

## Compliance Standards

| Standard | Full Name | Who It Applies To | Key VA/Pentest Requirement |
|----------|-----------|-------------------|---------------------------|
| **PCI DSS** | Payment Card Industry Data Security Standard | Any org storing, processing, or transmitting cardholder data | Internal and external asset scanning required. CDE must be segmented and tested. |
| **HIPAA** | Health Insurance Portability and Accountability Act | Healthcare orgs handling patient data | Risk assessment and vulnerability identification required -- not necessarily a full scan |
| **FISMA** | Federal Information Security Management Act | US federal agencies and contractors | Documentation and proof of a vulnerability management program required |
| **ISO 27001** | International Organization for Standardization 27001 | Any org seeking international infosec accreditation | Quarterly internal and external scans required |

> [!note]
> Compliance should not drive a vulnerability management program. It sets a floor, not a ceiling. Assess based on actual risk appetite and environment uniqueness, not just what the standard requires.

---

## Penetration Testing Standards

### PTES -- Penetration Testing Execution Standard

General-purpose standard applicable to all pentest types.

| Phase | Description |
|-------|-------------|
| Pre-engagement Interactions | Scope, contracts, rules of engagement |
| Intelligence Gathering | Recon and OSINT |
| Threat Modeling | Identify likely attack paths |
| Vulnerability Analysis | Identify weaknesses |
| Exploitation | Attempt to exploit findings |
| Post Exploitation | Assess impact, lateral movement |
| Reporting | Document findings and remediation |

---

### OSSTMM -- Open Source Security Testing Methodology Manual

Structured around five testing channels:

| Channel | Scope |
|---------|-------|
| Human Security | Social engineering, physical access via people |
| Physical Security | Locks, barriers, physical infrastructure |
| Wireless Communications | WiFi, Bluetooth, RF |
| Telecommunications | Phone systems, VoIP |
| Data Networks | Network and application testing |

---

### NIST -- National Institute of Standards and Technology

| Phase | Description |
|-------|-------------|
| Planning | Scope, rules, logistics |
| Discovery | Recon, enumeration, vulnerability scanning |
| Attack | Exploitation |
| Reporting | Findings and remediation guidance |

---

### OWASP -- Open Web Application Security Project

Web and mobile application focused. Key resources:

| Guide | Scope |
|-------|-------|
| **WSTG** -- Web Security Testing Guide | Web application testing methodology |
| **MSTG** -- Mobile Security Testing Guide | iOS and Android application testing |
| **Firmware Security Testing Methodology** | Embedded device and firmware testing |

OWASP is the standard reference for web app risk classification and testing methodology. The OWASP Top 10 is the industry-standard list of critical web application risks.
