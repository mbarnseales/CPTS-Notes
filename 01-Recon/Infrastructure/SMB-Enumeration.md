
# SMB

Layer 3 enumeration. SMB (Server Message Block) is a client-server protocol for sharing files, printers, and other network resources over TCP. Samba is the open-source Linux/Unix implementation. Misconfigurations are extremely common and often allow unauthenticated enumeration of users, shares, and domain information.

> [!warning] Null Sessions
> Always probe for null/anonymous access before touching credentials. What you find unauthenticated often gives you usernames for the next phase.

Ports: `445` TCP (modern), `139` TCP (legacy NetBIOS)

---

## Workflow

1. Nmap on 139 and 445 with default scripts
2. Probe for null session with `smbclient -N -L`
3. Check share permissions with `smbmap`
4. Browse accessible shares, pull interesting files
5. Use `rpcclient` for user, group, and domain enumeration
6. Brute force RID range if `enumdomusers` is restricted
7. Run `enum4linux-ng -A` for a consolidated sweep
8. Run `samrdump.py` to cross-check user enumeration

---

## Nmap

```bash
sudo nmap -sV -sC -p139,445 <target>

# Check for EternalBlue (if SMB 1 detected)
nmap --script smb-vuln-ms17-010 -p445 <target>

# OS discovery
nmap --script smb-os-discovery.nse -p445 <target>
```

---

## smbclient

```bash
# List shares -- null session
smbclient -N -L //<target>

# Connect to a share anonymously
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
!cat <file>           # read a local file without dropping session
```

---

## smbmap

```bash
# Null session -- shows READ/WRITE/NO ACCESS per share
smbmap -H <target>

# Authenticated
smbmap -H <target> -u <user> -p <password>

# Recursive listing of a share
smbmap -H <target> -r <share>

# Download a file directly
smbmap -H <target> --download '<share>\<file>'
```

---

## rpcclient

```bash
# Null session
rpcclient -N -U "" <target>
```

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

**RID brute force** (when `enumdomusers` is restricted):

```bash
for i in $(seq 500 1100); do
    rpcclient -N -U "" <target> -c "queryuser 0x$(printf '%x' $i)" | grep -E "User Name|user_rid|group_rid" && echo ""
done
```

---

## enum4linux-ng

```bash
./enum4linux-ng.py <target> -A
```

Covers: shares, users, groups, OS info, password policy, NetBIOS names, domain info in one sweep.

---

## samrdump.py

```bash
samrdump.py <target>
```

SAMR-based user enumeration. Returns usernames, UIDs, password last set, expiry status, and logon count.

---

## CrackMapExec

```bash
# Null session share enum
crackmapexec smb <target> --shares -u '' -p ''

# Authenticated share enum
crackmapexec smb <target> --shares -u <user> -p <password>

# Enumerate logged-on users
crackmapexec smb <target> --loggedon-users -u <user> -p <password>

# Dump SAM hashes (requires admin)
crackmapexec smb <target> --sam -u <user> -p <password>
```

---

## What to Look For

- **Null session allowed** -- enumerate everything before using credentials. Shares, users, domain info, password policy.
- **Browseable shares with sensitive names** -- `dev`, `notes`, `backup`, `home`, `it`, `scripts`. Browse them all.
- **Writable shares** -- potential file upload path. If also served by a web server, this is an RCE vector via webshell upload.
- **Users and RIDs from rpcclient** -- valid usernames for password spraying.
- **Password policy** -- check lockout threshold via `querydominfo` or enum4linux-ng. No lockout = spray aggressively.
- **SMB signing not required** -- prerequisite for NTLM relay attacks. Flag it.
- **SMB 1 running** -- check for EternalBlue (MS17-010). If it's running, assume unpatched.
- **Domain membership** -- opens AD-specific attack paths. See [[06-AD/]].
