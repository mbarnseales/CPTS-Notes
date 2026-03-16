
# DNS

Layer 3 enumeration. DNS leaks infrastructure details, internal hostnames, mail providers, third-party services, and sometimes entire zone files if misconfigured.

For record types, server types, and email authentication standards see [[DNS-Glossary|DNS Glossary]].

Port: `53` TCP/UDP - UDP by default, TCP for zone transfers and large responses.

---

## Workflow

1. Query NS records to identify authoritative nameservers
2. Pull the BIND version with a CHAOS query
3. Run `dig any` to pull all records at once
4. Attempt AXFR zone transfer against each nameserver found
5. If transfer is denied, brute force subdomains
6. Run `dnsenum` to automate all of the above

---

## dig

```bash
# Nameservers - start here, test AXFR against each result
dig ns <domain> @<target-dns>

# All available records
dig any <domain> @<target-dns>

# SOA -- admin email is encoded here (@  replaced with .)
dig soa <domain> @<target-dns>

# Reverse lookup
dig -x <ip> @<target-dns>

# BIND version
dig CH TXT version.bind <target-dns>

# Zone transfer -- try against every NS found
dig axfr <domain> @<target-dns>
dig axfr <subdomain>.<domain> @<target-dns>
```

> [!warning] AXFR
> If zone transfer succeeds unauthenticated it is a critical finding. Internal zones can expose DC names, VPN endpoints, mail relays, workstations, and WSUS servers in a single request.

---

## Subdomain Brute Force

```bash
# Bash loop
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do
  dig $sub.<domain> @<target-dns> \
  | grep -v ';\|SOA' \
  | sed -r '/^\s*$/d' \
  | grep $sub \
  | tee -a subdomains.txt
done
```

---

## dnsenum

NS, MX, AXFR attempt, and subdomain brute force in one run:

```bash
dnsenum --dnsserver <target-dns> \
        --enum \
        -p 0 -s 0 \
        -o subdomains.txt \
        -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
        <domain>
```

See [[_Tools/DNS|DNS Tools]] for flag reference.

---

## What to Look For

- **AXFR succeeds** -- full zone dump. Internal zones reveal DC names, mail servers, VPN endpoints, internal IPs
- **Internal subdomains** -- `dc`, `vpn`, `mail`, `internal`, `dev`, `staging`, `wsus`, `backup` are high-value. Cross-reference IPs with [[01-Recon/Host-Discovery|Host Discovery]]
- **Version disclosure** -- BIND version from CHAOS query, check against known CVEs
- **SOA admin email** -- real contact address, useful for social engineering scope
- **TXT records** -- SaaS stack intel from verification tokens, internal IPs from SPF `ip4:` entries. See [[DNS-Glossary#TXT Record|TXT Record]]
- **NS resolving to internal IP** -- query it directly for additional records
- **Subdomain brute force hits** -- `dev`, `test`, `staging`, `admin`, `api` tend to have looser security posture
