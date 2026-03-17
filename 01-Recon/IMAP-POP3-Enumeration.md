
# IMAP / POP3

Layer 3 enumeration. IMAP and POP3 are email retrieval protocols -- if you find credentials from any other service, these are the first places to test them. Mailboxes regularly contain credentials, internal documents, and sensitive communications. For protocol definitions and acronyms see [[Mail-Glossary|Mail Glossary]].

---

## Ports

| Port | Protocol | Notes |
|------|----------|-------|
| 143 | IMAP | Plaintext, supports STARTTLS upgrade |
| 993 | IMAPS | IMAP over SSL/TLS |
| 110 | POP3 | Plaintext, supports STARTTLS upgrade |
| 995 | POP3S | POP3 over SSL/TLS |

---

## Workflow

1. Nmap on ports 110, 143, 993, 995 -- grab banners and capabilities
2. Check SSL certificate for hostname, org, and email intel
3. If you have credentials, connect with cURL to list mailboxes
4. Use OpenSSL for encrypted sessions, telnet/netcat for plaintext
5. Browse mailboxes for credentials, internal docs, or sensitive comms

---

## Nmap

```bash
sudo nmap -sV -sC -p110,143,993,995 <target>
```

Default scripts return capability lists and SSL certificate details. The cert `commonName` and `emailAddress` fields often reveal internal hostnames and valid email addresses.

---

## cURL

```bash
# List mailboxes (IMAP)
curl -k 'imaps://<target>' --user <user>:<pass>

# Verbose -- shows TLS version, cert details, full capability exchange
curl -k 'imaps://<target>' --user <user>:<pass> -v

# Fetch emails from INBOX
curl -k 'imaps://<target>/INBOX' --user <user>:<pass>

# Fetch a specific message by ID
curl -k 'imaps://<target>/INBOX;MAILINDEX=1' --user <user>:<pass>
```

---

## OpenSSL -- Encrypted Interaction

```bash
# IMAP over TLS
openssl s_client -connect <target>:993

# POP3 over TLS
openssl s_client -connect <target>:995
```

Once connected, interact with the protocol directly using the command tables below.

---

## IMAP Commands

Used after connecting via OpenSSL or a raw client. Commands are prefixed with a sequence ID (e.g. `A001`).

| Command | Description |
|---------|-------------|
| `A001 LOGIN <user> <pass>` | Authenticate |
| `A001 LIST "" *` | List all mailboxes |
| `A001 SELECT INBOX` | Open a mailbox |
| `A001 FETCH <id> all` | Fetch a message by ID |
| `A001 FETCH <id> body[]` | Fetch full message body |
| `A001 LSUB "" *` | List subscribed mailboxes |
| `A001 CLOSE` | Close selected mailbox |
| `A001 LOGOUT` | End session |

---

## POP3 Commands

| Command | Description |
|---------|-------------|
| `USER <username>` | Identify user |
| `PASS <password>` | Authenticate |
| `STAT` | Message count and total size |
| `LIST` | List all messages with sizes |
| `RETR <id>` | Fetch a message by ID |
| `DELE <id>` | Delete a message |
| `CAPA` | List server capabilities |
| `RSET` | Reset -- undo any deletions this session |
| `QUIT` | End session |

---

## Dangerous Settings

| Setting | Risk |
|---------|------|
| `auth_debug` | Logs all authentication activity |
| `auth_debug_passwords` | Logs submitted passwords in plaintext |
| `auth_verbose` | Logs failed auth attempts with reasons |
| `auth_verbose_passwords` | Logs and may truncate used passwords |
| `auth_anonymous_username` | Enables anonymous login via SASL ANONYMOUS |

If you gain access to `/etc/dovecot/` or similar, these settings in the config are an immediate finding.

---

## What to Look For

- **Valid credentials from other services** -- reuse them here immediately. SMTP usernames, SMB users, OSINT emails. Mailboxes are high value.
- **SSL certificate intel** -- `commonName`, `organizationName`, `emailAddress` in the cert reveal internal hostnames and real email addresses
- **Capability list** -- absence of `STARTTLS` means credentials sent over plaintext ports are unencrypted
- **Anonymous login** -- `auth_anonymous_username` set means potential unauthenticated access
- **Sensitive mailbox content** -- look for password resets, internal tooling credentials, HR communications, anything forwarded from other systems
- **auth_debug_passwords in config** -- passwords are being logged to disk. Locate and read the log file.
- **Port 143/110 open externally** -- plaintext mail retrieval accessible from the internet. Any credentials used are exposed in transit.
