
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
