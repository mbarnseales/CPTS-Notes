
# SMB Tools Reference

Quick command reference for SMB enumeration tools. For theory, workflow, and what to look for, see [[01-Recon/SMB-Enumeration|SMB Enumeration]].

Default ports: `445` (TCP), `139` (TCP legacy NetBIOS)

---

## smbclient

```bash
# List shares -- null session
smbclient -N -L //<target>

# Connect to share (anonymous)
smbclient //<target>/<share>

# Connect with credentials
smbclient -U <user> //<target>/<share>
```

Inside the session:
```bash
ls                  # list directory
cd <dir>            # change directory
get <file>          # download file
put <file>          # upload file
!<cmd>              # run local shell command mid-session
```

---

## rpcclient

```bash
# Null session
rpcclient -U "" <target>
rpcclient -N -U "" <target>
```

| Query | Description |
|-------|-------------|
| `srvinfo` | OS, platform ID, server type |
| `enumdomains` | List deployed domains |
| `querydominfo` | Domain/server info, user and group counts |
| `netshareenumall` | All shares with paths |
| `netsharegetinfo <share>` | Share detail including ACLs |
| `enumdomusers` | Users with RIDs |
| `queryuser <RID>` | Full user record |
| `querygroup <RID>` | Group info and member count |

**RID brute force:**
```bash
for i in $(seq 500 1100); do
  rpcclient -N -U "" <target> -c "queryuser 0x$(printf '%x\n' $i)" \
  | grep "User Name\|user_rid\|group_rid" && echo ""
done
```

---

## smbmap

```bash
# Null session
smbmap -H <target>

# Authenticated
smbmap -H <target> -u <user> -p <password>

# Recursive share listing
smbmap -H <target> -r <share>

# Download file
smbmap -H <target> --download '<share>\<file>'
```

---

## CrackMapExec

```bash
# Null session share enum
crackmapexec smb <target> --shares -u '' -p ''

# Authenticated share enum
crackmapexec smb <target> --shares -u <user> -p <password>

# Logged-on users
crackmapexec smb <target> --loggedon-users -u <user> -p <password>

# Dump SAM (requires admin)
crackmapexec smb <target> --sam -u <user> -p <password>
```

---

## enum4linux-ng

```bash
# Full automated enumeration
./enum4linux-ng.py <target> -A
```

Covers: shares, users, groups, OS info, password policy, NetBIOS names, domain info.

---

## Impacket - samrdump.py

```bash
samrdump.py <target>
```

SAMR-based user enumeration. Returns usernames, UIDs, and account status.

---

## Nmap

```bash
# Service scan on SMB ports
sudo nmap -sV -sC -p139,445 <target>

# Check for EternalBlue (MS17-010)
nmap --script smb-vuln-ms17-010 -p445 <target>

# OS discovery via SMB
nmap --script smb-os-discovery.nse -p445 <target>
```

---

## SMB Versions

| Version | Platform | Notes |
|---------|----------|-------|
| CIFS / SMB 1 | Windows NT 4.0 / 2000 | Legacy. NetBIOS interface. Affected by EternalBlue. |
| SMB 2.0 | Windows Vista / Server 2008 | Performance improvements, signing, caching |
| SMB 2.1 | Windows 7 / Server 2008 R2 | Locking mechanisms |
| SMB 3.0 | Windows 8 / Server 2012 | Multichannel, end-to-end encryption |
| SMB 3.1.1 | Windows 10 / Server 2016 | Integrity checking, AES-128 |

---

## Samba

Linux/Unix SMB implementation. Key points:
- Implements SMB/CIFS for cross-platform file sharing
- Two daemons: `smbd` (file/print sharing) and `nmbd` (NetBIOS name service)
- Samba 3 can join an AD domain as a member; Samba 4 can act as a full DC

Config file: `/etc/samba/smb.conf`

```bash
# Strip comments to see only active settings
cat /etc/samba/smb.conf | grep -v "#\|;"
```

---

## Samba Configuration

### Key Settings

| Setting | Description |
|---------|-------------|
| `workgroup` | Workgroup/domain name shown to querying clients |
| `path` | Filesystem path being shared |
| `server string` | Banner string shown on connection |
| `usershare allow guests` | Allow unauthenticated access |
| `map to guest = bad user` | Map unknown logins to the guest account |
| `browseable` | Show share in the available share list |
| `guest ok` | Allow connections without a password |
| `read only` | Prevent write operations |
| `create mask` | Permissions applied to newly created files |

### Dangerous Settings

| Setting | Risk |
|---------|------|
| `browseable = yes` | Exposes share list to unauthenticated users |
| `read only = no` / `writable = yes` | Allows file creation and modification |
| `guest ok = yes` | Passwordless access |
| `create mask = 0777` | New files are world-readable and writable |
| `directory mask = 0777` | New directories are world-accessible |
| `logon script = script.sh` | Executes a script at user logon |
| `magic script = script.sh` | Executes when the script file is closed |
| `enable privileges = yes` | Honours privilege assignments based on SID |
