
# Web Recon

Systematic information gathering against a web target before active exploitation. Goal is to map the attack surface -- domains, subdomains, technologies, exposed files, and infrastructure -- with as little noise as possible.

**Passive first. Active second.**

---

## Passive Recon

No direct interaction with the target. Pull from public sources only. Low/no detection risk.

| Technique | What You're Looking For | Reference |
|-----------|------------------------|-----------|
| WHOIS | Registrant details, name servers, registration dates, admin contacts | [[_Tools/WHOIS\|WHOIS]] |
| DNS Records | A, MX, NS, TXT, CNAME. TXT records reveal third-party services and SPF config | [[01-Recon/DNS-Enumeration\|DNS Enumeration]] |
| Certificate Transparency | Subdomains from SSL cert history via crt.sh. Often finds what nothing else does | [[01-Recon/OSINT-Online-Presence\|OSINT]] |
| Web Archives | Old pages, removed endpoints, exposed configs, historical tech stack via Wayback Machine | [[01-Recon/Web-Archives\|Web Archives]] |
| Google Dorking | Exposed files, login pages, config files, and indexed sensitive content | [[01-Recon/Google-Dorking\|Google Dorking]] |
| GitHub / Code Repos | Leaked credentials, API keys, internal tooling, infrastructure details | [[01-Recon/GitHub-Recon\|GitHub Recon]] |
| Shodan / Censys | Open ports, service banners, TLS cert info -- external view without touching the target | [[01-Recon/OSINT-Online-Presence\|OSINT]] |

---

## Active Recon

Direct interaction with the target. Higher detection risk. Do after passive phase.

| Technique | What You're Looking For | Reference |
|-----------|------------------------|-----------|
| Technology Fingerprinting | Web server, framework, CMS, client-side libraries. Shapes the rest of the assessment | [[01-Recon/Tech-Fingerprinting\|Tech Fingerprinting]] |
| Subdomain Enumeration | Active DNS brute force to find subdomains not visible in CT logs or passive sources | [[01-Recon/Subdomain-Enumeration\|Subdomain Enumeration]] |
| Web Crawling | Map pages, directories, parameters, and hidden endpoints by spidering the app | [[01-Recon/Web-Crawling\|Web Crawling]] |
| Directory / File Fuzzing | Admin panels, backup files, config files, and unlinked content | [[_Tools/Gobuster\|Gobuster]] |
| HTTP Headers & Banners | Server version, security headers, cookies, and info disclosure in responses | [[01-Recon/Tech-Fingerprinting\|Tech Fingerprinting]] |

---

## What You're Building

By the end of recon you should have:

- Full domain and subdomain list with resolved IPs
- Tech stack identified (server, framework, CMS, JS libraries)
- Known endpoints, parameters, and any admin/login surfaces
- Any exposed sensitive files or historical data from archives
- Third-party services in use (and what data they handle)
- Candidate credentials or API keys from code repos
- A prioritised list of attack surface items to investigate further
