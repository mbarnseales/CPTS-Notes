
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
| `-O` | OS detection |
| `-sC` | Run default NSE scripts |
| `-sV` | Version scan (fingerprint services) |
| `-A` | Aggressive: `-sV` + `-O` + `--traceroute` + `-sC` |
| `-p-` | Scan all 65,535 TCP ports |
| `-p <port>` | Scan specific port(s) |
| `-PE` | Force ICMP Echo request for host discovery |
| `-iL <file>` | Read targets from file |
| `-oN <name>` | Save output in normal format (.nmap) |
| `-oG <name>` | Save output in grepable format (.gnmap) |
| `-oX <name>` | Save output in XML format (.xml) |
| `-oA <name>` | Save output in all formats (normal, XML, grepable) |
| `--top-ports=<n>` | Scan top N most frequent ports |
| `--traceroute` | Trace packet route to target |
| `--script <name>` | Run a specific NSE script or category |
| `--packet-trace` | Show all packets sent and received |
| `--reason` | Show why a host/port was assigned its state |
| `--disable-arp-ping` | Force ICMP instead of defaulting to ARP on local network |
| `--stats-every=<time>` | Print scan progress at set intervals (e.g. `5s`, `1m`) |
| `-v` | Verbose — show open ports as discovered |
| `-vv` | Very verbose — more detail during scan |
| `-T<0-5>` | Timing template — controls scan aggressiveness |
| `--min-rate <n>` | Send at least N packets per second |
| `--max-retries <n>` | Max port scan probe retransmissions (default: 10) |
| `--initial-rtt-timeout <time>` | Initial round-trip-time timeout (e.g. `50ms`) |
| `--max-rtt-timeout <time>` | Maximum round-trip-time timeout (e.g. `100ms`) |
| `--min-parallelism <n>` | Minimum number of probes in parallel |

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

## NSE — Nmap Scripting Engine

Scripts written in Lua that interact with services for deeper enumeration, vuln detection, brute forcing, and more.

### Script Categories

| Category | Description |
|----------|-------------|
| `auth` | Credential determination |
| `broadcast` | Host discovery via broadcast — found hosts added to scan |
| `brute` | Brute-force login attempts against services |
| `default` | Run with `-sC` — safe, commonly useful scripts |
| `discovery` | Service and host information gathering |
| `dos` | Denial of service checks — use with caution |
| `exploit` | Attempts to exploit known vulnerabilities |
| `external` | Uses external services for processing |
| `fuzzer` | Identifies unexpected packet handling — slow |
| `intrusive` | May negatively affect the target |
| `malware` | Checks for malware infection indicators |
| `safe` | Non-intrusive, non-destructive scripts |
| `version` | Extends service version detection |
| `vuln` | Identifies specific known vulnerabilities |

### Script Usage

```bash
# Run default scripts
sudo nmap <target> -sC

# Run all scripts in a category
sudo nmap <target> --script <category>

# Run specific named script(s)
sudo nmap <target> --script <script-name>,<script-name>

# Vuln scan on a specific port
sudo nmap <target> -p 80 -sV --script vuln

# Banner + SMTP commands
sudo nmap <target> -p 25 --script banner,smtp-commands
```

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
sudo nmap -sV -sC -p- <target> -oA full

# Fast scan — top 100 ports
sudo nmap <target> -F -oA fast

# Top 10 most frequent ports
sudo nmap <target> --top-ports=10 -oA top10

# UDP scan — top 100 ports
sudo nmap <target> -sU -F -oA udp

# Aggressive scan (version + OS + traceroute + default scripts)
sudo nmap <target> -A -oA aggressive

# Banner grabbing with Nmap
sudo nmap -sV --script=banner <target>

# Manual banner grab with nc (catches what Nmap may miss)
nc -nv <target> <port>

# Specific port with script
sudo nmap --script <script-name> -p <port> <target>
```

---

## Performance & Timing

> [!warning] Speed vs Accuracy
> Every performance optimisation is a trade-off. Reducing timeouts or retries can cause Nmap to miss hosts and ports. Validate aggressive scans with a slower follow-up if results look thin.

### Timing Templates

| Template | Flag | Use Case |
|----------|------|----------|
| Paranoid | `-T0` | Maximum evasion — extremely slow |
| Sneaky | `-T1` | Evasion focused |
| Polite | `-T2` | Low bandwidth / avoid disrupting services |
| Normal | `-T3` | Default |
| Aggressive | `-T4` | Fast networks — good for labs and HTB |
| Insane | `-T5` | Fastest — may miss results, can trigger IDS |

> `-T4` is the go-to for lab environments. Use `-T2` or lower on real engagements where stealth matters.

### Performance Flags

```bash
# Timing template (T4 for labs, T2 for stealth)
sudo nmap <target> -T4

# Set packet rate — useful when whitelisted and bandwidth is known
sudo nmap <target> --min-rate 300

# Reduce retries — speeds up scan but may miss filtered ports
sudo nmap <target> --max-retries 0

# Tune RTT timeouts for fast local networks
sudo nmap <target> --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
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
