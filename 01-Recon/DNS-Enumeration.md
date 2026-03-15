
# DNS

Layer 3 enumeration. DNS (Domain Name System) translates hostnames to IP addresses across a distributed, hierarchical network of servers. There is no central database -- records are spread across thousands of authoritative name servers worldwide. For recon purposes, DNS leaks infrastructure details, internal hostnames, mail providers, third-party services, and sometimes entire zone files.

> [!tip] Record Types
> For a full breakdown of DNS record types (A, AAAA, MX, NS, TXT, CNAME, PTR, SOA) and email authentication standards (SPF, DKIM, DMARC), see [[_Glossary/DNS|DNS Glossary]].

---

## Port

| Port | Protocol | Purpose |
|------|----------|---------|
| 53 | TCP/UDP | DNS queries (UDP default, TCP for zone transfers and large responses) |

---

## DNS Server Types

| Type | Role |
|------|------|
| Root Server | Top of the hierarchy. 13 globally. Handles TLD resolution when no other server responds. Coordinated by ICANN. |
| Authoritative Nameserver | Holds the actual records for a zone. Answers are binding. |
| Non-authoritative Nameserver | Collects records from other servers via recursive or iterative queries. Not the source of truth. |
| Caching Server | Stores responses for a TTL-defined period to speed up repeat queries. |
| Forwarding Server | Passes queries upstream to another DNS server without resolving them directly. |
| Resolver | Local resolution -- runs on the client machine or router. |

---

## DNS Encryption

DNS is unencrypted by default. Anyone on the local network or the path to the resolver can observe queries. Three main solutions exist:

| Method | Description |
|--------|-------------|
| DNS over TLS (DoT) | Wraps DNS in TLS. Port 853. |
| DNS over HTTPS (DoH) | Tunnels DNS inside HTTPS. Port 443. Harder to block. |
| DNSCrypt | Encrypts and authenticates DNS traffic between client and resolver. |

---

## BIND9 Configuration (Linux)

BIND9 is the most common DNS server on Linux. Understanding its config helps identify what might be misconfigured or exposed.

Config files:
- `/etc/bind/named.conf` -- main config, references the others
- `/etc/bind/named.conf.local` -- zone definitions
- `/etc/bind/named.conf.options` -- global settings
- `/etc/bind/named.conf.log` -- logging config

### Zone Definition (named.conf.local)

```
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};
```

### Zone File (db.domain.com)

Contains forward records -- maps hostnames to IPs:

```
$ORIGIN domain.com
$TTL 86400
@     IN     SOA    dns1.domain.com.  hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh
                    3600       ; retry
                    604800     ; expire
                    86400 )    ; minimum TTL
      IN     NS     ns1.domain.com.
      IN     MX     10  mx.domain.com.
server1      IN     A       10.129.14.5
ns1          IN     A       10.129.14.2
www          IN     CNAME   server2
```

### Reverse Lookup Zone File (db.10.129.14)

Maps IPs back to hostnames via PTR records:

```
$ORIGIN 14.129.10.in-addr.arpa
5    IN     PTR    server1.domain.com.
```

---

## Dangerous Settings

| Option | Risk |
|--------|------|
| `allow-query { any; }` | Any host can query the server, not just internal clients |
| `allow-recursion { any; }` | Server will recursively resolve queries for anyone -- potential for DNS amplification abuse |
| `allow-transfer { any; }` | Any host can request a full zone transfer (AXFR). Critical misconfiguration. |
| `zone-statistics yes` | Leaks query statistics |

---

## Enumeration Workflow

1. Query NS records to identify authoritative nameservers
2. Query the version with a CHAOS TXT request
3. Run `dig any` to pull all available records at once
4. Attempt AXFR zone transfer against identified nameservers
5. If transfer is denied, brute force subdomains with a wordlist
6. Run `dnsenum` for automated enumeration combining all of the above

---

## dig

### NS -- Find Authoritative Nameservers

```bash
dig ns <domain> @<target-dns>
```

Start here. Identifies the authoritative name servers, which may be different from the one you're querying. Test AXFR against each one found.

### Version Query

```bash
dig CH TXT version.bind <target-dns>
```

CHAOS class query. Returns the BIND version if the server is configured to respond. Useful for identifying vulnerable versions.

### ANY -- Pull All Available Records

```bash
dig any <domain> @<target-dns>
```

Returns everything the server is willing to disclose. Not all records are guaranteed -- servers can suppress results for ANY queries, but it's worth trying before more targeted queries.

### SOA

```bash
dig soa <domain> @<target-dns>
```

Admin email is encoded in the SOA record -- the `@` is replaced with `.`. Cross-reference with [[_Glossary/DNS#SOA Record|SOA Record]].

### PTR -- Reverse Lookup

```bash
dig -x <ip> @<target-dns>
```

### AXFR -- Zone Transfer

```bash
# External zone
dig axfr <domain> @<target-dns>

# Internal/sub-zones found from initial enum
dig axfr <subdomain>.<domain> @<target-dns>
```

A successful AXFR dumps the entire zone -- every A record, CNAME, MX, TXT, and NS entry. Against an internal zone this can reveal DCs, mail relays, VPN servers, workstations, and WSUS servers in one request. Always try it. Many servers are misconfigured with `allow-transfer { any; }`.

> [!warning] allow-transfer misconfiguration
> If AXFR succeeds unauthenticated, it is an immediate high severity finding. Internal zones are particularly damaging -- they expose the full internal infrastructure layout.

---

## Subdomain Brute Force

### Bash Loop with dig

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do
  dig $sub.<domain> @<target-dns> \
  | grep -v ';\|SOA' \
  | sed -r '/^\s*$/d' \
  | grep $sub \
  | tee -a subdomains.txt
done
```

Slow but transparent. Good when you want to understand exactly what is and isn't resolving.

### dnsenum

Automated enumeration: NS lookup, MX lookup, AXFR attempt, and subdomain brute force in one run.

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
| `--dnsserver` | Specify the target DNS server |
| `--enum` | Run full enumeration (NS, MX, AXFR attempt, brute force) |
| `-p 0` | No Google scraping (set to 0 to skip) |
| `-s 0` | No Scrap (disable scraping) |
| `-o <file>` | Save results to file |
| `-f <wordlist>` | Wordlist for subdomain brute force |

---

## What to Look For

- **AXFR succeeds** -- full zone dump. Highest value DNS finding. Internal zones reveal DC names, mail servers, VPN endpoints, and internal IPs.
- **Internal subdomains** -- `dc`, `vpn`, `mail`, `internal`, `dev`, `staging`, `wsus`, `backup` in the zone are high-value targets. Cross-reference IPs with [[01-Recon/Host-Discovery|Host Discovery]].
- **SOA admin email** -- decodes to a real email address. Useful for social engineering context and identifying the DNS provider.
- **TXT records** -- third-party verification tokens reveal SaaS stack (Atlassian, Google, Microsoft, Mailgun). SPF `ip4:` entries leak internal mail relay IPs. See [[_Glossary/DNS#TXT Record|TXT Record]].
- **NS records pointing inward** -- if an NS record resolves to an internal IP, you may be able to query it directly for more detail.
- **Version disclosure** -- BIND version from CHAOS query. Cross-reference with known CVEs.
- **allow-transfer { any; }** -- if set, AXFR works. Flag as critical and document the full zone dump.
- **Subdomain bruteforce hits** -- resolve each found subdomain and add to scope. Look for `dev`, `test`, `staging`, `admin`, `api` subdomains with looser security posture.
