
# Mail Protocols

Acronyms and concepts covering the full mail stack -- sending, routing, and retrieval. For enumeration commands and workflow see [[SMTP-Enumeration|SMTP Enumeration]] and [[IMAP-POP3-Enumeration|IMAP / POP3 Enumeration]].

---

# The Mail Chain

These four components describe the path an email takes from sender to recipient. Worth knowing because misconfigurations in any of them can be exploitable.

## MUA
**Mail User Agent.** The email client -- Outlook, Thunderbird, Gmail in a browser. Composes the email and hands it off to the MSA for delivery.

## MSA
**Mail Submission Agent.** Receives email from the MUA and validates it before passing it to the MTA. Also called a relay server. Typically runs on port 587 with STARTTLS. This is the component vulnerable to open relay misconfiguration.

## MTA
**Mail Transfer Agent.** The core of the mail server -- handles sending, receiving, and routing email between servers. Responsible for looking up the destination MX record in DNS and delivering to the next hop. Postfix and Sendmail are common MTAs.

## MDA
**Mail Delivery Agent.** Receives email from the MTA and places it into the recipient's mailbox. The final step before POP3 or IMAP retrieves it for the MUA.

```
MUA  ->  MSA  ->  MTA  ->  MTA (remote)  ->  MDA  ->  Mailbox
                                                          |
                                                    IMAP / POP3
                                                          |
                                                         MUA
```

---

# Sending Protocols

## SMTP
**Simple Mail Transfer Protocol.** The protocol for sending email between servers and from clients to servers. Runs on port 25 (server-to-server) and 587 (client submission). Unencrypted by default -- transmits commands and credentials in plaintext unless STARTTLS is negotiated.

## ESMTP
**Extended SMTP.** The modern version of SMTP with support for extensions like AUTH, STARTTLS, SIZE, and PIPELINING. When you send `EHLO` instead of `HELO`, the server responds with a list of supported ESMTP extensions. In practice, SMTP and ESMTP are used interchangeably -- most servers run ESMTP. See [[SMTP-Enumeration#Telnet / Manual Interaction|EHLO usage]].

## STARTTLS
A command that upgrades an existing plaintext connection to an encrypted TLS connection mid-session. Used on ports 143 (IMAP) and 587 (SMTP). The sequence is: connect in plaintext, server advertises `STARTTLS` in its capability list, client sends `STARTTLS`, TLS handshake occurs. If a server does not advertise STARTTLS, credentials are sent in plaintext. See [[SMTP-Enumeration#What to Look For|SMTP: What to Look For]] and [[IMAP-POP3-Enumeration#What to Look For|IMAP/POP3: What to Look For]].

---

# Retrieval Protocols

## IMAP
**Internet Message Access Protocol.** Protocol for reading and managing email stored on a mail server. Runs on port 143 (plaintext / STARTTLS) and 993 (SSL/TLS). Key characteristic: email stays on the server. Changes sync across all connected clients. Supports folder structures, flags, and server-side search. Dovecot is the most common IMAP server on Linux. See [[IMAP-POP3-Enumeration|IMAP / POP3 Enumeration]].

## POP3
**Post Office Protocol version 3.** Older, simpler protocol for retrieving email. Runs on port 110 (plaintext / STARTTLS) and 995 (SSL/TLS). Key characteristic: email is downloaded to the client and typically deleted from the server. No folder structures, no sync across clients. Still commonly found, especially on legacy systems or embedded mail servers. See [[IMAP-POP3-Enumeration|IMAP / POP3 Enumeration]].

### IMAP vs POP3

| | IMAP | POP3 |
|--|------|------|
| Email location | Stays on server | Downloaded, usually deleted |
| Multi-client sync | Yes | No |
| Folder support | Yes | No |
| Ports | 143 / 993 | 110 / 995 |
| Relevance to attacker | Higher -- full mailbox accessible on server | Lower -- only unread/undownloaded mail available |

---

# Authentication and Anti-Spam

## SPF
**Sender Policy Framework.** A DNS TXT record that specifies which IP addresses are authorised to send email on behalf of a domain. Receiving servers check the SPF record to verify the sender's IP is listed. If not, the email may be rejected or marked as spam. SPF `ip4:` entries in DNS TXT records can leak internal mail relay IPs -- useful during DNS enumeration. See [[DNS-Enumeration#What to Look For|DNS: TXT Records]].

## DKIM
**DomainKeys Identified Mail.** Adds a cryptographic signature to outgoing emails using a private key held by the sending server. The corresponding public key is published as a DNS TXT record. Receiving servers verify the signature to confirm the email was not tampered with in transit and genuinely originated from the claimed domain.

## DMARC
**Domain-based Message Authentication, Reporting and Conformance.** A policy layer built on top of SPF and DKIM. Tells receiving servers what to do if an email fails SPF or DKIM checks -- options are `none` (monitor only), `quarantine` (spam folder), or `reject` (block outright). Also specifies a reporting address where failure reports are sent. Absence of a DMARC record means the domain has no enforcement policy and is more susceptible to spoofing.

---

# Open Relay

An SMTP server that forwards email for any sender to any recipient without authentication. Historically common, now a misconfiguration. Caused by `mynetworks = 0.0.0.0/0` in Postfix or equivalent in other MTAs. An open relay allows anyone to send spoofed email through the server, making it useful for phishing and bypassing spam filters that trust the relay's reputation. Critical finding on any engagement. See [[SMTP-Enumeration#What to Look For|SMTP: What to Look For]].
