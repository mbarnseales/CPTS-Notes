
# Tech Fingerprinting

Active recon. Identify the web server, CMS, frameworks, and client-side technologies running on the target. Shapes everything that follows -- exploits, wordlists, and attack paths all depend on what's actually running.

See Also: [[00-Methedology/Web-Recon|Web Recon Methodology]], [[_Tools/Whatweb|WhatWeb]], [[_Tools/Nmap|Nmap]]

---

## Workflow

1. Banner grab with `curl -I` -- follow any redirects manually, each hop can reveal new info
2. Check for WAF with `wafw00f` -- know what you're up against before probing further
3. Run `whatweb` for automated tech stack identification
4. Run `nikto -Tuning b` for deeper software identification and header analysis
5. Check Wappalyzer / BuiltWith in browser for quick passive confirmation
6. Note all identified technologies and version numbers for vuln research

---

## Banner Grabbing -- curl

```bash
# Fetch headers only (no body)
curl -I <target>

# Follow each redirect manually -- each hop may reveal different headers
curl -I http://<target>
curl -I https://<target>
curl -I https://www.<target>
```

> [!tip] Follow the Redirects
> Don't just grab the first response. Each redirect can expose different headers. A bare domain redirecting to www often reveals the CMS doing the redirect via `X-Redirect-By`.

---

## Key HTTP Headers to Read

| Header | What It Reveals |
|--------|----------------|
| `Server` | Web server software and version (e.g. `Apache/2.4.41 (Ubuntu)`) |
| `X-Powered-By` | Backend language or framework (e.g. `PHP/7.4`, `ASP.NET`) |
| `X-Redirect-By` | What issued the redirect -- often reveals CMS (e.g. `WordPress`) |
| `Link` | Paths containing `wp-json` or `wp/v2` confirm WordPress |
| `Set-Cookie` | Cookie names like `PHPSESSID`, `JSESSIONID`, `wordpress_*` identify platforms |
| `X-Generator` | Sometimes explicitly states the CMS or framework |

### Missing Security Headers -- Note as Findings

| Missing Header | Risk |
|----------------|------|
| `Strict-Transport-Security` | No HSTS -- HTTPS not enforced |
| `X-Content-Type-Options` | Browser may sniff MIME types |
| `Content-Security-Policy` | No CSP -- XSS impact is higher |
| `X-Frame-Options` | Clickjacking potential (deprecated but still relevant) |

---

## WAF Detection -- wafw00f

Run before probing further. A WAF may block or log your fingerprinting attempts.

```bash
# Install
pip3 install git+https://github.com/EnableSecurity/wafw00f

# Detect WAF
wafw00f <target>
```

Identifies WAF vendor and product (e.g. Wordfence, Cloudflare, AWS WAF). If a WAF is present, note it -- you may need to adapt techniques to avoid triggering it.

---

## WhatWeb

```bash
whatweb <target>
```

See [[_Tools/Whatweb|WhatWeb]] for full usage and flags.

---

## Nikto -- Software Identification

```bash
# Fingerprinting only (Software Identification modules)
nikto -h <target> -Tuning b
```

Returns: server version, CMS detection, exposed files (`license.txt`, `readme.html`, `wp-login.php`), and missing security headers. Also flags outdated software versions.

Useful files to note if found:
- `/license.txt` -- may identify CMS and version
- `/readme.html` -- common in WordPress, reveals version
- `/wp-login.php` -- confirms WordPress, direct brute force target
- `/robots.txt` -- may list hidden paths
- `/.git/` -- exposed git repo

---

## Browser Tools

- **Wappalyzer** -- browser extension, passive, identifies tech stack from page load
- **BuiltWith** -- `https://builtwith.com/<domain>` -- detailed tech stack report, no interaction with target

Good for a quick passive confirmation before running active tools.

---

## What to Look For

- **Server version** -- cross-reference against known CVEs. Apache 2.4.41, Nginx 1.14, IIS 7.5 etc. all have public vulnerabilities.
- **WordPress / Drupal / Joomla detected** -- CMS-specific attack paths open up. Enumerate plugins, themes, and users next.
- **`X-Powered-By: PHP/x.x`** -- older PHP versions have known RCE and type juggling issues.
- **WAF present** -- Wordfence, Cloudflare, ModSecurity all have different bypass approaches. Note the vendor.
- **No WAF detected** -- fewer obstacles for active probing and fuzzing.
- **Missing HSTS** -- man-in-the-middle and SSL stripping attacks are viable.
- **Exposed `/wp-login.php`** -- credential brute force target, especially if no lockout is in place.
- **`license.txt` or `readme.html` accessible** -- often explicitly states CMS version number.
