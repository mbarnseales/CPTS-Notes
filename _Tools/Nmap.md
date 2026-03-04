
# Nmap

## Flags
| Flag | Description |
|------|-------------|
| `-sC` | Run default scripts |
| `-sV` | Version scan (fingerprint services) |
| `-p-` | Scan all 65,535 TCP ports |
| `-p <port>` | Scan specific port(s) |
| `-A` | Aggressive scan (OS detection, version, scripts, traceroute) |
| `-sU` | UDP scan |
| `--script <name>` | Run a specific NSE script |
| `--script=banner` | Banner grabbing |

## Common Scans
```bash
# Default (top 1000 ports)
nmap <target>

# Full scan with version and scripts
nmap -sV -sC -p- <target>

# Banner grabbing
nmap -sV --script=banner <target>

# Specific port with script
nmap --script <script-name> -p <port> <target>

# Aggressive scan
nmap -A -p <port> <target>
```

## Useful Scripts
```bash
# SMB OS discovery
nmap --script smb-os-discovery.nse -p445 <target>

# Find Citrix scripts
locate scripts/citrix
```
