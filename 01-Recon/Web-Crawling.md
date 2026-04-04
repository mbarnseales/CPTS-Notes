
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

## Tools

*(Commands to be added as module progresses)*

---

## What to Look For

- **Directory browsing enabled** -- a path like `/files/` returning a directory listing is an immediate find. Pull everything.
- **Backup files in web root** -- `index.php.bak`, `config.old` etc. often contain plaintext credentials or commented-out code.
- **Admin or internal paths** -- `/admin/`, `/internal/`, `/staging/` pages that appear in links but aren't publicly advertised.
- **Consistent URL patterns** -- if you see `/user/1`, `/user/2`, check for IDOR. Crawlers surface these patterns.
- **Comments with sensitive info** -- database names, internal IPs, version numbers, TODO notes referencing vulnerabilities.
- **Exposed `.git/`** -- if `/.git/HEAD` returns a response, the entire git history may be downloadable. Check with `git-dumper`.
- **Links to subdomains** -- pages may link to `dev.target.com`, `api.target.com`, or `staging.target.com` not found during passive recon.
