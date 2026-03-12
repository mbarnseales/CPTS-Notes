
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
