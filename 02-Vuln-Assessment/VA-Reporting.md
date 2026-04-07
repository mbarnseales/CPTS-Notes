# Vulnerability Assessment Reporting

## Report Structure

| Section | Audience | Purpose |
|---|---|---|
| **Executive Summary** | Non-technical (execs) | High-level findings, severity breakdown, immediate priorities |
| **Overview of Assessment** | Technical + management | Methodology, tools used, testing process |
| **Scope and Duration** | All | Authorised targets, testing window |
| **Vulnerabilities and Recommendations** | Technical | Full findings, evidence, remediation steps |

---

## Section Breakdown

### Executive Summary
- High-level overview of findings and risk posture
- Severity breakdown (graph/table of critical/high/medium/low counts)
- Top priorities for immediate remediation
- Written for someone with no technical background

### Overview of Assessment
- Methodology used (e.g., credentialed vs uncredentialed, internal vs external)
- Tools used (Nessus, OpenVAS, etc.)
- Testing approach and execution timeline

### Scope and Duration
- All client-authorised targets (IPs, ranges, domains)
- Testing start/end dates and times

### Vulnerabilities and Recommendations

Each finding must include:

| Field | Notes |
|---|---|
| **Vulnerability Name** | Descriptive, consistent naming |
| **CVE** | Reference ID if applicable |
| **CVSS Score** | Base score + severity rating |
| **Description** | What the issue is and why it matters |
| **References** | CVE links, vendor advisories, NVD |
| **Remediation Steps** | Specific, actionable steps |
| **Proof of Concept** | Screenshots, output, reproduction steps |
| **Affected Systems** | All impacted hosts |

> Group related findings together — by type or severity — for clarity.

---

## Key Principles

- **Eliminate false positives first** — manually verify before including in report
- **Write for any audience** — define technical terms, include references a reader can follow
- **Be concise** — clear sentences, proper grammar, no fluff
- **Never hand over raw scanner output** as the final deliverable — it's appendix/supplementary data only
