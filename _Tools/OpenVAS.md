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

## Scan Capabilities

- Authenticated and unauthenticated scanning
- Network vulnerability scanning
- CVE/plugin-based detection (NVT feed)
- Compliance auditing

---

## Notes

- No host limit (unlike Nessus Essentials)
- NVT (Network Vulnerability Tests) feed updates regularly via `gvm-feed-update`
- `gvm-check-setup` — verify installation is healthy
