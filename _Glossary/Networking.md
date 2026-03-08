
# Networking — Protocols & Concepts

Core networking protocols referenced throughout enumeration, exploitation, and post-exploitation. Understanding these helps you reason through tool behavior and unexpected scan results.

# ARP — Address Resolution Protocol

##### Layer:
2 (Data Link)

##### Purpose:
Resolves an IP address to a MAC address on a local network. Before a device can send traffic to another device on the same subnet, it needs the destination's MAC address. ARP handles this mapping.

```
Host A wants to reach 10.129.2.18:

Host A  →  ARP Request: "Who has 10.129.2.18?"  →  Broadcast (everyone on subnet)
Host A  ←  ARP Reply:   "10.129.2.18 is at DE:AD:00:00:BE:EF"  ←  Target
```

##### Key Points:
- Only works on the **local subnet** — ARP cannot cross a router
- Responses are cached in the ARP table (`arp -a`)
- Nmap defaults to ARP for host discovery on local networks before trying ICMP — an ARP reply alone is enough to confirm a host is alive

# ICMP — Internet Control Message Protocol

##### Layer:
3 (Network)

##### Purpose:
Used by devices to send error messages and operational information across a network. Not used for data transfer. The most well-known use is the ping — an Echo Request/Reply pair used to check if a host is reachable.

```
Host A  →  ICMP Echo Request (Type 8)  →  Target
Host A  ←  ICMP Echo Reply  (Type 0)  ←  Target  (host is alive)
```

##### Common ICMP Types:

| Type | Name | Meaning |
|------|------|---------|
| 0 | Echo Reply | Response to a ping |
| 3 | Destination Unreachable | Host/port/network not reachable |
| 8 | Echo Request | Ping |
| 11 | Time Exceeded | TTL expired in transit — used by traceroute |

##### Key Points:
- Works **across subnets** — unlike ARP
- Commonly blocked or rate-limited by firewalls — a non-response does not mean the host is down
- Nmap uses `-PE` to force ICMP Echo Requests for host discovery
- Use `--disable-arp-ping` to stop Nmap defaulting to ARP on local networks

# TTL — Time To Live

##### Layer:
3 (Network)

##### Purpose:
Limits how many hops a packet can traverse before being discarded. Prevents packets from looping indefinitely through the network. Each router that forwards a packet decrements the TTL by 1. If TTL reaches 0, the packet is dropped and an ICMP Type 11 (Time Exceeded) message is sent back to the sender.

##### Default TTL Values by OS:
Useful for passive OS fingerprinting — a received TTL slightly below these values indicates the number of hops between you and the target.

| OS | Default TTL |
|----|-------------|
| Linux | 64 |
| Windows | 128 |
| Solaris / AIX | 254 |
