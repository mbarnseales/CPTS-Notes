
# Nmap

Network Mapper — open-source tool for network discovery, port scanning, service/version detection, and OS fingerprinting. The go-to for initial enumeration on any engagement.

> [!info] See Also
> [[Nmap|Nmap - Packet Behaviour & Port States]]

> [!tip] Always Save Scans
> Use `-oA <name>` on every scan. Different tools produce different results — saved output lets you compare, document, and report accurately.

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
| `-sn` | Disable port scan (host discovery / ping sweep only) |
| `-Pn` | Skip host discovery — treat all hosts as online |
| `-n` | Disable DNS resolution (speeds up scan) |
| `-F` | Fast scan — top 100 ports |
| `-sC` | Run default scripts |
| `-sV` | Version scan (fingerprint services) |
| `-p-` | Scan all 65,535 TCP ports |
| `-p <port>` | Scan specific port(s) |
| `-A` | Aggressive scan (OS detection, version, scripts, traceroute) |
| `-PE` | Force ICMP Echo request for host discovery |
| `-iL <file>` | Read targets from file |
| `-oN <name>` | Save output in normal format (.nmap) |
| `-oG <name>` | Save output in grepable format (.gnmap) |
| `-oX <name>` | Save output in XML format (.xml) |
| `-oA <name>` | Save output in all formats (normal, XML, grepable) |
| `--top-ports=<n>` | Scan top N most frequent ports |
| `--script <name>` | Run a specific NSE script |
| `--script=banner` | Banner grabbing |
| `--packet-trace` | Show all packets sent and received |
| `--reason` | Show why a host/port was assigned its state |
| `--disable-arp-ping` | Force ICMP instead of defaulting to ARP on local network |
| `--stats-every=<time>` | Print scan progress at set intervals (e.g. `5s`, `1m`) |
| `-v` | Verbose — show open ports as discovered |
| `-vv` | Very verbose — more detail during scan |

---

## Port Specification

| Method | Example | Description |
|--------|---------|-------------|
| Specific | `-p 22,25,80` | Comma-separated individual ports |
| Range | `-p 22-445` | All ports within a range |
| Top N | `--top-ports=10` | N most frequently seen ports |
| All | `-p-` | All 65,535 ports |
| Fast | `-F` | Top 100 ports |

---

## Common Scans

### Host Discovery

```bash
# Ping sweep — network range
sudo nmap 10.129.2.0/24 -sn -oA tnet

# Ping sweep — from IP list
sudo nmap -sn -oA tnet -iL hosts.lst

# Ping sweep — multiple IPs
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20

# Ping sweep — IP range
sudo nmap -sn -oA tnet 10.129.2.18-20

# Single host — force ICMP (bypass ARP default on local subnet)
sudo nmap 10.129.2.18 -sn -oA host -PE --disable-arp-ping
```

### Port Scanning

> [!tip] Scan Strategy
> Run a quick scan first (`-F` or `--top-ports`) to get an immediate picture. Then kick off a full `-p- -sV` in the background. Minimises noise early while the deep scan runs.

```bash
# Default (top 1000 ports)
nmap <target>

# Full scan with version and scripts
nmap -sV -sC -p- <target>

# Fast scan — top 100 ports
sudo nmap <target> -F -oA fast

# Top 10 most frequent ports
sudo nmap <target> --top-ports=10 -oA top10

# UDP scan — top 100 ports
sudo nmap <target> -sU -F -oA udp

# Banner grabbing with Nmap
nmap -sV --script=banner <target>

# Manual banner grab with nc (catches what Nmap may miss)
nc -nv <target> <port>

# Specific port with script
nmap --script <script-name> -p <port> <target>

# Aggressive scan
nmap -A -p <port> <target>
```

---

## Output & Reporting

```bash
# Convert XML output to HTML report (readable for non-technical stakeholders)
xsltproc target.xml -o target.html
```

---

## Useful Scripts

```bash
# SMB OS discovery
nmap --script smb-os-discovery.nse -p445 <target>

# Find Citrix scripts
locate scripts/citrix
```
