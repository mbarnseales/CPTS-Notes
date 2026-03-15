
# NFS

Layer 3 enumeration. NFS (Network File System) is a distributed file system protocol developed by Sun Microsystems. Its purpose is the same as SMB -- share filesystems over a network as if they were local -- but it uses a completely different protocol and is native to Linux/Unix environments. NFS clients cannot communicate with SMB servers and vice versa.

> [!warning] UID/GID Authentication
> NFS has no built-in authentication or authorisation mechanism. Access control is handled entirely by the RPC layer using client UID/GID mappings. If you can control your local UID/GID, you can often impersonate users on a mounted share.

---

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 2049 | TCP/UDP | NFS service (primary) |
| 111 | TCP/UDP | RPC portmapper / rpcbind |

NFSv4 and above only require port 2049, which simplifies firewall traversal. Older versions use portmapper on 111 to dynamically assign additional ports for `mountd`, `nlockmgr`, etc.

---

## NFS Versions

| Version | Key Features                                                                                                                                                                      |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NFSv2   | Legacy. Originally UDP only. Widely supported.                                                                                                                                    |
| NFSv3   | Variable file sizes, better error reporting. Not fully backwards compatible with NFSv2.                                                                                           |
| NFSv4   | Kerberos support, stateful protocol, ACLs, firewall-friendly (single port 2049), no portmapper required. First version to authenticate users rather than just the client machine. |
| NFSv4.1 | Adds pNFS (parallel NFS) for distributed/clustered storage and session trunking (multipathing).                                                                                   |

---

## How Authentication Works

NFS delegates authentication entirely to the RPC protocol. The most common method is UNIX UID/GID:

- The server trusts the UID/GID the client presents
- The server maps those IDs to its own local users and applies the corresponding filesystem permissions
- There is no server-side verification of whether the client's UID mapping is legitimate
- If the client and server have different UID/GID to username mappings, access decisions will be based on the numeric ID alone

This is why NFS with UNIX authentication should only be used on trusted internal networks, and why mismatched UID mappings are a common privilege escalation vector.

---

## Configuration

Config file: `/etc/exports`

Defines which directories are shared, to which hosts, and with what options:

```bash
cat /etc/exports
```

### Export Options

| Option | Description |
|--------|-------------|
| `rw` | Read and write permissions |
| `ro` | Read only permissions |
| `sync` | Synchronous data transfer (slower, safer) |
| `async` | Asynchronous data transfer (faster, less safe) |
| `secure` | Restrict to ports below 1024 (root-owned ports only) |
| `insecure` | Allow ports above 1024 |
| `no_subtree_check` | Disable subdirectory tree checking. Improves reliability when files are renamed while open. |
| `root_squash` | Remap root (UID/GID 0) to the anonymous user. Prevents remote root from owning files. |

### Add and Apply an Export

```bash
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs    # verify active exports
```

---

## Dangerous Settings

| Option | Risk |
|--------|------|
| `rw` | Allows writing to the share, not just reading |
| `insecure` | Allows connections from unprivileged ports (above 1024). Any process, not just root, can connect. |
| `nohide` | Exposes filesystems mounted below an exported directory as part of that export |
| `no_root_squash` | Remote root retains UID/GID 0 on the share. Combined with write access, this means remote root can own any file on the export. |

> [!danger] no_root_squash
> If `no_root_squash` is set and the share is writable, you can place a SUID binary on the share as root and execute it to escalate locally. See the privilege escalation note below.

---

## Enumeration Workflow

1. Nmap on ports 111 and 2049 with default scripts
2. Run `nfs*` NSE scripts to enumerate exports, contents, and stats without mounting
3. `showmount -e` to list available exports
4. Mount the share locally and inspect contents
5. Check file ownership -- use `ls -l` for names, `ls -n` for raw UIDs/GIDs
6. Look for UID/GID mismatches you can exploit locally
7. Check for writable shares with `no_root_squash` for privesc potential

---

## Nmap

```bash
# Service scan on NFS ports
sudo nmap -sV -sC -p111,2049 <target>

# Run all NFS NSE scripts (ls, showmount, statfs, rpcinfo)
sudo nmap --script nfs* -sV -p111,2049 <target>
```

The `rpcinfo` script lists all running RPC services, their program numbers, versions, and dynamic ports. The `nfs-ls` script will attempt to list share contents without mounting. The `nfs-showmount` script shows available exports.

---

## showmount

Query the server's mountd to list available exports:

```bash
showmount -e <target>
```

Returns the export path and the allowed client range (e.g. `10.129.14.0/24` or `*` for everyone).

---

## Mounting and Inspecting

```bash
# Create mount point
mkdir target-NFS

# Mount the NFS share (nolock avoids needing lockd on older exports)
sudo mount -t nfs <target>:/ ./target-NFS/ -o nolock

# Navigate and inspect
cd target-NFS
tree .

# List with usernames and group names
ls -l mnt/nfs/

# List with raw UIDs and GIDs
ls -n mnt/nfs/
```

> [!tip] ls -l vs ls -n
> `ls -l` resolves UIDs/GIDs to names using your local `/etc/passwd` and `/etc/group`. If the server's users don't exist locally, you'll see the numeric IDs anyway. Use `ls -n` to always see the raw numbers, which is what matters for impersonation.

---

## Unmounting

```bash
cd ..
sudo umount ./target-NFS
```

Always unmount cleanly. Leaving NFS shares mounted can cause issues if the target goes down mid-session.

---

## UID/GID Impersonation

Because NFS trusts the UID/GID the client presents, you can impersonate a user whose files you can see on the share:

1. Mount the share and run `ls -n` to get the UID of the file owner
2. Create a local user with that same UID on your attack machine
3. Switch to that user and read/write files as if you were them on the remote system

```bash
# Create a user with a matching UID (e.g. 1000)
sudo useradd -u 1000 fakeuser
sudo su fakeuser

# Now read files owned by UID 1000 on the mounted share
cat ./target-NFS/mnt/nfs/<file>
```

---

## Privilege Escalation via NFS

If `no_root_squash` is set on a writable share and you have SSH access to the target:

1. As root on your attack machine, copy a shell binary to the mounted share
2. Set the SUID bit on it (`chmod +s`)
3. SSH into the target and execute the binary from the NFS path
4. The shell runs with the UID of the owner (root) -- privilege escalation achieved

> [!warning]
> This only works when `no_root_squash` is set. With `root_squash` (the default), your root writes are remapped to the anonymous UID and the SUID bit will not grant elevated access.

See [[05-Post-Exploitation/Linux-Privilege-Escalation|Linux Privilege Escalation]] for broader context.

---

## What to Look For

- **World-readable exports (`*`)** -- any host can mount. Enumerate everything immediately.
- **no_root_squash on a writable share** -- SUID shell privesc. High severity finding.
- **insecure option set** -- weakens the port restriction. Lower barrier for exploitation.
- **SSH keys or credentials on the share** -- `id_rsa`, `.env`, config files with passwords. Pull and inspect anything non-obvious.
- **UID/GID mismatch opportunity** -- files owned by a UID you can recreate locally. Impersonate and read restricted files.
- **Backup scripts or cron-adjacent files** -- writable scripts owned by a privileged user are a privesc path if they run on a schedule.
- **nohide on nested mounts** -- can expose filesystems the admin did not intend to share.
