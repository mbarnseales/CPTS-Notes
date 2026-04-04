
# Web Crawling

Active recon. Systematically follow links across a target site to map its structure, discover hidden pages, and extract data. Different from directory fuzzing -- crawling follows real links that exist, fuzzing guesses paths that might exist.

See Also: [[00-Methedology/Web-Recon|Web Recon Methodology]]

---

## Workflow

1. Start with the target's root URL as the seed
2. Run crawler -- collect all internal links, comments, metadata, and file references
3. Review extracted links for patterns -- unusual directories, unlinked pages, admin paths
4. Check for exposed sensitive files in discovered paths
5. Cross-reference findings -- a comment mentioning "file server" + a `/files/` directory = investigate immediately
6. Feed discovered URLs into directory fuzzer for deeper coverage

---

## What to Extract

| Data Type | What to Look For |
|-----------|-----------------|
| **Internal links** | Hidden pages, admin panels, unlinked endpoints, unusual directory names |
| **External links** | Third-party services in use, CDN providers, potential for subdomain takeover |
| **Comments in source** | Developer notes, version hints, internal hostnames, credentials left in HTML |
| **Metadata** | Author names, software versions, dates -- can reveal CMS version or internal usernames |
| **Sensitive files** | `.bak`, `.old`, `.config`, `web.config`, `settings.php`, `error_log`, `access_log` |

---

## Sensitive File Extensions to Flag

```
.bak .old .orig .backup    # Backup files -- may contain source code or configs
.config .conf .cfg         # Configuration files -- often contain credentials
.log .txt                  # Log files -- errors, access logs, debug output
.sql .db                   # Database dumps
.env                       # Environment files -- API keys, DB passwords
.git/                      # Exposed git repo -- full source code history
```

---

## robots.txt

Always check `/robots.txt` before or during crawling. It's a plain text file at the site root that tells crawlers what they're not supposed to access -- which from a recon perspective is a list of interesting paths the admin wants hidden.

```bash
curl https://<target>/robots.txt
```

**Common directives:**

| Directive | Meaning |
|-----------|---------|
| `Disallow: /admin/` | Path the owner doesn't want indexed -- high interest |
| `Allow: /public/` | Explicitly permitted -- usually less interesting |
| `Sitemap: https://example.com/sitemap.xml` | Free map of the site's content -- pull it |
| `Crawl-delay: 10` | Rate limit hint -- less relevant for pentest, but note it |

**Example:**
```
User-agent: *
Disallow: /admin/
Disallow: /private/
Disallow: /backup/
Sitemap: https://www.example.com/sitemap.xml
```

Every `Disallow` entry is a path worth manually visiting. The `Sitemap` URL gives you a structured list of all indexed pages -- fetch it and add to your URL list.

> [!note]
> robots.txt is advisory -- bots are expected to respect it but nothing enforces it. As a pentester, read it for intel but don't feel bound by it.

---

## Tools

### ReconSpider (Scrapy-based)

HTB's custom Scrapy spider. Extracts emails, links, external files, JS files, images, form fields, and HTML comments in one run.

```bash
# Install Scrapy
pip3 install scrapy

# Download ReconSpider
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip

# Run against target
python3 ReconSpider.py http://<target>
```

Output is saved to `results.json`. Key fields:

| Field | What It Contains |
|-------|-----------------|
| `emails` | Email addresses found on the domain |
| `links` | All internal and external URLs |
| `external_files` | PDFs and other external file URLs |
| `js_files` | JavaScript file URLs (check for API keys, endpoints, frameworks) |
| `form_fields` | Form inputs found across the site |
| `comments` | HTML comments in source -- developer notes, version hints |
| `images` / `videos` / `audio` | Media file URLs |

```bash
# Quick look at results
cat results.json | jq .

# Pull just emails
cat results.json | jq '.emails[]'

# Pull just comments
cat results.json | jq '.comments[]'

# Pull external files
cat results.json | jq '.external_files[]'
```

### Other Crawlers

| Tool | Use Case |
|------|---------|
| Burp Suite Spider | Integrated crawler inside Burp -- good for app mapping during manual testing |
| OWASP ZAP Spider | Free, open-source, GUI and CLI modes |
| Scrapy | Python framework for building custom crawlers when you need more control |

---

## What to Look For

- **Directory browsing enabled** -- a path like `/files/` returning a directory listing is an immediate find. Pull everything.
- **Backup files in web root** -- `index.php.bak`, `config.old` etc. often contain plaintext credentials or commented-out code.
- **Admin or internal paths** -- `/admin/`, `/internal/`, `/staging/` pages that appear in links but aren't publicly advertised.
- **Consistent URL patterns** -- if you see `/user/1`, `/user/2`, check for IDOR. Crawlers surface these patterns.
- **Comments with sensitive info** -- database names, internal IPs, version numbers, TODO notes referencing vulnerabilities.
- **Exposed `.git/`** -- if `/.git/HEAD` returns a response, the entire git history may be downloadable. Check with `git-dumper`.
- **Links to subdomains** -- pages may link to `dev.target.com`, `api.target.com`, or `staging.target.com` not found during passive recon.
