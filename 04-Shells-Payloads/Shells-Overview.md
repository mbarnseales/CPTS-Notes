# Shells & Payloads

## Shell Types

| Context | Description |
|---|---|
| **Computing** | Text-based interface to the OS — Bash, Zsh, cmd, PowerShell |
| **Exploitation** | Interactive access to a host gained by exploiting a vulnerability (e.g. EternalBlue → cmd shell) |
| **Web Shell** | Script uploaded to a web server — issues commands, reads files via browser request |

## Why a Shell Matters

- Direct OS/filesystem access
- Enables privilege escalation, pivoting, file transfer
- Persistence — stay on target longer
- CLI harder to detect than graphical access (VNC/RDP)
- Easier to automate

---

## Payload Definitions

In this context: **code crafted to exploit a vulnerability and deliver shell access.**

| Delivery Type | Notes |
|---|---|
| **Staged** | Small first-stage calls back to get the real payload — smaller footprint, requires C2 |
| **Stageless** | Full payload in one — noisier but simpler, no callback needed |

---

## Module Scope

- Bind shells, reverse shells, web shells
- Payload crafting (msfvenom, manual)
- Delivery methods
- Windows & Linux targets
