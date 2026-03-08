
# Nmap — Packet Behaviour & Port States

See Also: [[Nmap|Nmap - Commands & Flags]]

# Port States

##### Open:
Port is accepting connections. Target responded with SYN-ACK.

##### Closed:
Port is accessible but nothing is listening. Target responded with RST.

##### Filtered:
Nmap cannot determine state. No response received — packet likely dropped or ignored by a firewall.

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
