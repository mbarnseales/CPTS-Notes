
# Google Dorks Reference

Quick reference for Google search operators and common dork patterns. For workflow and what to look for see [[01-Recon/Web/Google-Dorking|Google Dorking]].

GHDB: `https://www.exploit-db.com/google-hacking-database`

---

## Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `site:` | Limit results to a domain | `site:example.com` |
| `inurl:` | Term must appear in URL | `inurl:login` |
| `allinurl:` | All terms must appear in URL | `allinurl:admin panel` |
| `intitle:` | Term must appear in page title | `intitle:"index of"` |
| `allintitle:` | All terms must appear in title | `allintitle:confidential report 2024` |
| `intext:` | Term must appear in body text | `intext:"password reset"` |
| `allintext:` | All terms must appear in body | `allintext:admin password reset` |
| `filetype:` | Specific file type | `filetype:pdf` |
| `ext:` | Specific file extension | `ext:conf` |
| `cache:` | Google's cached version of page | `cache:example.com` |
| `link:` | Pages linking to a URL | `link:example.com` |
| `related:` | Sites related to a URL | `related:example.com` |
| `info:` | Summary info about a URL | `info:example.com` |
| `" "` | Exact phrase match | `"information security policy"` |
| `AND` | Both terms required | `site:example.com AND inurl:admin` |
| `OR` | Either term | `inurl:login OR inurl:admin` |
| `NOT` / `-` | Exclude term | `site:example.com -inurl:login` |
| `*` | Wildcard | `site:example.com filetype:pdf user* manual` |
| `..` | Numerical range | `"price" 100..500` |
| `numrange:` | Number range | `numrange:1000-2000` |

---

## Dork Cheatsheet

### Scope and Mapping
```
site:example.com
site:example.com inurl:wp-content          # WordPress
site:example.com inurl:wp-admin            # WordPress admin
site:example.com inurl:/api/               # API endpoints
intitle:"index of" site:example.com        # Directory listings
```

### Login and Admin Panels
```
site:example.com inurl:login
site:example.com inurl:admin
site:example.com inurl:portal
site:example.com inurl:dashboard
site:example.com (inurl:login OR inurl:admin OR inurl:portal)
```

### Exposed Documents
```
site:example.com filetype:pdf
site:example.com filetype:xls OR filetype:xlsx
site:example.com filetype:doc OR filetype:docx
site:example.com filetype:ppt OR filetype:pptx
site:example.com filetype:txt
```

### Config and Env Files
```
site:example.com ext:env
site:example.com ext:conf OR ext:cnf OR ext:cfg
site:example.com ext:yml OR ext:yaml
site:example.com inurl:config.php
site:example.com inurl:web.config
site:example.com inurl:settings.py
site:example.com inurl:database.yml
```

### Backup and Database Files
```
site:example.com filetype:sql
site:example.com ext:bak OR ext:old OR ext:backup OR ext:orig
site:example.com inurl:backup
site:example.com ext:zip OR ext:tar OR ext:gz inurl:backup
```

### Credentials and Keys
```
site:example.com intext:"password"
site:example.com intext:"api_key"
site:example.com intext:"secret_key"
site:example.com intext:"BEGIN RSA PRIVATE KEY"
site:example.com intext:"DB_PASSWORD"
```

### Log and Error Files
```
site:example.com ext:log
site:example.com inurl:error_log
site:example.com intext:"Warning: mysql_"         # PHP MySQL error
site:example.com intext:"Fatal error"             # PHP fatal error -- reveals paths
site:example.com intext:"stack trace"
```

### Version and Technology Disclosure
```
site:example.com intext:"powered by"
site:example.com intitle:"phpinfo()"
site:example.com inurl:phpinfo.php
site:example.com intext:"Apache/2" intitle:"test page"
```

### Network and Infrastructure
```
site:example.com intext:"internal use only"
site:example.com intext:"not for public"
site:example.com intitle:"VPN" inurl:login
site:example.com inurl:citrix
site:example.com inurl:remote
```
