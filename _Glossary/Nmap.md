
# Nmap — Packet Behaviour & Port States

See Also: [[Nmap|Nmap - Commands & Flags]]

# Port States

##### Open:
Port is accepting connections. Target responded with SYN-ACK.

##### Closed:
Port is accessible but nothing is listening. Target responded with RST.

##### Filtered:
Nmap cannot determine state. No response received — packet likely dropped or ignored by a firewall. See Filtered Ports — Dropped vs Rejected below.

##### Unfiltered:
Port is accessible but state is undetermined. ACK scan reached the port but open/closed cannot be confirmed.

##### Open|Filtered:
Open or filtered — cannot tell which. No response from a scan type that would normally expect one (e.g. UDP, Null, FIN, Xmas).

##### Closed|Filtered:
Closed or filtered — cannot tell which. Only seen during IP ID idle scans.

# TCP SYN Scan (-sS)

The default Nmap scan. Sends a SYN packet and waits — never completes the full three-way handshake, so no full TCP connection is established. Faster and stealthier than a full connect scan. Requires `sudo` as raw packet construction needs root privileges.

```
Attacker  →  SYN        →  Target
Attacker  ←  SYN-ACK   ←  Target  (open)
Attacker  →  RST        →  Target  (Nmap tears it down immediately)

Attacker  ←  RST        ←  Target  (closed)
Attacker  ←  [nothing]             (filtered)
```

# TCP Connect Scan (-sT)

Completes the full three-way TCP handshake. Used automatically when sudo is unavailable — the OS networking stack handles the connection rather than raw packets. More accurate than SYN but significantly noisier. Creates connection logs on most systems and is easily detected by modern IDS/IPS.

```
Attacker  →  SYN        →  Target
Attacker  ←  SYN-ACK   ←  Target  (open)
Attacker  →  ACK        →  Target  (connection fully established)

Attacker  ←  RST        ←  Target  (closed)
```

In `--packet-trace` output, Connect scan shows `CONN` instead of `SENT`/`RCVD`:

```
CONN (0.0385s) TCP localhost > 10.129.2.28:443 => Operation now in progress
CONN (0.0396s) TCP localhost > 10.129.2.28:443 => Connected
```

# Filtered Ports — Dropped vs Rejected

Both states show as `filtered` in results but behave very differently. The timing tells you which one you're dealing with.

##### Dropped:
Firewall silently discards the packet. No response is sent back. Nmap retries up to 10 times (`--max-retries`) before giving up. Scan takes significantly longer — ~2 seconds vs ~0.05 seconds for a normal response.

```
Attacker  →  SYN  →  Firewall  →  [packet discarded, no reply]
Attacker  →  SYN  →  Firewall  →  [retries...]
```

##### Rejected:
Firewall actively refuses the connection and sends back an ICMP Type 3 Code 3 (Port Unreachable) response. Scan completes quickly — you get a definitive answer.

```
Attacker  →  SYN          →  Firewall
Attacker  ←  ICMP Type 3  ←  Firewall  (port unreachable — filtered)
```

> A slow filtered result = dropped. A fast filtered result = rejected. Both mean the port is inaccessible but the firewall behaviour is different.

# UDP Scan (-sU)

UDP is stateless — no three-way handshake, no acknowledgment. Nmap sends empty datagrams and interprets the response (or lack of one) to determine port state. Significantly slower than TCP scans due to longer timeouts.

```
Attacker  →  UDP datagram  →  Target
```

##### Open:
Target application responds. Only happens if the application is configured to reply.

```
Attacker  ←  UDP response  ←  Target  (open)
```

##### Closed:
Target returns ICMP Type 3 Code 3 (Port Unreachable).

```
Attacker  ←  ICMP Type 3 Code 3  ←  Target  (closed)
```

##### Open|Filtered:
No response at all — Nmap cannot tell if the port is open or being filtered. This is the most common result in UDP scans.

```
Attacker  ←  [nothing]  (open|filtered)
```

> Critical UDP ports to always check: DNS (53), SNMP (161/162), DHCP (67/68), TFTP (69).

# Banner Grabbing

When Nmap connects to an open port, it first reads the service banner — a string the service sends immediately upon connection to identify itself. Nmap prints this as the version info in `-sV` results.

If no banner is present or recognised, Nmap falls back to **signature-based matching** — sending specific probes and comparing responses against its database. This is slower and less reliable.

##### Important:
Nmap does not always capture everything in a banner. Services sometimes send more detail than Nmap displays. A manual grab with `nc` directly reads the raw banner and can reveal additional information such as OS, distribution, or software version that Nmap filtered or missed.

```bash
nc -nv <target> <port>
```

The banner is delivered via a TCP packet with the **PSH flag** set — the server pushes data immediately to the client without waiting for a request. See [[Networking|TCP Flags]] for detail on PSH.

# Host Discovery — ARP vs ICMP

On a local subnet, Nmap defaults to sending an ARP request before attempting ICMP echo requests. An ARP reply alone is enough for Nmap to mark a host as alive — it never needs to send ICMP at all. This matters because a host may appear down during an ICMP-based sweep simply because Nmap didn't use the right method, not because the host is offline.

```
Attacker  →  ARP who-has 10.129.2.18?        →  Local Network
Attacker  ←  ARP reply: DE:AD:00:00:BE:EF    ←  Target (host marked alive)
```

To force ICMP echo requests instead — use when testing across subnets or when you need to confirm host state explicitly:

```bash
sudo nmap 10.129.2.18 -sn -PE --disable-arp-ping
```

```
Attacker  →  ICMP Echo Request  →  Target
Attacker  ←  ICMP Echo Reply   ←  Target  (host marked alive)
```

Use `--reason` to see exactly what response caused Nmap to mark a host as up or down.
Use `--packet-trace` to see every packet sent and received in real time.
