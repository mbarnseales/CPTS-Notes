
# OSINT - Online Presence

Layer 1 of the [[Enumeration-Methodology|Enumeration Methodology]]. The goal is to map everything the target exposes on the internet before touching a single system directly.

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

---

## 1. SSL Certificate

Check the main site's certificate directly. Multi-domain certs (`Subject Alternative Names`) often list additional active subdomains.

View in browser: padlock > certificate > Details > Subject Alternative Name

---

## 2. Certificate Transparency (crt.sh)

Every certificate issued by a public CA is logged publicly. crt.sh queries those logs.

See [[DNS#Certificate Transparency|Certificate Transparency]] for the concept.

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
```

Save output to `subdomainlist` for the next step.

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

## Tools Summary

| Tool | Purpose |
|------|---------|
| crt.sh | Subdomain discovery via certificate transparency logs |
| `host` | Resolve subdomains to IPs |
| Shodan CLI | Passive port/service fingerprinting by IP |
| `dig any` | Pull all DNS record types |
| Browser cert viewer | Check SAN entries on main site certificate |
