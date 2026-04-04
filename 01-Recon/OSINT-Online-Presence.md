
# OSINT - Online Presence

Layer 1 of the [[Enumeration-Methodology|Enumeration Methodology]]. The goal is to map everything the target exposes on the internet before touching a single system directly.

See Also: [[00-Methedology/Web-Recon|Web Recon Methodology]]

> [!info] Passive Only
> Everything in this section is passive. No direct connections to the target. Navigate as a visitor. Direct active scans begin at Layer 3.

> [!warning] Third-Party Scope
> Hosts resolving to third-party providers (AWS, Google, Cloudflare, etc.) are out of scope unless you have explicit permission from that provider. Identify and exclude them early.

---

## Workflow

1. Read the company's main website -- understand what services they offer and what technologies are required to deliver them. Think like a developer.
2. Pull subdomains from SSL certificate transparency logs (crt.sh)
3. Resolve subdomains to IPs -- filter for company-owned vs third-party hosted
4. Run IPs through Shodan for open ports and service fingerprints
5. Query all DNS records -- read TXT records for third-party provider intelligence
6. Search for exposed cloud storage (S3, Azure Blob, GCP) via Google Dorks and GrayHatWarfare
7. Check website source code for cloud storage references loaded as assets
8. Search LinkedIn/Xing for technical employees and job postings -- extract tech stack and tooling

---

## 1. SSL Certificate

Check the main site's certificate directly. Multi-domain certs (`Subject Alternative Names`) often list additional active subdomains.

View in browser: padlock > certificate > Details > Subject Alternative Name

---

## 2. Certificate Transparency (crt.sh)

Every certificate issued by a public CA is logged publicly. crt.sh queries those logs. Finds subdomains that DNS brute force and wordlists miss -- including historical subdomains from expired certs.

See [[_Glossary/Web-Glossary#Certificate Transparency Logs|CT Logs]] for the concept.

```bash
# Browse manually
https://crt.sh/?q=<domain>

# JSON output
curl -s "https://crt.sh/?q=<domain>&output=json" | jq .

# Extract unique subdomains only
curl -s "https://crt.sh/?q=<domain>&output=json" \
  | jq . \
  | grep name \
  | cut -d":" -f2 \
  | grep -v "CN=" \
  | cut -d'"' -f2 \
  | awk '{gsub(/\\n/,"\n");}1;' \
  | sort -u

# Filter results by keyword (e.g. find only dev subdomains)
curl -s "https://crt.sh/?q=<domain>&output=json" \
  | jq -r '.[] | select(.name_value | contains("dev")) | .name_value' \
  | sort -u
```

Save output to `subdomainlist` for the next step.

**Censys** is a deeper alternative -- advanced filtering by domain, IP, and certificate attributes. Requires free registration. Useful when crt.sh results are thin or you need to pivot on certificate attributes.
`https://search.censys.io`

---

## 3. Resolve Subdomains -- Filter Company-Hosted

```bash
# Resolve each subdomain and keep only those pointing to company IPs
for i in $(cat subdomainlist); do
  host $i | grep "has address" | grep <domain> | cut -d" " -f1,4
done

# Extract IPs only into a file for Shodan
for i in $(cat subdomainlist); do
  host $i | grep "has address" | grep <domain> | cut -d" " -f4 >> ip-addresses.txt
done
```

Any result resolving to a third-party range (e.g. `amazonaws.com`, `cloudflare.com`) gets flagged and excluded from active testing unless scoped.

---

## 4. Shodan

Query each company-owned IP for open ports, services, and versions without sending a single packet to the target.

```bash
for i in $(cat ip-addresses.txt); do shodan host $i; done
```

Shodan output gives: city, org, last updated, open ports, service banners, TLS versions. Look for:
- Unexpected open ports
- Outdated service versions
- Disabled TLS versions (indicates older configs)

Shodan requires an API key: `shodan init <API_KEY>`

---

## 5. DNS Records

```bash
# Query all record types at once
dig any <domain>
```

See [[DNS]] for a full breakdown of record types. What to extract from each:

| Record | What It Tells You |
|--------|-------------------|
| `A` | IP addresses the domain resolves to |
| `MX` | Mail provider -- note it, skip testing unless in scope |
| `NS` | Name server / hosting provider |
| `TXT` | Third-party services in use, email config (SPF, DKIM, DMARC) |
| `SOA` | Primary DNS contact -- often reveals registrar or IT provider |

---

## 6. Reading TXT Records for Intel

TXT records contain verification tokens from every third-party service the company has connected. Each token reveals a platform in use.

| TXT Prefix | Service | Intel Value |
|------------|---------|-------------|
| `MS=ms...` | Microsoft (Office 365 / Azure) | Likely OneDrive, Azure Blob, SMB-accessible file storage |
| `atlassian-domain-verification=...` | Atlassian (Jira, Confluence) | Dev collaboration platform -- project data, internal wikis |
| `google-site-verification=...` | Google Workspace | Gmail in use -- possible open GDrive folders/files |
| `logmein-verification-code=...` | LogMeIn | Centralised remote access -- high-value target if credentials are obtained |
| `v=spf1 include:mailgun.org ...` | Mailgun | Email API in use -- look for IDOR, SSRF, exposed API endpoints |
| `include:spf.protection.outlook.com` | Outlook / Exchange Online | Document management, cloud storage |

SPF records also leak internal IPs -- the `ip4:` entries in an SPF record are often internal mail relay addresses. Note them.

```bash
# Example SPF with leaked IPs
"v=spf1 include:mailgun.org include:_spf.google.com ip4:10.129.24.8 ip4:10.129.27.2 ~all"
#                                                        ^internal     ^internal
```

---

## 7. Cloud Storage

Cloud misconfigurations are common. Even when a provider secures their infrastructure, the company's own bucket/blob permissions may be open to the public. Look for storage endpoints surfaced through DNS resolution, source code, and search engines.

> [!warning] Scope
> Cloud infrastructure belongs to a third-party provider. Do not interact with it without explicit written permission from that provider, even if the company contracted you.

### Identify Cloud Storage via DNS

When resolving subdomains, any result pointing to a cloud provider domain instead of a company IP is a cloud-hosted resource. Note it separately.

```bash
# During subdomain resolution, watch for entries like:
s3-website-us-west-2.amazonaws.com
*.blob.core.windows.net
storage.googleapis.com
```

### Google Dorks

```bash
# AWS S3
intext:"<company>" inurl:amazonaws.com

# Azure Blob
intext:"<company>" inurl:blob.core.windows.net

# GCP
intext:"<company>" inurl:storage.googleapis.com
```

Results often surface PDFs, documents, presentations, and source code stored in misconfigured buckets.

### Website Source Code

Cloud storage is frequently used to serve static assets (images, JS, CSS). Inspect page source for references to cloud storage domains -- these are direct bucket/blob URLs.

```bash
# In browser: Ctrl+U to view source, then search for:
amazonaws.com
blob.core.windows.net
storage.googleapis.com
```

### domain.glass

[domain.glass](https://domain.glass) is a third-party infrastructure lookup tool. Useful for confirming whether a target sits behind Cloudflare. A "Safe" Cloudflare classification at this stage means you've already identified a Layer 2 (Gateway) security measure before active scanning begins -- note it.

### GrayHatWarfare

[GrayHatWarfare](https://buckets.grayhatwarfare.com) indexes publicly accessible cloud storage across AWS, Azure, and GCP. Search by company name or abbreviation. Filter by file type.

High-value finds: SSH private keys (`id_rsa`), config files, credentials, internal documents. Private keys found in public buckets can give direct SSH access to company infrastructure.

### Company Name Variations

Try abbreviations and common variations of the company name -- IT infrastructure often uses shortened names that differ from the public-facing brand.

---

## 8. Staff OSINT

Employees reveal the tech stack -- often more accurately than any scan. Job postings list required technologies explicitly. Employee profiles and public code show what's actually in use day-to-day.

### What to Extract from Job Postings

| Category | What It Reveals |
|----------|----------------|
| Languages | Runtime environments on servers (Java, Python, PHP, etc.) |
| Databases | DB engines to target during exploitation (MySQL, PostgreSQL, Oracle, MSSQL) |
| Frameworks | Web app structure and known misconfig patterns (Django, Flask, Spring, ASP.NET) |
| Tools / Platforms | Internal services to look for (Atlassian, Git providers, CI/CD systems) |
| Certifications required | Indicates what security tooling/standards the company follows |

### What to Extract from Employee Profiles

- **Skills listed** -- confirms technologies in active use
- **Career history / project descriptions** -- reveals specific systems and internal product names
- **GitHub links** -- public repos may contain hardcoded secrets, JWT tokens, internal URLs, or config files with real values
- **Posts and shared content** -- shows what the employee is currently working on

### Who to Look For

Focus on technical employees in development and security roles. Security employees reveal what defensive tooling the company has deployed. Developers reveal what's being built and how.

Search filters on LinkedIn: job title, company, location, skills. Narrow by "software engineer", "DevOps", "security engineer", "infrastructure".

### Finding Misconfigurations via Framework Research

Once a framework is identified (e.g. Django), search for its known OWASP misconfigurations. Companies that follow framework tutorials often name files and structure projects exactly as the documentation shows -- which means default paths and patterns are predictable.

> [!tip]
> Employee GitHub profiles linked from LinkedIn are high value. Look for config files, `.env` files, hardcoded API keys, JWT secrets, and internal hostnames committed to public repos.

---

## Tools Summary

| Tool | Purpose |
|------|---------|
| crt.sh | Subdomain discovery via certificate transparency logs |
| Censys | Advanced CT log search with IP/cert attribute filtering |
| `host` | Resolve subdomains to IPs |
| Shodan CLI | Passive port/service fingerprinting by IP |
| `dig any` | Pull all DNS record types |
| Browser cert viewer | Check SAN entries on main site certificate |
| Google Dorks | Find exposed cloud storage buckets/blobs |
| GrayHatWarfare | Browse indexed public cloud storage files |
| domain.glass | Third-party infrastructure lookup, Cloudflare detection |
| LinkedIn / Xing | Employee profiles, job postings, tech stack extraction |
| GitHub | Employee public repos -- secrets, configs, internal hostnames |
