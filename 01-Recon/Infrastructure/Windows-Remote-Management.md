
# Windows Remote Management Protocols

Three services covered: **RDP** (port 3389), **WinRM** (ports 5985/5986), and **WMI** (port 135). All three allow remote command execution or GUI access to Windows hosts. Credentials found anywhere in an environment should be tested against all three.

---

## RDP

Port: `3389` TCP/UDP

Provides full GUI access to the remote desktop. Enabled by default on Windows Server. Uses self-signed certificates by default -- expect certificate warnings even on legitimate connections.

**NLA (Network Level Authentication)** -- requires the user to authenticate *before* a desktop session is established. Harder to attack; most modern Windows servers enforce it. If NLA is *not* required, you get a login screen over RDP without pre-authentication.

### Footprinting

```bash
# Nmap with all RDP scripts -- reveals NLA status, hostname, domain, OS version
nmap -sV -sC <target> -p3389 --script rdp*
```

Key output to note:
- `CredSSP (NLA): SUCCESS` -- NLA is enforced
- `Target_Name` / `DNS_Domain_Name` -- hostname and domain (useful for AD recon)
- `Product_Version` -- maps to Windows version (e.g. `10.0.17763` = Server 2019)

> [!warning] OPSEC
> Nmap RDP scripts send the cookie `mstshash=nmap` in the handshake. This is a known EDR/IDS signature. On hardened networks, use `--packet-trace` to verify what's being sent, or skip the script scan and use rdp-sec-check instead.

**rdp-sec-check** -- checks supported security layers without triggering the Nmap cookie:

```bash
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
# Install dependency first: sudo cpan install Encoding::BER
./rdp-sec-check.pl <target>
```

Tells you if the server accepts weak encryption modes or falls back to classic RDP Security (no NLA).

### Connecting

```bash
# Standard connection
xfreerdp /u:<user> /p:<password> /v:<target>

# Pass-the-hash (no plaintext password needed)
xfreerdp /u:<user> /pth:<NTLM hash> /v:<target>

# With domain
xfreerdp /u:<user> /p:<password> /d:<domain> /v:<target>

# Ignore certificate warning
xfreerdp /u:<user> /p:<password> /v:<target> /cert:ignore
```

### What to Look For

- **NLA not enforced** -- login prompt exposed without pre-auth. Lower barrier for password spray/brute force.
- **`Protocol 1` / weak encryption accepted** -- rdp-sec-check flags this. Downgrade attacks possible.
- **Self-signed certificate** -- standard, but worth noting. Combined with no NLA, MITM becomes practical.
- **RDP open externally** -- should be a finding. RDP exposed to the internet is a common ransomware initial access vector.
- **Pass-the-hash** -- if you have an NTLM hash for a local admin, xfreerdp with `/pth` may give you a desktop without ever cracking the password.

---

## WinRM

Ports: `5985` TCP (HTTP), `5986` TCP (HTTPS)

Windows-native remote command execution over HTTP/HTTPS. Enabled by default on Windows Server 2012+. Gives you a PowerShell session on the remote host. `evil-winrm` is the standard pentesting client.

### Footprinting

```bash
nmap -sV -sC <target> -p5985,5986 --disable-arp-ping -n
```

If 5985 is open and returning `Microsoft HTTPAPI httpd 2.0`, WinRM is active.

### Connecting

```bash
# evil-winrm -- interactive PowerShell shell
evil-winrm -i <target> -u <user> -p <password>

# With NTLM hash (pass-the-hash)
evil-winrm -i <target> -u <user> -H <NTLM hash>

# With SSL (port 5986)
evil-winrm -i <target> -u <user> -p <password> -S
```

Once connected, you have a full PowerShell prompt. Run commands, upload/download files, load PowerShell scripts.

```powershell
# Check from Windows if WinRM is reachable (PowerShell)
Test-WsMan <target>
```

### What to Look For

- **5985 open** -- immediately try any credentials you have. WinRM = PowerShell shell.
- **5985 open, 5986 closed** -- no SSL enforced. Credentials transmitted over plain HTTP. MITM opportunity.
- **Admin user access** -- WinRM requires the user to be in the `Remote Management Users` group or be a local admin. Access confirms privilege level.

---

## WMI

Port: `135` TCP (initial negotiation), then random high port for data transfer

WMI provides read/write access to almost all Windows system settings and configurations. Used for remote execution, enumeration, and administration. The Impacket `wmiexec.py` script gives command execution via WMI from Linux.

### Footprinting

```bash
# Port 135 being open is a prerequisite -- standard Windows, almost always open
nmap -sV -p 135 <target>
```

### Remote Execution

```bash
# Execute a single command and return output
wmiexec.py <user>:"<password>"@<target> "<command>"

# Example
wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "whoami"
wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"

# Pass-the-hash
wmiexec.py <user>@<target> -hashes :<NTLM hash>

# Domain account
wmiexec.py <domain>/<user>:"<password>"@<target> "<command>"
```

### What to Look For

- **Valid credentials + port 135 open** -- wmiexec.py gives immediate command execution. No additional service needs to be enabled.
- **Output returned in-band** -- unlike some execution methods, wmiexec.py returns command output directly to your terminal.
- **Firewall may block random high ports** -- if the initial 135 handshake works but commands time out, an intermediate firewall is likely filtering the dynamic port range. Try WinRM instead.
- **Pass-the-hash works here too** -- NTLM hashes from secretsdump or mimikatz can be used directly.
