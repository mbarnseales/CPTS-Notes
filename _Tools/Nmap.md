
# Nmap

Network Mapper — open-source tool for network discovery, port scanning, service/version detection, and OS fingerprinting. The go-to for initial enumeration on any engagement.

> [!info] See Also
> [[Nmap|Nmap - Packet Behaviour & Port States]]

---

## Scan Types

| Flag | Scan Type | Notes |
|------|-----------|-------|
| `-sS` | TCP SYN | Default. Half-open, never completes handshake. Requires sudo |
| `-sT` | TCP Connect | Full 3-way handshake. Use when sudo unavailable |
| `-sA` | TCP ACK | Maps firewall rules, not open ports |
| `-sW` | TCP Window | Like ACK but checks TCP window size |
| `-sM` | Maimon | FIN/ACK probe |
| `-sU` | UDP | Slow but critical — DNS, SNMP, DHCP |
| `-sN` | TCP Null | No flags set |
| `-sF` | TCP FIN | FIN flag only |
| `-sX` | TCP Xmas | FIN, PSH, URG flags |
| `-sI` | Idle/Zombie | Spoofs scan through zombie host |
| `-sO` | IP Protocol | Detects supported IP protocols |

---

## Flags

| Flag | Description |
|------|-------------|
| `-sC` | Run default scripts |
| `-sV` | Version scan (fingerprint services) |
| `-p-` | Scan all 65,535 TCP ports |
| `-p <port>` | Scan specific port(s) |
| `-A` | Aggressive scan (OS detection, version, scripts, traceroute) |
| `--script <name>` | Run a specific NSE script |
| `--script=banner` | Banner grabbing |

---

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

---

## Useful Scripts

```bash
# SMB OS discovery
nmap --script smb-os-discovery.nse -p445 <target>

# Find Citrix scripts
locate scripts/citrix
```
