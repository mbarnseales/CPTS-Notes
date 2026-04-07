# Nessus

## Scan Templates

| Category | Template | Use Case |
|---|---|---|
| **Discovery** | Host Discovery | Live hosts, open ports only |
| **Vulnerabilities** | Basic Network Scan | General-purpose vuln scan |
| **Vulnerabilities** | Advanced Scan | Full control over all settings |
| **Vulnerabilities** | Credentialed Patch Audit | Missing patches (requires creds) |
| **Vulnerabilities** | Web Application Tests | Web app vulns |
| **Vulnerabilities** | Malware Scan | Malware/backdoor detection |
| **Compliance** | Audit & Compliance | CIS/STIG benchmark checks |

---

## Key Scan Settings

### Discovery Tab

| Setting | Notes |
|---|---|
| **Fragile Devices** | Disable — scanning printers can cause them to print garbage and crash |
| **Port Range** | Common ports / All ports / Custom range |
| **Probe all ports** | On by default — may crash poorly written services |
| **SSL/TLS detection** | On by default — also flags expiring/revoked certs |
| **Ping methods** | TCP, ARP, ICMP (2 retries default) |

### Assessment Tab

| Setting | Notes |
|---|---|
| **Web App Scanning** | Enable if scoping includes web apps; set custom User-Agent if needed |
| **Brute Force** | Can use provided creds or wordlists; Hydra integration available |
| **User Enumeration** | SAM Registry, ADSI Query, WMI Query |
| **RID Brute Forcing** | Enumerate domain/local users by UID range (default 1000–1200) |

### Advanced Tab

| Setting | Notes |
|---|---|
| **Safe Checks** | On by default — skips checks that could crash targets |
| **Throttle on congestion** | Slows scan if network congestion detected |
| **Stop unresponsive hosts** | Drops hosts that stop responding mid-scan |
| **Randomise target order** | Shuffles IP list to reduce detection signatures |

---

## Credentialed vs Uncredentialed

| Type | What It Finds |
|---|---|
| **Uncredentialed** | Network-exposed vulns, open ports, service banners, missing patches inferred from version |
| **Credentialed** | Actual installed software, local config issues, missing patches confirmed, deeper audit |

> Credentialed scans are significantly more accurate — fewer false positives on patch status.

---

## Severity Levels

| Level | CVSS Range |
|---|---|
| Critical | 9.0 – 10.0 |
| High | 7.0 – 8.9 |
| Medium | 4.0 – 6.9 |
| Low | 0.1 – 3.9 |
| Info | N/A |

---

## Scan Policies

Custom saved scan configs — appear as **User Defined** templates when creating a new scan.

- Use for: evasive scans, web-focused scans, per-client credential sets
- Can be exported/imported between Nessus instances
- Base on **Advanced Scan** for full control with no pre-configured defaults

---

## Plugins

- Written in **NASL** (Nessus Attack Scripting Language)
- Each plugin maps to a CVE/BugtraqID and includes: vuln name, impact, remediation, detection method
- Can enable/disable entire plugin families or individual plugins per policy

### Plugin Rules

Suppress false positives without disabling the plugin globally:

1. **Resources → Plugin Rules → New Rule**
2. Set: Host, Plugin ID, (optional) Expiration Date, Action = `Hide this result`

> Use when a finding is by design for a specific host (e.g., DirectAccess null cipher suites, SSL self-signed cert on an internal CA).

---

## Credentials

### Host Authentication

| Method | Options |
|---|---|
| **SSH** | Password, Public Key, Certificate, Kerberos |
| **Windows** | Password, Kerberos, LM Hash, NTLM Hash |

> Enable "Do not use NTLMv1 authentication" — NTLMv1 is insecure.

### Database Authentication

Supported: Oracle, PostgreSQL, DB2, MySQL, SQL Server, MongoDB, Sybase

### Service Authentication (Plaintext)

FTP, HTTP, IMAP, IPMI, Telnet — configure login page + submission page for form-based auth.

### Confirming Credential Success

Look for info-level finding: **"Microsoft SQL Server login possible"** / **"Credentialed checks enabled"** — confirms creds worked.

---

## Scan Output & Reporting

### Export Formats

| Format | Use Case |
|---|---|
| **.pdf / .html** | Executive Summary or custom report — share with stakeholders |
| **.csv** | Select specific columns — import into Splunk, share with remediation teams, analytics |
| **.nessus** | XML — scan settings + plugin output — import into other tools/archive |
| **.db** | .nessus + KB + Audit Trail + attachments — full scan record |

> Always group vulnerabilities together in reports for clarity across affected assets.
> Nessus reports are **appendix/supplementary data only** — never the final client deliverable.

### CLI Export (nessus-report-downloader)

```bash
./nessus_downloader.rb
# Prompts: server IP, port (8834), username, password
# Select scan ID, file type (0=Nessus, 1=HTML, 2=PDF, 3=CSV, 4=DB), output path
```

### EyeWitness Integration

Pass `.nessus` file to EyeWitness to auto-screenshot all discovered web apps:

```bash
eyewitness --nessus -f scan.nessus -d /output/screenshots
```

---

## Scanning Issues & Mitigations

| Problem | Fix |
|---|---|
| All ports showing open / no ports open | Advanced Scan → disable **Ping the remote host** (firewall returning ICMP Unreachable = false live host) |
| Target under heavy load | Reduce **Max Concurrent Checks Per Host** under Performance Options |
| Sensitive/legacy hosts | Exclude from scope or configure `nessusd.rules` |
| DoS-type checks firing | Always enable **Safe Checks** — disables plugins that can crash daemons |
| Printers crashing | Disable **Scan Network Printers** under Fragile Devices |
| Low bandwidth / congested links | Throttle scan; monitor impact with `vnstat` |

```bash
# Monitor bandwidth impact during a scan
sudo vnstat -l -i eth0
# Idle: ~0 bit/s | During scan (single host): ~300 kbit/s rx, ~380 kbit/s tx
```

> A single-host scan generates ~1 MB rx + 1.3 MB tx in 38 seconds — significant on low-BW links.

### Pre-Scan Checklist

- Confirm scope with client — exclude sensitive/legacy/HA hosts or schedule separately (off-hours)
- Get written approval before scanning
- Keep detailed scan activity logs in case an incident needs investigating
- Never run DoS checks unless explicitly scoped

---

## Notes

- Free tier (Nessus Essentials) — limited to **16 hosts**
- Scans can take **1–2 hours** — review findings as they come in, don't wait for completion
- Re-running a scan on a live target counts against Essentials host limit
- Nessus REST API (port 8834) — can script exports, scan creation, and scheduling
