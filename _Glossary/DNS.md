
# DNS

DNS (Domain Name System) concepts, record types, and email authentication standards referenced throughout recon, enumeration, and exploitation.

---

# Record Types

## A Record

Maps a hostname to an IPv4 address. The most fundamental DNS record -- tells you what IP a domain or subdomain actually resolves to.

```
inlanefreight.com.  300  IN  A  10.129.27.33
```

Multiple A records for the same domain indicate load balancing or multiple hosting locations.

## AAAA Record

Same as A but for IPv6 addresses.

## MX Record

Mail Exchange. Points to the mail server(s) responsible for receiving email for the domain. Priority value (lower = higher priority) determines delivery order when multiple records exist.

```
inlanefreight.com.  3600  IN  MX  1   aspmx.l.google.com.
inlanefreight.com.  3600  IN  MX  5   alt1.aspmx.l.google.com.
```

If MX records point to Google, Microsoft, or another major provider, email is hosted externally. Note the provider -- skip direct testing unless in scope.

## NS Record

Name Server. Specifies which DNS servers are authoritative for the domain. Often reveals the hosting or registrar.

```
inlanefreight.com.  21600  IN  NS  ns.inwx.net.
```

## CNAME Record

Canonical Name. An alias that points one hostname to another hostname (not an IP). Common for `www` pointing to an apex domain, or for CDN/load balancer hostnames.

```
shop.inlanefreight.com.  300  IN  CNAME  inlanefreight.com.
```

## TXT Record

Arbitrary text attached to a domain. Used for domain ownership verification, email authentication, and service validation. See [[OSINT-Online-Presence#6. Reading TXT Records for Intel|Reading TXT Records for Intel]] for how to extract third-party intel from these.

Common TXT record contents:
- SPF policy
- DKIM public key
- DMARC policy
- Third-party domain verification tokens (Google, Atlassian, LogMeIn, etc.)
- Internal IP ranges via SPF `ip4:` entries

## SOA Record

Start of Authority. Contains administrative information about the zone -- primary name server, email of the responsible admin (encoded as a hostname), and timing values.

```
inlanefreight.com.  21600  IN  SOA  ns.inwx.net. hostmaster.inwx.net. 2021072600 ...
```

The second field (`hostmaster.inwx.net.`) is the admin contact email with the `@` replaced by `.` -- so this is `hostmaster@inwx.net`. Often reveals the registrar or DNS management provider.

## PTR Record

Pointer. Reverse DNS -- maps an IP back to a hostname. Used for reverse lookups.

```bash
dig -x <IP>
```

Useful for confirming ownership of an IP and discovering additional hostnames on the same address.

---

# Email Authentication Standards

## SPF -- Sender Policy Framework

A TXT record that lists which mail servers are authorised to send email on behalf of the domain. Receiving servers check it to detect spoofing.

```
"v=spf1 include:mailgun.org include:_spf.google.com ip4:10.129.24.8 ~all"
```

Key modifiers:
| Symbol | Meaning |
|--------|---------|
| `+all` | Allow all (effectively no restriction -- dangerous) |
| `~all` | Soft fail -- accept but mark as suspicious |
| `-all` | Hard fail -- reject if not in list |
| `include:` | Include another domain's SPF policy |
| `ip4:` | Specific authorised IPv4 address |

> [!tip] Intel Value
> The `ip4:` entries often contain internal mail relay addresses -- note these as potential internal IPs even before foothold.

## DKIM -- DomainKeys Identified Mail

A cryptographic signature added to outgoing emails. The public key is published as a TXT record. Receiving servers use it to verify the email was genuinely sent by the domain and not tampered with in transit.

```
selector._domainkey.inlanefreight.com.  IN  TXT  "v=DKIM1; k=rsa; p=<public-key>"
```

DKIM alone doesn't prevent spoofing -- it only verifies the signing domain, which may differ from the From: address.

## DMARC -- Domain-based Message Authentication, Reporting and Conformance

Builds on SPF and DKIM. Specifies what to do when an email fails both checks and where to send reports. Published as a TXT record at `_dmarc.<domain>`.

```
_dmarc.inlanefreight.com.  IN  TXT  "v=DMARC1; p=reject; rua=mailto:dmarc@inlanefreight.com"
```

Policy options:
| Value | Action |
|-------|--------|
| `p=none` | Monitor only -- no enforcement. Easy to spoof. |
| `p=quarantine` | Send to spam |
| `p=reject` | Block delivery entirely |

A `p=none` DMARC policy means the domain may be spoofable in phishing scenarios.

---

# Certificate Transparency

A public logging standard (RFC 6962) requiring Certificate Authorities to record every SSL/TLS certificate they issue in append-only, publicly auditable logs. The intent is to detect misissued or fraudulent certificates.

**For recon:** every certificate issued for a domain -- including wildcard certs, subdomain certs, and expired ones -- is permanently logged and searchable. This makes crt.sh a reliable passive subdomain discovery tool even without touching the target.

```bash
https://crt.sh/?q=<domain>
curl -s "https://crt.sh/?q=<domain>&output=json" | jq .
```

Entries include the common name, SANs, issuer CA, and issue/expiry dates. Expired certificates still reveal historical subdomains that may have been reactivated or left running.

---

# FQDN -- Fully Qualified Domain Name

The complete, unambiguous domain name specifying the exact location in the DNS hierarchy, ending with a trailing dot (`.`) representing the root zone.

```
mail.inlanefreight.com.
```

In practice the trailing dot is usually omitted but is implied. An FQDN includes all labels from the hostname through to the root: `hostname.domain.tld.`
