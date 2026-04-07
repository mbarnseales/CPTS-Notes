
# Vulnerability Assessment

A structured process to identify, categorise, and prioritise security weaknesses in an environment. **Little to no manual exploitation** -- the goal is to understand and communicate risk, not gain access. Remediation guidance is a core deliverable.

> [!note] Scope Clarification
> Before starting, confirm with the client whether they want:
> (a) scanner output only, or
> (b) minimal exploitation to validate findings and rule out false positives

See Also: [[_Glossary/Security-Concepts|Security Concepts]] for Vulnerability / Threat / Risk definitions

---

## Methodology

| Step | Description |
|------|-------------|
| 1. Risk Identification & Analysis | Identify assets, understand what needs protecting and why |
| 2. Develop Scanning Policies | Define scope, timing, credentials, exclusions |
| 3. Identify Scan Types | Credentialed vs uncredentialed, internal vs external, network vs web app |
| 4. Configure the Scan | Set targets, credentials, scan templates, and intensity |
| 5. Perform the Scan | Run scanner, monitor for disruptions or false positives |
| 6. Evaluate Risks | Apply the risk matrix -- score by likelihood and impact |
| 7. Interpret Results | Remove false positives, correlate findings, add context |
| 8. Create Remediation Plan | Prioritise by risk score, assign owners, set timelines |

---

## Scan Types

| Type | Description |
|------|-------------|
| **Uncredentialed** | Scanner has no login. Sees what an unauthenticated attacker sees. Faster but less thorough -- misses many local vulns. |
| **Credentialed** | Scanner authenticates to targets. Sees installed software, patch levels, config files. Far more accurate. |
| **Internal** | Run from inside the network. Finds what's exposed to a compromised internal host or insider threat. |
| **External** | Run from outside. Finds what's exposed to the internet. |
| **Network** | Scans hosts, ports, and services across the network. |
| **Web Application** | Targets web apps specifically -- OWASP Top 10 class issues. |

---

## Static vs Dynamic Testing

| Type | How It Works | Limitation |
|------|-------------|------------|
| **Static** | Identifies vulnerability based on detected software version matching a known CVE | Inaccurate if a patch has been applied or the target isn't specifically affected |
| **Dynamic** | Sends benign payloads to the target (weak creds, SQLi, command injection) and observes the response | Slower, but confirms exploitability rather than just version matching |

Most scanners use a combination of both. Static gives breadth, dynamic gives confidence.

---

## Tools

| Tool | Notes |
|------|-------|
| **Nessus** | Industry standard. Free tier (Essentials) limited to 16 hosts. | [[_Tools/Nessus\|Nessus]] |
| **OpenVAS** | Open-source. Full-featured. No host limit. | [[_Tools/OpenVAS\|OpenVAS]] |
| **Qualys** | Cloud-based platform. Free community edition available. | -- |
| **Nexpose** | Rapid7's scanner. 30-day trial. Integrates with Metasploit. | -- |

---

## Key Concepts

See [[_Glossary/Security-Concepts|Security Concepts]] for full definitions.

- **CVSS** -- Common Vulnerability Scoring System. 0–10 scale. Used to standardise severity ratings across tools and reports. See [[_Glossary/CVSS|CVSS Reference]].
- **CVE** -- Common Vulnerabilities and Exposures. MITRE's registry of known vulnerabilities. Referenced as `CVE-YEAR-NUMBER`.
- **False Positive** -- Scanner flags an issue that doesn't actually exist. Always validate before reporting.
- **False Negative** -- Scanner misses a real vulnerability. Credentialed scans reduce these significantly.
- **Risk** -- Likelihood × Impact. A high-severity vuln in an isolated dev environment may be lower risk than a medium-severity vuln on an internet-facing production server.
