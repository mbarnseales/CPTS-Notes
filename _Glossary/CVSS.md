
# CVSS Reference

Common Vulnerability Scoring System. Industry standard for scoring vulnerability severity. Scores range from 0–10.

NVD Calculator: `https://nvd.nist.gov/vuln-metrics/cvss`

---

## Score Ranges

| Score | Severity |
|-------|----------|
| 0.0 | None |
| 0.1 – 3.9 | Low |
| 4.0 – 6.9 | Medium |
| 7.0 – 8.9 | High |
| 9.0 – 10.0 | Critical |

---

## Base Metric Group

Represents the intrinsic characteristics of a vulnerability. Stays constant regardless of environment or time.

### Exploitability Metrics

| Metric | Values | Notes |
|--------|--------|-------|
| **Attack Vector (AV)** | Network / Adjacent / Local / Physical | Network = highest score. No proximity to target needed. |
| **Attack Complexity (AC)** | Low / High | Low = exploit works reliably without special conditions |
| **Privileges Required (PR)** | None / Low / High | None = unauthenticated = higher score |
| **User Interaction (UI)** | None / Required | None = no victim action needed = higher score |

### Impact Metrics (CIA Triad)

| Metric | High | Low | None |
|--------|------|-----|------|
| **Confidentiality (C)** | Passwords, keys, all data exposed | Some data exposed, attacker has no control over what | No impact |
| **Integrity (I)** | Attacker can modify any/all files | Attacker can modify some files with limited control | No impact |
| **Availability (A)** | Complete denial of service | Partial denial, some resources still accessible | No impact |

**Scope (S):** Changed / Unchanged -- whether the vuln can impact components beyond its authorization scope. A Changed scope significantly increases the score.

---

## Temporal Metric Group

Adjusts the base score based on current exploit availability and patch status. Optional -- all values default to "Not Defined" (no change to score).

| Metric | Values | Meaning |
|--------|--------|---------|
| **Exploit Code Maturity (E)** | Not Defined / Unproven / PoC / Functional / High | High = automated exploit exists and works reliably |
| **Remediation Level (RL)** | Not Defined / Unavailable / Workaround / Temporary Fix / Official Fix | Official Fix = score reduced; Unavailable = score unchanged |
| **Report Confidence (RC)** | Not Defined / Unknown / Reasonable / Confirmed | Confirmed = multiple detailed sources validate the vuln |

---

## Environmental Metric Group

Adjusts the score for a specific organization's environment. Accounts for how critical CIA are to that particular org. Optional.

| Metric | Values | Use Case |
|--------|--------|---------|
| **Modified Base Metrics** | Not Defined / Low / Medium / High | Override any base metric if your environment changes the risk (e.g. AV is Network but the system is air-gapped) |
| **Confidentiality Requirement (CR)** | Not Defined / Low / Medium / High | High if data breach would be catastrophic for the org |
| **Integrity Requirement (IR)** | Not Defined / Low / Medium / High | High if data integrity is mission-critical |
| **Availability Requirement (AR)** | Not Defined / Low / Medium / High | High if downtime directly impacts operations or safety |

---

## DREAD Model

Microsoft's supplementary risk scoring model. Five factors rated 0–10, averaged for a final score.

| Factor | Question |
|--------|---------|
| **Damage Potential** | How bad is the damage if exploited? |
| **Reproducibility** | How easy is it to reproduce the attack? |
| **Exploitability** | How easy is it to exploit? |
| **Affected Users** | How many users are impacted? |
| **Discoverability** | How easy is it to discover the vulnerability? |

Often used alongside CVSS, not as a replacement. DREAD adds a discoverability and affected-user dimension that pure CVSS doesn't capture.

---

## Practical Notes

- **Temporal metrics matter for prioritisation** -- a CVSS 9.8 with no public exploit (Unproven) may be less urgent than a CVSS 7.5 with a Functional public exploit and no patch available.
- **Environmental metrics are underused** -- a "Critical" finding on an isolated dev box with no sensitive data is not the same risk as the same finding on an internet-facing production server.
- **Scope = Changed** -- any vuln that can jump from a sandboxed process to the OS, or from a plugin to the host app, will have Scope = Changed and a significantly higher score.
