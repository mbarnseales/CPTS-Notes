# SMTP Enumeration

![[Pasted image 20260317164220.png]]

**SMTP Error Codes:** [TurboSMTP](https://serversmtp.com/smtp-error/)

For protocol acronyms (MUA, MTA, MDA, ESMTP, SPF, DKIM, DMARC, Open Relay) see [[Mail-Glossary|Mail Glossary]].

---

## Ports

| Port | Purpose |
|------|---------|
| 25 | Default SMTP. Server-to-server. |
| 587 | Submission port. Authenticated clients, uses STARTTLS. |
| 465 | SMTP over SSL/TLS (legacy but still seen). |

---

## Workflow

1. Nmap on port 25 with default scripts -- grab banner and supported commands
2. Connect via telnet, send `EHLO` to enumerate supported extensions
3. Check if `VRFY` is enabled -- use it to enumerate valid users
4. Check `EXPN` for mailing list expansion
5. Test for open relay with `smtp-open-relay` NSE script
6. Check `mynetworks` config if you gain access to the server

---

## Nmap

```bash
# Service scan + default scripts (runs smtp-commands via EHLO)
sudo nmap -sC -sV -p25 <target>

# Open relay check (runs 16 relay tests)
sudo nmap -p25 --script smtp-open-relay -v <target>
```

---

## Telnet / Manual Interaction

```bash
telnet <target> 25
```

```
# Initiate session -- EHLO returns supported extensions, HELO does not
EHLO <hostname>

# Enumerate valid users (unreliable -- server may return 252 for any input)
VRFY <username>

# Expand a mailing list alias
EXPN <alias>

# Keep connection alive
NOOP
```

> [!warning] VRFY Unreliable
> Many servers return `252` for any username, valid or not. Confirm findings manually before using results downstream.

---

## Send an Email via Telnet

Useful for testing open relays or phishing simulation on an authorised engagement:

```
EHLO <domain>
MAIL FROM: <sender@domain>
RCPT TO: <recipient@domain> NOTIFY=success,failure
DATA
From: <sender@domain>
To: <recipient@domain>
Subject: <subject>

<body>
.
QUIT
```

---

## SMTP Commands Reference

| Command | Description |
|---------|-------------|
| `HELO` | Identify client to server, starts session (basic) |
| `EHLO` | Same as HELO but requests extended feature list |
| `AUTH PLAIN` | Authenticate the client |
| `MAIL FROM` | Declare the sender address |
| `RCPT TO` | Declare the recipient address |
| `DATA` | Begin email body input, end with a single `.` on a new line |
| `VRFY` | Check if a mailbox exists |
| `EXPN` | Expand a mailing list alias to member addresses |
| `RSET` | Abort current transaction, keep connection open |
| `NOOP` | No-op ping to prevent timeout |
| `QUIT` | End the session |

---

## Dangerous Settings

| Setting | Risk |
|---------|------|
| `mynetworks = 0.0.0.0/0` | Open relay -- server will forward mail for any source. Enables spam and spoofing. |
| `VRFY` enabled | Username enumeration |
| `EXPN` enabled | Mailing list member enumeration |

---

## What to Look For

- **Open relay** -- `smtp-open-relay` returns 16/16. Critical finding. Allows unauthenticated mail spoofing from any source.
- **VRFY enabled** -- probe with known usernames found from other services (SMB, NFS, OSINT). Even unreliable responses narrow the field.
- **EXPN enabled** -- can expose internal distribution list members and reveal usernames
- **Banner disclosure** -- server banner often includes software name and version (`220 ESMTP Postfix`). Cross-reference with CVEs.
- **EHLO extension list** -- look for `AUTH`, `STARTTLS`, `VRFY`, `EXPN`. Absence of `STARTTLS` means plaintext auth is in use.
- **Port 25 open externally** -- should only be used for server-to-server. If reachable from the internet with no auth required, test for open relay.
- **mynetworks misconfiguration** -- if you have access to `/etc/postfix/main.cf`, `0.0.0.0/0` is an immediate critical.
