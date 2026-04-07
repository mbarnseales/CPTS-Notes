# OpenVAS (Greenbone)

Part of the **Greenbone Vulnerability Manager (GVM)** suite. Open-source, no host limit.

Web UI: `https://127.0.0.1:9392` (default) or `https://<IP>:8080` (hosted)

---

## Installation (Parrot/Kali)

```bash
sudo apt-get update && apt-get -y full-upgrade
sudo apt-get install gvm && openvas

gvm-setup    # Initial setup — takes up to 30 min, generates admin password
gvm-start    # Start services
```

> Save the generated admin password from `gvm-setup` output.

---

## Scan Configurations

| Config | Checks Vulns? | Purpose |
|---|---|---|
| **Base** | No | Host status + OS info only |
| **Discovery** | No | Services, ports, software, hardware |
| **Host Discovery** | No | Alive check only (ping-based) |
| **System Discovery** | No | Extended Discovery — OS + hardware identification |
| **Full and fast** | Yes | Recommended — uses NVT intelligence to select best checks per host |

> Only use the configs above — other options can cause system disruptions.

---

## Workflow

### 1. Add Target (Configurations → Targets)

- Set: Name, Host IP(s), Port List, Alive Test
- **Alive Test default**: uses NVT Ping Host (NVT Family)
- Add credentials here for authenticated scans

### 2. Create Scan Task (Scans → Tasks → Wizard icon)

- Set: Name, Scan Target (from targets list), Scan Config
- Authenticated scans require a high-privilege account (root / Administrator) for maximum findings

### 3. Credentials (for authenticated scans)

| Target | Credentials |
|---|---|
| Linux | `htb-student_adm : HTB_@cademy_student!` |
| Windows | `administrator : Academy_VA_adm1!` |

---

## NVT Families

OpenVAS has **~96,000+ NVTs** across families including: Windows, Linux, Web Applications, Databases, Network devices, etc.

---

## Notes

- No host limit (unlike Nessus Essentials)
- Scans take 30–60 min per host — review as findings come in
- NVT feed updates via `gvm-feed-update`
- `gvm-check-setup` — verify installation is healthy
