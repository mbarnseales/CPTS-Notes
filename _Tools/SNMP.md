
# SNMP Tools Reference

Quick command reference for SNMP enumeration. For workflow and what to look for see [[01-Recon/SNMP-Enumeration|SNMP Enumeration]].

Ports: `161` UDP (queries), `162` UDP (traps)

---

## snmpwalk

```bash
# SNMPv2c with public community string -- full walk
snmpwalk -v2c -c public <target>

# SNMPv2c -- specific OID subtree
snmpwalk -v2c -c public <target> <OID>

# SNMPv1
snmpwalk -v1 -c public <target>

# Try private string
snmpwalk -v2c -c private <target>
```

---

## onesixtyone

Brute force community strings:

```bash
# Single target
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <target>

# Multiple targets from file
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt -i targets.txt
```

---

## braa

Mass OID enumeration once community string is known:

```bash
# Syntax
braa <community string>@<target>:<OID>

# Full subtree walk
braa public@<target>:.1.3.6.*
```

---

## Nmap

```bash
# UDP scan
sudo nmap -sU -sV -p161,162 <target>

# SNMP NSE scripts
sudo nmap -sU -p161 --script snmp-info,snmp-sysdescr,snmp-win32-users <target>
```
