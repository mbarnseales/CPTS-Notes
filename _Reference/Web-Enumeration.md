
# Web Enumeration Tips

## SSL/TLS Certificates
Browse to `https://<target>` and inspect the certificate. Can reveal: company name, email address, internal hostnames. Useful for phishing (if in scope) or further OSINT.

## robots.txt
`http://<target>/robots.txt` — tells search engines what not to index. Often reveals private directories and admin pages.

## Source Code
`CTRL + U` in browser to view page source. Check for:
- Developer comments with credentials
- Hidden fields
- Internal paths and endpoints
- API keys

## EyeWitness
Takes screenshots of web targets, fingerprints them, and flags possible default credentials. Useful for quickly triaging large numbers of web services.
