
# IPMI

Layer 3 enumeration. IPMI (Intelligent Platform Management Interface) is a hardware-level remote management protocol built into most enterprise servers. It runs independently of the OS -- accessible even when the server is powered off. Implemented via a **Baseboard Management Controller (BMC)**, which is essentially a separate embedded computer on the motherboard. Common BMC implementations: **HP iLO**, **Dell iDRAC**, **Supermicro IPMI**.

Port: `623` UDP

> [!danger] Access Impact
> Getting into a BMC is near-equivalent to physical access. You can power cycle, reinstall the OS, access serial console, and read hardware-level data. Treat BMC access as full host compromise.

---

## Workflow

1. Nmap UDP scan on 623 with `ipmi-version` script
2. Try default credentials against web console, SSH, or Telnet
3. If defaults fail -- run Metasploit `ipmi_dumphashes` to exploit the RAKP flaw
4. Crack hashes offline with Hashcat mode 7300
5. Try cracked password against the BMC web console
6. Test for password reuse across other systems in the environment

---

## Nmap

```bash
sudo nmap -sU --script ipmi-version -p 623 <target>
```

Returns: IPMI version, authentication methods supported, and BMC vendor (often visible in MAC address OUI).

---

## Metasploit - Version Discovery

```bash
msf6 > use auxiliary/scanner/ipmi/ipmi_version
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts <target>
msf6 auxiliary(scanner/ipmi/ipmi_version) > run
```

---

## Default Credentials

Try these against the web console, SSH, and Telnet before running any exploits:

| Product | Username | Password |
|---------|----------|----------|
| Dell iDRAC | `root` | `calvin` |
| HP iLO | `Administrator` | randomized 8-char (uppercase + digits) |
| Supermicro IPMI | `ADMIN` | `ADMIN` |

HP iLO's default password is printed on a physical tag on the server. In lab/CTF environments it's often `password` or left at default.

---

## RAKP Hash Extraction (IPMI 2.0 Flaw)

A flaw in the IPMI 2.0 RAKP authentication protocol causes the BMC to send a salted hash of the user's password **before** authentication completes. This means you can request a hash for **any valid username** without knowing the password -- no credentials required.

```bash
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts <target>
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set OUTPUT_HASHCAT_FILE ipmi.txt
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run
```

The module also attempts to auto-crack common passwords against hashes as they're retrieved.

> [!note] No Fix Exists
> The RAKP hash leak is a flaw in the IPMI 2.0 specification itself, not a misconfiguration. The only mitigations are long/complex passwords and network segmentation to restrict BMC access.

---

## Cracking IPMI Hashes - Hashcat

```bash
# Dictionary attack (mode 7300 = IPMI2 RAKP HMAC-SHA1)
hashcat -m 7300 ipmi.txt /usr/share/wordlists/rockyou.txt

# HP iLO factory default mask (8 chars, uppercase + digits)
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

---

## What to Look For

- **623 UDP open on internal network** -- IPMI should never be on a routable network. Presence during an internal pentest is a near-automatic finding.
- **Default credentials work** -- especially `ADMIN/ADMIN` on Supermicro or `root/calvin` on Dell. Immediate BMC access = full host control.
- **RAKP hash retrieval succeeds** -- even without creds, dump hashes for all likely usernames. Crack offline. If the password is unique and reused elsewhere, this can pivot to SSH access across many servers.
- **Password reuse** -- cracked BMC passwords are frequently the same as OS root/Administrator passwords, domain accounts, or network device credentials.
- **BMC web console accessible** -- web UI typically offers virtual KVM, power control, hardware sensor data, and firmware management. High-risk finding on its own.
- **SSH/Telnet on BMC** -- some BMCs expose a CLI on standard ports. Credential spray with found passwords.
- **Older IPMI 1.5** -- MD2/MD5 auth with no encryption. Passive sniffing possible. Note the auth methods returned by Nmap/Metasploit version scan.
