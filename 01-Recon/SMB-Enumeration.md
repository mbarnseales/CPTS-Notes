
# SMB

Layer 3 enumeration. SMB (Server Message Block) is a client-server protocol for sharing files, printers, and other network resources over TCP. Samba is the open-source Linux/Unix implementation, enabling cross-platform SMB communication. Misconfigurations are extremely common and often allow unauthenticated enumeration of users, shares, and domain information.

> [!warning] Null Sessions
> SMB frequently allows anonymous access even without explicit misconfiguration. Always probe for null sessions before touching credentials. What you find unauthenticated often gives you usernames for the next phase.

---

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 445 | TCP | SMB direct over TCP (modern) |
| 139 | TCP | SMB over NetBIOS session service |
| 137-138 | UDP | NetBIOS name and datagram service |

---

## SMB Versions

| Version | Platform | Notes |
|---------|----------|-------|
| CIFS (SMB 1) | Windows NT 4.0 | Legacy. Communication via NetBIOS interface |
| SMB 1.0 | Windows 2000 | Direct connection via TCP |
| SMB 2.0 | Windows Vista / Server 2008 | Performance upgrades, improved signing, caching |
| SMB 2.1 | Windows 7 / Server 2008 R2 | Locking mechanisms |
| SMB 3.0 | Windows 8 / Server 2012 | Multichannel, end-to-end encryption, remote storage |
| SMB 3.0.2 | Windows 8.1 / Server 2012 R2 | Incremental improvements |
| SMB 3.1.1 | Windows 10 / Server 2016 | Integrity checking, AES-128 encryption |

> [!warning] SMB 1 / CIFS
> SMB 1 is legacy and considered dangerous. If it's running, assume the host is unpatched or poorly maintained. It is the version affected by EternalBlue.

---

## Samba

Samba is the Linux/Unix SMB implementation. Key points:

- Implements CIFS (SMB 1 dialect) but supports newer SMB versions
- Legacy NetBIOS connections use TCP ports 137/138/139, modern CIFS uses port 445
- Samba 3 can join an AD domain as a member server
- Samba 4 can act as a full AD domain controller
- Two core daemons: `smbd` (file and print sharing) and `nmbd` (NetBIOS name service)

---

## Samba Configuration

Config file: `/etc/samba/smb.conf`

Strip comments to see only active settings:
```bash
cat /etc/samba/smb.conf | grep -v "#\|;"
```

### Key Settings

| Setting | Description |
|---------|-------------|
| `workgroup` | Workgroup/domain name shown to querying clients |
| `path` | Filesystem path being shared |
| `server string` | Banner string shown on connection initiation |
| `unix password sync` | Synchronise UNIX password with SMB password |
| `usershare allow guests` | Allow unauthenticated access to defined shares |
| `map to guest = bad user` | Map unknown user logins to the guest account |
| `browseable` | Show this share in the list of available shares |
| `guest ok` | Allow connections without a password |
| `read only` | Prevent write operations |
| `create mask` | Permissions applied to newly created files |

### Dangerous Settings

| Setting | Risk |
|---------|------|
| `browseable = yes` | Exposes share list to unauthenticated users |
| `read only = no` / `writable = yes` | Allows file creation and modification |
| `guest ok = yes` | Passwordless access to the share |
| `enable privileges = yes` | Honours privilege assignments based on SID |
| `create mask = 0777` | New files are world-readable and writable |
| `directory mask = 0777` | New directories are world-accessible |
| `logon script = script.sh` | Executes a script at user logon |
| `magic script = script.sh` | Executes when the script file is closed |
| `magic output = script.out` | Output destination for magic script |

---

## Enumeration Workflow

1. Nmap on ports 139 and 445 with default scripts
2. Probe for null session access with `smbclient -N -L`
3. Browse accessible shares, pull interesting files
4. Use `rpcclient` for user, group, and domain enumeration
5. Brute force RID range if `enumdomusers` is restricted
6. Run `enum4linux-ng -A` for a consolidated automated sweep
7. Cross-reference findings with [[_Tools/SMB|SMB Tools Reference]]

---

## Nmap

```bash
sudo nmap -sV -sC -p139,445 <target>
```

Default scripts on SMB return OS information, SMB version, signing status, and NetBIOS names. Output is often limited compared to manual tools. Use this as a starting point, not a complete picture.

---

## smbclient

```bash
# List shares -- null session (no credentials)
smbclient -N -L //<target>

# Connect to a share -- press Enter for anonymous login
smbclient //<target>/<share>

# Connect with credentials
smbclient -U <user> //<target>/<share>
```

Once connected:

```bash
ls                    # list directory
cd <dir>              # change directory
get <file>            # download a file
put <file>            # upload a file
!ls                   # run a local shell command without disconnecting
!cat <file>           # read a local file (e.g. one just downloaded)
```

> [!tip]
> `help` inside smbclient lists every available command. Use `!<cmd>` to run local shell commands mid-session without dropping the connection.

---

## smbstatus

Run on the Samba server itself to view active connections:

```bash
sudo smbstatus
```

Shows Samba version, connected users, source IPs, shares in use, and locked files. Useful on authorised internal tests to confirm your session is established and see who else is connected.

---

## rpcclient

MS-RPC interface for Samba/SMB. Frequently allows unauthenticated enumeration of users, groups, shares, and domain info. See [[_Tools/SMB|SMB Tools Reference]] for the full query table.

```bash
# Null session
rpcclient -U "" <target>
# Press Enter at the password prompt

# Explicit no-password flag
rpcclient -N -U "" <target>
```

### Core Queries

| Command | Description |
|---------|-------------|
| `srvinfo` | Server OS, platform ID, and server type |
| `enumdomains` | List all deployed domains |
| `querydominfo` | Domain name, server name, total users/groups |
| `netshareenumall` | List all shares with paths |
| `netsharegetinfo <share>` | Detailed share info including ACL entries |
| `enumdomusers` | List domain users with RIDs |
| `queryuser <RID>` | Full user record (home drive, logon times, account flags) |
| `querygroup <RID>` | Group name, description, and member count |

### RID Brute Force

When `enumdomusers` is restricted or returns incomplete results, brute force the RID range directly:

```bash
for i in $(seq 500 1100); do
  rpcclient -N -U "" <target> -c "queryuser 0x$(printf '%x\n' $i)" \
  | grep "User Name\|user_rid\|group_rid" && echo ""
done
```

RIDs below 500 are reserved. Common user accounts start at 1000 (0x3e8). Scan to at least 1100 to cover typical ranges.

---

## Impacket - samrdump.py

Alternative to rpcclient for SAMR-based user enumeration. Cleaner output, less manual:

```bash
samrdump.py <target>
```

Returns usernames, UIDs, password last set, expiry status, and logon count. Useful as a quick cross-check against rpcclient output.

---

## SMBmap

Share enumeration with per-share permission visibility:

```bash
# Null session
smbmap -H <target>

# Authenticated
smbmap -H <target> -u <user> -p <password>

# Recursive listing of a specific share
smbmap -H <target> -r <share>

# Download a file directly
smbmap -H <target> --download '<share>\<file>'
```

Clearly shows READ, WRITE, or NO ACCESS per share. Faster for permission triage than smbclient browsing.

---

## CrackMapExec

Broader SMB enumeration with credential spraying capability:

```bash
# Null session share enumeration
crackmapexec smb <target> --shares -u '' -p ''

# Authenticated share enum
crackmapexec smb <target> --shares -u <user> -p <password>

# Enumerate logged-on users (requires auth)
crackmapexec smb <target> --loggedon-users -u <user> -p <password>

# Dump SAM hashes (requires admin-level access)
crackmapexec smb <target> --sam -u <user> -p <password>
```

---

## enum4linux-ng

Automated SMB/RPC enumeration. Combines share listing, user enumeration, OS detection, and password policy into a single sweep.

```bash
# Install
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt

# Full enumeration
./enum4linux-ng.py <target> -A
```

Good as a first-pass sweep. Always manually verify anything interesting it returns -- automated tools miss context and occasionally misparse output.

---

## What to Look For

- **Null session allowed** -- enumerate everything before using credentials. Shares, users, domain info, password policy.
- **Browseable shares with sensitive names** -- `dev`, `notes`, `backup`, `home`, `it`, `scripts`. Browse them all.
- **guest ok = yes with writable shares** -- potential file upload path. If the share is also served by a web server, this is an RCE vector via webshell upload.
- **Users and RIDs from rpcclient** -- valid usernames for password spraying. Cross-reference with [[_Reference/File-Transfers|File Transfers]] if you need to exfiltrate a wordlist.
- **Password policy** -- check minimum length and lockout threshold via `querydominfo` or enum4linux-ng. No lockout threshold means safe to spray aggressively.
- **Domain membership** -- Samba in an AD domain opens AD-specific attack paths. See [[06-AD/]].
- **ACLs on shares** -- SID `S-1-1-0` means Everyone. Full access for Everyone on a writable share is a critical finding.
- **magic script / logon script settings** -- if the script path points to a writable share, you can replace it for code execution on the next logon.
- **SMB signing not required** -- a prerequisite for NTLM relay attacks. Flag it and note the host.
- **SMB 1 running** -- check for EternalBlue (MS17-010) with `nmap --script smb-vuln-ms17-010 -p445 <target>`.
