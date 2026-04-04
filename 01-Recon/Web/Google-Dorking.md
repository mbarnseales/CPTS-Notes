
# Google Dorking

Passive recon. Use Google search operators to surface information about a target that isn't directly visible -- exposed files, login pages, config files, credentials, and indexed sensitive content. No interaction with the target.

See Also: [[00-Methedology/Web-Recon|Web Recon Methodology]], [[_Reference/Google-Dorks|Google Dorks Reference]]

---

## Workflow

1. Start with `site:<target>` to see what Google has indexed
2. Narrow with `inurl:` to find admin, login, and API paths
3. Hunt for exposed files with `filetype:` and `ext:`
4. Look for config and backup files by name
5. Search for credentials and sensitive strings with `intext:`
6. Check the Google Hacking Database (GHDB) for target-specific dorks
7. Use `cache:` to retrieve old versions of pages that may have been cleaned up

---

## Core Operators

See full operator reference: [[_Reference/Google-Dorks|Google Dorks Reference]]

| Operator | Example | Purpose |
|----------|---------|---------|
| `site:` | `site:example.com` | Scope all results to one domain |
| `inurl:` | `inurl:admin` | Find specific strings in URLs |
| `filetype:` | `filetype:pdf` | Find specific file types |
| `intitle:` | `intitle:"index of"` | Find pages by title |
| `intext:` | `intext:"password"` | Find specific strings in page body |
| `ext:` | `ext:conf` | Find files by extension |
| `cache:` | `cache:example.com` | View Google's cached version |

---

## High-Value Dork Categories

### Login and Admin Pages
```
site:example.com inurl:login
site:example.com inurl:admin
site:example.com (inurl:login OR inurl:admin OR inurl:portal)
```

### Exposed Files
```
site:example.com filetype:pdf
site:example.com (filetype:xls OR filetype:xlsx OR filetype:csv)
site:example.com (filetype:doc OR filetype:docx)
```

### Config and Environment Files
```
site:example.com inurl:config.php
site:example.com (ext:conf OR ext:cnf OR ext:cfg OR ext:env)
site:example.com (ext:yml OR ext:yaml) inurl:config
```

### Database and Backup Files
```
site:example.com inurl:backup
site:example.com filetype:sql
site:example.com (ext:bak OR ext:old OR ext:backup)
```

### Credentials and Sensitive Strings
```
site:example.com intext:"password"
site:example.com intext:"api_key" OR intext:"api key" OR intext:"apikey"
site:example.com intext:"secret"
```

### Directory Listings
```
intitle:"index of" site:example.com
intitle:"index of /" "parent directory" site:example.com
```

---

## Google Hacking Database (GHDB)

Extensive collection of community-contributed dorks organised by category:
`https://www.exploit-db.com/google-hacking-database`

Categories to check first:
- **Files Containing Passwords**
- **Sensitive Directories**
- **Web Server Detection**
- **Vulnerable Files**
- **Error Messages** -- verbose errors often expose paths, versions, and DB info

---

## What to Look For

- **`intitle:"index of"`** -- directory listing enabled. Pull everything visible.
- **Login pages not linked from main nav** -- `/admin`, `/portal`, `/staff`, `/internal`
- **Exposed config files** -- `.env`, `config.php`, `settings.py` with real credentials
- **Backup files in web root** -- `.sql`, `.bak`, `.zip` containing source or DB dumps
- **PDF/Office docs** -- internal reports, org charts, network diagrams, contracts
- **Cached pages** -- previous versions may show content since removed or patched
- **Error messages with stack traces** -- reveal framework, file paths, DB engine
