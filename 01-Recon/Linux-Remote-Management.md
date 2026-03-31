
# Linux Remote Management Protocols

Three services covered: **SSH** (port 22), **Rsync** (port 873), and **R-Services** (ports 512-514). SSH is ubiquitous. Rsync and R-Services are less common but regularly encountered on internal networks and can expose credentials or allow unauthenticated access via trust misconfigurations.

---

## SSH

Port: `22` TCP

### Footprinting

**Banner reading** -- the banner tells you which protocol versions are accepted:

| Banner | Meaning |
|--------|---------|
| `SSH-1.99-OpenSSH_3.9p1` | Accepts both SSH-1 and SSH-2 |
| `SSH-2.0-OpenSSH_8.2p1` | SSH-2 only |

SSH-1 is vulnerable to MITM attacks. If you see it in use, document it.

**ssh-audit** -- checks supported algorithms, identifies weak ciphers/KEX, and reveals version info:

```bash
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py <target>
```

**Check available auth methods** with verbose flag:

```bash
ssh -v <user>@<target>
# Look for: "Authentications that can continue: publickey,password,keyboard-interactive"
```

**Force password auth** (useful before brute forcing):

```bash
ssh -v <user>@<target> -o PreferredAuthentications=password
```

### Dangerous Settings

| Setting | Risk |
|---------|------|
| `PasswordAuthentication yes` | Enables password brute force |
| `PermitEmptyPasswords yes` | Allows blank passwords |
| `PermitRootLogin yes` | Direct root login possible |
| `Protocol 1` | SSH-1 in use -- MITM vulnerable |
| `AllowTcpForwarding yes` | Can be used for tunneling through the server |
| `PermitTunnel yes` | Full tunnel capability -- useful for pivoting if you get in |
| `X11Forwarding yes` | Historical RCE vector (CVE-2016, OpenSSH 7.2p1) |

### What to Look For

- **`PermitRootLogin yes`** -- direct root SSH = game over. Common on poorly hardened servers.
- **`PasswordAuthentication yes`** -- opens the door for credential spraying and brute force.
- **Weak KEX/ciphers flagged by ssh-audit** -- document as findings. Downgrade attacks and MITM are possible if SSH-1 is in use.
- **CVE-2020-14145** -- MITM on initial connection if client doesn't verify host key. Affects OpenSSH < 8.4.
- **Password reuse** -- SSH is the first service to try any credentials you find elsewhere.

---

## Rsync

Port: `873` TCP

Used for file sync and backups. Can run unauthenticated. If shares are exposed, sensitive files (SSH keys, configs, secrets) are often present.

### Footprinting

```bash
# Confirm service
sudo nmap -sV -p 873 <target>

# Probe for available shares via netcat
nc -nv <target> 873
# After connection, type: #list
```

### Enumerate Shares

```bash
# List contents of a share (no auth)
rsync -av --list-only rsync://<target>/<share>

# Download entire share
rsync -av rsync://<target>/<share> ./local-dir/
```

### With Authentication

```bash
rsync -av --list-only rsync://<user>@<target>/<share>

# If Rsync is tunneled over SSH
rsync -av -e ssh rsync://<user>@<target>/<share>
rsync -av -e "ssh -p 2222" rsync://<user>@<target>/<share>
```

### What to Look For

- **Unauthenticated share listing** -- any response to `#list` without credentials is a finding.
- **`.ssh` directories in shares** -- private keys accessible via Rsync = direct SSH access.
- **`secrets.yaml`, `.env`, config files** -- credentials are frequently stored in backup shares.
- **Password reuse** -- if you have creds from elsewhere, try them against Rsync shares.
- **Write access** -- if you can upload to a web root or cron directory via Rsync, that's RCE.

---

## R-Services

Ports: `512` (rexec), `513` (rlogin), `514` (rsh/rcp) TCP

Legacy Unix remote management suite. Unencrypted -- everything is cleartext over the wire. Still encountered on older Solaris, HP-UX, and AIX systems. The key attack vector is trust misconfiguration via `/etc/hosts.equiv` and `.rhosts`.

### Trust Files

**`/etc/hosts.equiv`** -- global. Any user from listed hosts is trusted without a password.

**`~/.rhosts`** -- per-user. Same concept but scoped to a specific account.

Example `.rhosts` entries:

```
htb-student   10.0.17.5     # user htb-student from this IP is trusted
+             10.0.17.10    # any user from this IP is trusted
+             +             # EVERYONE from EVERYWHERE is trusted -- full unauthenticated access
```

The `+` wildcard means any host or any user. `+ +` in `.rhosts` is a complete authentication bypass.

### Footprinting

```bash
sudo nmap -sV -p 512,513,514 <target>
```

### Exploitation

```bash
# Login without password if .rhosts trusts you
rlogin <target> -l <username>

# Run a command remotely
rsh <target> <command>

# Copy file
rcp <target>:<remote_file> <local_path>
```

### User Enumeration

```bash
# List active sessions on local network (broadcasts on UDP 513)
rwho

# Detailed session info for a specific host
rusers -al <target>
```

Returns: username, hostname, TTY, login time, idle time, and source host.

### What to Look For

- **Ports 512-514 open** -- immediate flag. These services have no business being exposed.
- **`.rhosts` with `+` entries** -- unauthenticated access as that user. Check `~/.rhosts` on any system where you already have shell access.
- **`/etc/hosts.equiv` exists and is permissive** -- grants trusted access to anyone from listed hosts.
- **Cleartext traffic** -- if you have a network position, sniff for credentials since nothing is encrypted.
- **`rwho`/`rusers` responds** -- reveals active usernames and which hosts they're logged into. Useful for building a target user list.
