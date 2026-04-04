
# Gobuster

Brute-forces directories, files, DNS subdomains, and vhosts.

## Directory/File Enumeration
```bash
gobuster dir -u http://<target>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

## DNS Subdomain Enumeration
```bash
# Add a DNS server to /etc/resolv.conf first (e.g. 1.1.1.1)
gobuster dns -d <domain> -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

## VHost Fuzzing
```bash
# Basic VHost discovery
gobuster vhost -u http://<target_IP> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain

# With non-standard port
gobuster vhost -u http://<target_IP>:8080 -w <wordlist> --append-domain

# Useful flags
# --append-domain   Appends base domain to each wordlist entry (required in newer versions)
# -t <n>            Threads for faster scanning (default 10)
# -k                Ignore SSL/TLS certificate errors
# -o <file>         Save output to file
```

> [!note] --append-domain
> Required in Gobuster v3.2+. Constructs full hostnames by appending the base domain to each wordlist word (e.g. `word` → `word.example.com`). Older versions handled this automatically.

## HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | OK  -  resource exists and is accessible |
| 301 | Redirect  -  follow it |
| 403 | Forbidden  -  exists but access denied |
| 404 | Not found |

## SecLists
```bash
# Install
sudo apt install seclists -y

# Or clone
git clone https://github.com/danielmiessler/SecLists
```
