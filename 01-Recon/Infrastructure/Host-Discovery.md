
# Host Discovery

Before scanning ports or fingerprinting services, confirm which hosts are alive. Scanning dead hosts wastes time and creates noise. Always save every scan.

> [!info] See Also
> [[Nmap|Nmap - Commands & Flags]] | [[Nmap|Nmap - Packet Behaviour & Port States]] | [[Networking|Networking - Protocols & Concepts]]

> [!warning] Always Output
> Every scan gets `-oA <name>`. No exceptions. You need this for documentation, reporting, and comparing results across tools.

---

## Workflow


1. Identify scope (range, list, or single IP)
2. Run ping sweep to find live hosts
3. If hosts appear down  -  don't assume they are. Firewalls block ICMP.
4. Try alternate discovery methods (ARP, TCP, UDP probes)
5. Document everything  -  what was scanned, when, and what responded

---

## Ping Sweep  -  By Scope

```bash
# Network range
sudo nmap 10.129.2.0/24 -sn -oA tnet

# From provided IP list
sudo nmap -sn -oA tnet -iL hosts.lst

# Multiple specific IPs
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20

# IP range shorthand
sudo nmap -sn -oA tnet 10.129.2.18-20

# Single host
sudo nmap 10.129.2.18 -sn -oA host
```

---

## When Hosts Appear Down

A host that doesn't respond to a ping sweep isn't necessarily offline  -  it may be filtering ICMP. Try:

```bash
# Force ICMP echo (bypass ARP default on local subnet)
sudo nmap 10.129.2.18 -sn -PE --disable-arp-ping -oA host

# See exactly why Nmap marked host as up or down
sudo nmap 10.129.2.18 -sn -PE --disable-arp-ping --reason -oA host

# Trace every packet sent and received
sudo nmap 10.129.2.18 -sn -PE --disable-arp-ping --packet-trace -oA host
```

> On a local subnet, Nmap defaults to ARP before ICMP  -  see [[Nmap|Nmap Glossary: ARP vs ICMP]] for why this matters.

---

## Output Formats

`-oA <name>` saves three files simultaneously:

| Extension | Format | Use |
|-----------|--------|-----|
| `.nmap` | Normal | Human-readable, good for notes |
| `.xml` | XML | Tool-parseable, good for imports |
| `.gnmap` | Grepable | Fast parsing with `grep`, `cut`, `awk` |

```bash
# Pull live hosts from grepable output
grep "Up" tnet.gnmap | cut -d" " -f2
```
