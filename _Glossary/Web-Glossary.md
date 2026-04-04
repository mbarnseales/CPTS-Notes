
# Web Glossary

Core web concepts referenced throughout web recon, enumeration, and exploitation.

---

# Certificate Transparency Logs

Public, append-only ledgers that record every SSL/TLS certificate issued by a Certificate Authority (CA). When a CA issues a certificate, it must submit it to multiple CT logs. Anyone can query them.

**Why useful for recon:** CT logs contain a historical record of every subdomain that has ever had a certificate issued for it -- including subdomains that no longer have DNS records, dev/staging environments, and internal services that were briefly exposed. This makes CT logs one of the most reliable passive subdomain discovery sources.

Key tools:
- **crt.sh** -- `https://crt.sh/?q=<domain>` -- free, no registration, good for quick lookups
- **Censys** -- `https://search.censys.io` -- advanced filtering, requires free registration

See: [[01-Recon/Web/OSINT-Online-Presence|OSINT]] for usage commands.

---

# Virtual Hosts (VHosts)

A configuration on a web server that allows **multiple websites or applications to be hosted on a single server and IP address**. The server determines which site to serve based on the `Host` header in the incoming HTTP request.

```
Browser sends:  GET / HTTP/1.1
                Host: dev.example.com

Server reads the Host header → matches virtual host config → serves dev.example.com content
```

---

## VHosts vs Subdomains

Often confused but distinct concepts:

| | Subdomains | Virtual Hosts |
|-|-----------|---------------|
| **What it is** | A DNS record (e.g. `blog.example.com` → IP) | A web server configuration entry |
| **DNS required** | Yes -- needs an A/CNAME record | Not necessarily -- can work with only a hosts file entry |
| **Visibility** | Publicly resolvable | Can be entirely internal/hidden |
| **Scope** | DNS layer | Web server layer |

A virtual host can exist with **no DNS record at all**. This is why VHost fuzzing finds things that subdomain enumeration misses -- the subdomain may never have been registered in DNS but the web server still responds to that `Host` header.

---

## Types of Virtual Hosting

| Type | How It Works | Common? |
|------|-------------|---------|
| **Name-Based** | Server reads the `Host` header to route requests. One IP, many sites. | Most common |
| **IP-Based** | Each site gets its own IP address. No reliance on `Host` header. | Less common, expensive |
| **Port-Based** | Each site runs on a different port on the same IP (e.g. :80, :8080) | Rare, not user-friendly |

---

## Why It Matters for Recon

- **Hidden VHosts are not in DNS** -- standard subdomain enumeration won't find them. Requires VHost fuzzing.
- **Dev/staging/admin panels** are commonly configured as VHosts without public DNS records (e.g. `dev.example.com`, `admin.example.com`, `staging.example.com`).
- **Different VHosts = different attack surfaces** -- a dev VHost on the same server may run older software, have debug mode enabled, or expose internal APIs.

See: [[_Tools/Gobuster|Gobuster]] for VHost fuzzing commands.
