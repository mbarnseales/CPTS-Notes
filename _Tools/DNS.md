
# DNS Tools Reference

Quick command reference for DNS enumeration. For theory, workflow, and what to look for, see [[01-Recon/DNS-Enumeration|DNS Enumeration]]. For record type definitions, see [[_Glossary/DNS|DNS Glossary]].

Port: `53` (UDP default, TCP for zone transfers)

---

## dig

```bash
# Nameservers
dig ns <domain> @<target-dns>

# All available records
dig any <domain> @<target-dns>

# SOA record (admin contact)
dig soa <domain> @<target-dns>

# Specific record type
dig <type> <domain> @<target-dns>     # e.g. A, MX, TXT, CNAME

# Reverse lookup (PTR)
dig -x <ip> @<target-dns>

# BIND version (CHAOS class)
dig CH TXT version.bind <target-dns>

# Zone transfer (AXFR)
dig axfr <domain> @<target-dns>
dig axfr <subdomain>.<domain> @<target-dns>
```

---

## dnsenum

Full automated DNS enumeration -- NS, MX, AXFR attempt, and subdomain brute force:

```bash
dnsenum --dnsserver <target-dns> \
        --enum \
        -p 0 \
        -s 0 \
        -o subdomains.txt \
        -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
        <domain>
```

| Flag | Description |
|------|-------------|
| `--dnsserver` | Target DNS server |
| `--enum` | Full enumeration mode |
| `-p 0` | Disable Google scraping |
| `-s 0` | Disable scraping |
| `-o <file>` | Output file |
| `-f <wordlist>` | Wordlist for brute force |

---

## Subdomain Brute Force (Bash)

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do
  dig $sub.<domain> @<target-dns> \
  | grep -v ';\|SOA' \
  | sed -r '/^\s*$/d' \
  | grep $sub \
  | tee -a subdomains.txt
done
```

---

## Wordlists

```
/opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt
/opt/useful/seclists/Discovery/DNS/dns-Jhaddix.txt
```
