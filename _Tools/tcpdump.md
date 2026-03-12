
# tcpdump

Command-line packet capture tool. Captures and displays network traffic in real time. Essential for hunting cleartext credentials on internal interfaces and understanding what services are communicating on a compromised host.

See Also: [[Credential-Hunting]]

---

## Flags

| Flag | Description |
|------|-------------|
| `-i <interface>` | Interface to listen on (`any` for all) |
| `-n` | Don't resolve hostnames |
| `-nn` | Don't resolve hostnames or port names |
| `-A` | Print packet contents as ASCII |
| `-X` | Print packet contents as hex + ASCII |
| `-w <file>` | Write capture to file (.pcap) |
| `-r <file>` | Read from .pcap file |
| `-v` | Verbose |
| `-vv` | More verbose |
| `port <n>` | Filter by port number |
| `host <IP>` | Filter by host |
| `tcp` / `udp` | Filter by protocol |

---

## Common Captures

```bash
# Capture all traffic on all interfaces  -  print as ASCII
tcpdump -i any -nn -A

# Capture specific TCP port
tcpdump -i any -nn -A tcp port 80

# Capture traffic to/from a specific host
tcpdump -i any -nn -A host 10.10.10.1

# Save capture to file for analysis
tcpdump -i any -w capture.pcap

# Read and display saved capture
tcpdump -r capture.pcap -nn -A
```

---

## Cleartext Credential Hunting

After foothold, run tcpdump on internal interfaces to catch credentials transmitted without encryption. Common targets: FTP, HTTP basic auth, SMTP, telnet, internal APIs.

```bash
# Watch all internal traffic for readable strings
tcpdump -i any -nn -A

# Target a suspicious internal port
tcpdump -i any -nn -A tcp port <port>

# Watch loopback (services only accessible locally)
tcpdump -i lo -nn -A
```

Look for:
- `Authorization: Basic` headers (base64 encoded  -  decode immediately)
- `username=` / `password=` in POST bodies
- `USER` / `PASS` in FTP/SMTP traffic
- JSON payloads with credential fields

```bash
# Decode base64 credentials from Basic Auth
echo "<base64string>" | base64 -d
```

---

## Seen In

| Box | Interface | Port | What Was Found |
|-----|-----------|------|----------------|
| [[CCTV]] | any | 5000 | Cleartext `sa_mark` credentials from internal service traffic |
