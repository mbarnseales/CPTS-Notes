
# WHOIS

Passive recon. Query public registration databases to retrieve ownership and infrastructure details for a domain. No interaction with the target -- queries go to registrar databases.

See Also: [[00-Methedology/Web-Recon|Web Recon Methodology]]

---

## Usage

```bash
whois <domain>

# Example
whois inlanefreight.com
```

No flags needed for standard use. Output varies by registrar.

---

## What to Pull From the Output

| Field | Why It Matters |
|-------|---------------|
| **Registrant / Admin / Tech Contact** | Names, emails, phone numbers -- targets for phishing and social engineering |
| **Registrar** | Where the domain is managed -- useful context |
| **Creation Date** | Old domain = established org. Very new domain = possible phishing/staging infrastructure |
| **Expiration Date** | Domains expiring soon may be candidates for hijacking |
| **Name Servers** | Reveals DNS provider -- sometimes internal or self-hosted, which is useful for DNS recon |
| **Registrar WHOIS Server** | Where to query for more detail if the initial response is thin |

---

## Privacy and Redacted Records

Many registrants use **WHOIS privacy protection** (e.g. Domains By Proxy). When enabled, contact fields show the proxy service's details instead of the real registrant. This is increasingly common post-GDPR.

If records are redacted:
- Try historical WHOIS -- privacy wasn't always enabled
- Cross-reference the name servers and registrar for infrastructure clues
- Move on to other passive sources (crt.sh, Shodan, DNS)

---

## Historical WHOIS

Past records can reveal previous owners, old contact details, and infrastructure changes:

- **WhoisFreaks** -- `https://whoisfreaks.com`
- **WhoisXML API** -- `https://whois.whoisxmlapi.com`
- **DomainTools** -- `https://whois.domaintools.com`

Useful for tracking when a domain changed hands, finding older contact info before privacy was enabled, or confirming a domain was previously used for something else.

---

## Domain Status Codes

Appear in WHOIS output under `Domain Status`. Multiple statuses are common on large/secure domains.

| Status | Meaning |
|--------|---------|
| `clientDeleteProhibited` | Registrar won't delete the domain unless the client requests removal of this lock |
| `clientTransferProhibited` | Domain cannot be transferred to another registrar |
| `clientUpdateProhibited` | Domain record cannot be modified |
| `serverDeleteProhibited` | Registry-level delete lock -- stronger than client-side |
| `serverTransferProhibited` | Registry-level transfer lock |
| `serverUpdateProhibited` | Registry-level update lock |

Large orgs like Meta typically have all six set. A domain with no status locks is less protected and worth noting.

---

## What to Look For

- **Real contact details not hidden by privacy** -- direct pivot to social engineering or phishing targets
- **Recently registered domain** -- days-old registration on a domain claiming to be a bank or known brand = phishing infrastructure
- **Self-hosted name servers** -- e.g. `A.NS.TARGET.COM` means the org manages their own DNS. Large orgs only. Worth probing further
- **Shared name servers across domains** -- multiple target domains pointing to the same NS can indicate shared infrastructure or related assets
- **No status locks set** -- domain is less protected, lower barrier for hijacking or social engineering the registrar
- **Mismatched registrant info** -- different contacts across domains in the same org can reveal subsidiaries or acquisitions
- **Expiring domains** -- potential domain hijacking opportunity if in scope
