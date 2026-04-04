
# SNMP

Layer 3 enumeration. SNMP is used to monitor and manage network devices -- routers, switches, servers, IoT devices. From an attacker's perspective it can expose OS info, running processes, installed packages, network config, and credentials depending on what the device broadcasts and what access the community string grants.

For MIB, OID, and community string definitions see [[_Glossary/SNMP-Glossary|SNMP Glossary]].

> [!warning] UDP Service
> SNMP runs on UDP. Use `-sU` with Nmap or you will miss it entirely on a standard TCP scan.

---

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 161 | UDP | SNMP queries and commands |
| 162 | UDP | SNMP traps -- unsolicited alerts sent from device to manager |

---

## Workflow

1. Nmap UDP scan on ports 161 and 162
2. Try default community strings (`public`, `private`) with `snmpwalk`
3. If no access, brute force community strings with `onesixtyone`
4. Once a valid string is found, enumerate all OIDs with `braa`
5. If you gain server access, check `/etc/snmp/snmpd.conf` for `rwcommunity` or `rwuser noauth`

---

## Nmap

```bash
# UDP service scan
sudo nmap -sU -sV -p161,162 <target>

# SNMP NSE scripts
sudo nmap -sU -p161 --script snmp-info,snmp-sysdescr,snmp-win32-users <target>
```

---

## snmpwalk

Walks the OID tree and returns all values the community string has access to.

```bash
# SNMPv2c with public string -- full walk
snmpwalk -v2c -c public <target>

# SNMPv2c -- specific OID subtree
snmpwalk -v2c -c public <target> <OID>

# Try private string
snmpwalk -v2c -c private <target>
```

A successful walk against a misconfigured device can return OS version, hostname, contact email, uptime, installed packages, running processes, and network interfaces in one command. See [[_Tools/SNMP|SNMP Tools]] for full flag reference.

---

## onesixtyone

Brute forces community strings. Use when default strings fail.

```bash
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <target>
```

Returns the valid community string and the system description. Feed it straight into `snmpwalk` or `braa`.

---

## braa

Mass OID enumeration once a community string is known. Faster than `snmpwalk` for large sweeps.

```bash
braa public@<target>:.1.3.6.*
```

---

## Dangerous Settings

Config file: `/etc/snmp/snmpd.conf`

```bash
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
```

| Setting | Risk |
|---------|------|
| `rwuser noauth` | Full read-write OID tree access with no authentication |
| `rwcommunity <string> <IP>` | Read-write access for any source using that community string |
| `rwcommunity6 <string> <IPv6>` | Same over IPv6 |
| `rocommunity public default` | Read-only access granted to everyone with the default string |

---

## What to Look For

- **Default community strings** -- `public` and `private` are left in place far more often than they should be. Always try before brute forcing.
- **Read-write access** -- `rwcommunity` or `rwuser noauth` means remote device config modification. Critical on network infrastructure.
- **OS and kernel version** -- first OID in a walk. Cross-reference with CVEs.
- **Contact email** -- real admin address, useful for social engineering scope.
- **Installed packages** -- full list from `snmpwalk` can reveal vulnerable software versions.
- **Running processes** -- can expose cleartext arguments including credentials passed on the command line.
- **Network interfaces and routing** -- internal subnet and routing info useful for network mapping.
- **SNMPv1 or v2c in use** -- community strings sent in plaintext. Capture passively with [[_Tools/tcpdump|tcpdump]] if you have a network position.
- **SNMPv3 with weak credentials** -- version 3 is authenticated but still brute-forceable with weak username/password pairs.
