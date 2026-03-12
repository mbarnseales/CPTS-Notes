
# FTP

Layer 3 enumeration. FTP (File Transfer Protocol) runs on TCP -- two channels, control on port 21 and data on port 20. TFTP is a lighter, UDP-based variant with no authentication. Both are worth probing: FTP for credentials and file access, TFTP where it's present on internal networks.

> [!warning] Cleartext Protocol
> FTP transmits credentials and data in plaintext. If you have a position on the network, [[tcpdump]] will capture credentials. See [[Credential-Hunting]].

---

## How FTP Works

Two channels are established per session:

| Channel | Port | Purpose |
|---------|------|---------|
| Control | 21 (TCP) | Commands and status codes |
| Data | 20 (TCP) | File transfers |

### Active vs Passive Mode

| Mode | Who Opens Data Channel | Problem |
|------|----------------------|---------|
| Active | Server connects back to client | Blocked by client-side firewalls |
| Passive | Client connects to a server-announced port | Firewall-friendly -- client initiates both connections |

When connecting through a firewall or NAT, passive mode is required. Most modern FTP clients default to passive.

---

## TFTP

UDP-based, no authentication, no directory listing. Access is controlled only by filesystem read/write permissions on the server. Limited to globally readable/writable directories. Only used on local, trusted networks.

| Command | Description |
|---------|-------------|
| `connect` | Set remote host and optional port |
| `get` | Download file(s) from remote |
| `put` | Upload file(s) to remote |
| `status` | Show current mode, connection status, timeout |
| `verbose` | Toggle verbose output during transfers |
| `quit` | Exit |

---

## Enumeration Workflow

1. Nmap service scan with default scripts
2. Check for anonymous login
3. List directory contents -- use recursive listing if available
4. Note file ownership and permissions
5. Attempt download of interesting files
6. Attempt upload to check write access
7. If TLS is active, inspect certificate for hostname and org intel

---

## Nmap

```bash
# Service version + default scripts + aggressive scan
sudo nmap -sV -p21 -sC -A <target>

# Trace NSE script activity at network level
sudo nmap -sV -p21 -sC -A <target> --script-trace

# Update NSE script database
sudo nmap --script-updatedb

# Find all FTP-related NSE scripts
find / -type f -name ftp* 2>/dev/null | grep scripts
```

Key NSE scripts that run automatically with `-sC`:

| Script | What It Does |
|--------|-------------|
| `ftp-anon` | Tests for anonymous login, lists root directory if allowed |
| `ftp-syst` | Runs `STAT` command -- shows server version and config |
| `ftp-brute` | Brute-force credentials (not run by default) |
| `ftp-vsftpd-backdoor` | Tests for the vsFTPd 2.3.4 backdoor CVE |
| `ftp-bounce` | Tests for FTP bounce attack |

The banner shown at connection (`220`) often includes the service name and version.

---

## Manual Interaction

```bash
# Standard FTP client
ftp <target>

# Anonymous login
Name: anonymous
Password: (blank or any email-format string)

# Netcat (banner grab only)
nc -nv <target> 21

# Telnet
telnet <target> 21

# TLS-encrypted FTP -- also reveals SSL certificate
openssl s_client -connect <target>:21 -starttls ftp
```

The SSL certificate from openssl will show the CN (hostname), org, and sometimes email. The same intel as from a browser cert view -- cross-reference with [[OSINT-Online-Presence#1. SSL Certificate|OSINT: SSL Certificate]].

---

## FTP Client Commands

Once connected:

| Command | Description |
|---------|-------------|
| `ls` | List current directory |
| `ls -R` | Recursive listing (requires `ls_recurse_enable=YES` on server) |
| `get <file>` | Download a file |
| `put <file>` | Upload a file |
| `status` | Show session settings (mode, type, verbose, etc.) |
| `debug` | Enable debug output |
| `trace` | Enable packet-level trace output |
| `bye` / `exit` | Disconnect |

---

## Bulk Download

```bash
# Download entire FTP site recursively (active mode -- bypasses passive requirement)
wget -m --no-passive ftp://anonymous:anonymous@<target>

# Output is saved to a directory named after the target IP
tree ./<target-ip>
```

> [!warning]
> Downloading everything at once is noisy. No legitimate user does a bulk recursive pull. Only do this when stealth is not a concern.

---

## vsFTPd Configuration

Config file: `/etc/vsftpd.conf`
Blocked users file: `/etc/ftpusers` (users listed here are denied FTP access even if valid system users)

### Key Default Settings

| Setting | Description |
|---------|-------------|
| `listen=NO` | Run from inetd or as standalone daemon |
| `anonymous_enable=NO` | Anonymous access disabled by default |
| `local_enable=YES` | Local system users can log in |
| `xferlog_enable=YES` | Log all uploads and downloads |
| `connect_from_port_20=YES` | Use port 20 for data channel |
| `ssl_enable=NO` | TLS disabled by default |
| `hide_ids=YES` | Replace UID/GID in directory listings with "ftp" -- makes ownership enumeration harder |
| `ls_recurse_enable=YES` | Allow recursive directory listings |

### Dangerous / Anonymous Settings

| Setting | Description |
|---------|-------------|
| `anonymous_enable=YES` | Allow unauthenticated access |
| `anon_upload_enable=YES` | Allow anonymous file upload |
| `anon_mkdir_write_enable=YES` | Allow anonymous directory creation |
| `no_anon_password=YES` | Skip password prompt for anonymous users |
| `anon_root=/home/username/ftp` | Anonymous user root directory |
| `write_enable=YES` | Allow write commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, SITE |

---

## What to Look For

- **Anonymous login** -- automatic access without credentials. Even read-only access can leak internal documents, calendars, client data.
- **Write access** -- if upload is allowed and the FTP root is served by a web server, upload a webshell for RCE. Also check FTP log poisoning paths.
- **File ownership** -- if `hide_ids=NO`, directory listings show real UIDs. These can confirm valid usernames for further attacks (SSH brute-force, etc.).
- **TLS certificate** -- reveals hostname, org, and sometimes email. Treat the same as any other cert find.
- **File contents** -- config files, credentials, internal documents, keys. Pull and inspect anything non-obvious.
