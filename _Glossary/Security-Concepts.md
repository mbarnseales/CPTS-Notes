
# Security Concepts

Fundamental security infrastructure concepts referenced throughout enumeration, exploitation, and evasion. Understanding what these systems do and how they differ informs how you scan, move, and stay quiet.

# Firewall

##### Purpose:
Controls network traffic between networks or hosts based on a defined ruleset. Acts as a gatekeeper — packets are either allowed, dropped, or rejected based on rules configured by the administrator.

##### Packet handling:
- **Dropped** — packet is silently discarded, no response sent. Nmap marks the port as `filtered`. Scan is slower (retries up to 10 times).
- **Rejected** — packet is refused and an ICMP error is returned (e.g. Type 3 — Destination Unreachable). Nmap marks the port as `filtered`. Scan is fast.

##### Common ICMP rejection codes:
| Code | Meaning |
|------|---------|
| Net Unreachable | The target network is not reachable |
| Net Prohibited | Network access is administratively blocked |
| Host Unreachable | The specific host is not reachable |
| Host Prohibited | Host access is administratively blocked |
| Port Unreachable | The port is not open on the host |
| Proto Unreachable | The protocol is not supported |

##### Evasion approach:
Firewalls typically block inbound **SYN** packets (new connections) but allow **ACK** packets (which look like established connection responses). ACK scans (`-sA`) exploit this. Source port spoofing (`--source-port 53`) exploits trust rules for DNS traffic. See [[Nmap|Nmap - Firewall & IDS/IPS Evasion]].

# IDS — Intrusion Detection System

##### Purpose:
Passively monitors network traffic for suspicious patterns and signatures. Analyses connections between hosts and alerts the administrator when a potential attack is detected. **Does not block traffic** — only observes and reports.

##### Detection method:
Pattern matching against known attack signatures (e.g. recognising a service detection scan by the volume and structure of packets). More sophisticated IDS systems use behavioural analysis.

##### Evasion approach:
Because IDS is passive, the goal is to stay below detection thresholds — slow scans (`-T1`, `-T2`), decoys (`-D`) to create noise, and fragmentation. If an IP gets flagged, the administrator is notified but the IDS itself does not block anything.

# IPS — Intrusion Prevention System

##### Purpose:
Extends IDS with active response capability. When a potential attack is detected, the IPS automatically takes defensive action — typically blocking the source IP without waiting for administrator input.

##### Relationship to IDS:
IPS is a complement to IDS, not a replacement. IDS detects and reports — IPS detects and acts. Both can coexist on the same network.

##### Evasion approach:
IPS is the more dangerous system to trigger. If blocked, the engagement is disrupted. Use VPS instances with different IPs so that if one is blocked, scanning continues from another. Test aggressiveness carefully — a single aggressive scan can burn an IP.

# DMZ — Demilitarized Zone

##### Purpose:
A network segment that sits between the public internet and the internal corporate network. Hosts publicly accessible services (web servers, mail servers, DNS) in an isolated zone so that a compromise of those services does not directly expose the internal network.

##### Relevance to enumeration:
Internal DNS servers within a DMZ are typically more trusted by the target infrastructure than external resolvers. Using `--dns-server` to query internal DNS can reveal internal hostnames and services not visible from outside. See [[Nmap|Nmap - Firewall & IDS/IPS Evasion]].
