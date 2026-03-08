
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
| `-sC` | Run default scripts |
| `-sV` | Version scan (fingerprint services) |
| `-p-` | Scan all 65,535 TCP ports |
| `-p <port>` | Scan specific port(s) |
| `-A` | Aggressive scan (OS detection, version, scripts, traceroute) |
| `-PE` | Force ICMP Echo request for host discovery |
| `-iL <file>` | Read targets from file |
| `-oA <name>` | Save output in all formats (normal, XML, grepable) |
| `--script <name>` | Run a specific NSE script |
| `--script=banner` | Banner grabbing |
| `--packet-trace` | Show all packets sent and received |
| `--reason` | Show why a host/port was assigned its state |
| `--disable-arp-ping` | Force ICMP instead of defaulting to ARP on local network |

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
