
# Nmap — Packet Behaviour & Port States

> [!info] See Also
> [[Nmap|Nmap - Commands & Flags]]

---

## Port States

| State | Meaning | Why |
|-------|---------|-----|
| `open` | Port is accepting connections | Target responded with SYN-ACK |
| `closed` | Port is accessible but nothing listening | Target responded with RST |
| `filtered` | Nmap can't determine state | No response — packet likely dropped by firewall |
| `unfiltered` | Accessible but state undetermined | ACK scan reached port, but open/closed unclear |
| `open\|filtered` | Open or filtered, can't tell | No response from open-expected scan (e.g. UDP, Null, FIN, Xmas) |
| `closed\|filtered` | Closed or filtered, can't tell | Only seen in IP ID idle scans |

---

## TCP SYN Scan (-sS)

The default Nmap scan. Sends a SYN packet and waits — never completes the full 3-way handshake, so no full connection is established. Faster and stealthier than a full connect scan.

```
Attacker  →  SYN         →  Target
Attacker  ←  SYN-ACK     ←  Target  (open)
Attacker  →  RST         →  Target  (Nmap tears it down immediately)

Attacker  ←  RST         ←  Target  (closed)
Attacker  ←  [nothing]              (filtered)
```

> Requires `sudo` — raw packet construction needs root privileges.
