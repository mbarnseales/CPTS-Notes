
# FinalRecon

All-in-one Python web recon tool. Combines header grabbing, WHOIS, SSL info, DNS enumeration, subdomain discovery, crawling, directory search, port scanning, and Wayback Machine queries into a single tool.

See Also: [[00-Methedology/Web-Recon|Web Recon Methodology]]

---

## Installation

```bash
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
./finalrecon.py --help
```

---

## Flags

| Flag | Description |
|------|-------------|
| `--url <URL>` | Target URL |
| `--headers` | HTTP header information |
| `--sslinfo` | SSL certificate details |
| `--whois` | WHOIS lookup |
| `--crawl` | Crawl the target |
| `--dns` | DNS enumeration (40+ record types including DMARC) |
| `--sub` | Subdomain enumeration (crt.sh, VirusTotal, Shodan, and more) |
| `--dir` | Directory brute force |
| `--wayback` | Retrieve URLs from Wayback Machine (last 5 years) |
| `--ps` | Fast port scan |
| `--full` | Run all modules |

### Extra Options

| Flag | Description |
|------|-------------|
| `-w <path>` | Custom wordlist for directory enum |
| `-e <ext>` | File extensions to search (e.g. `txt,php,xml`) |
| `-dt <n>` | Threads for directory enum (default 30) |
| `-pt <n>` | Threads for port scan (default 50) |
| `-T <n>` | Request timeout in seconds (default 30) |
| `-d <DNS>` | Custom DNS server (default 1.1.1.1) |
| `-o <format>` | Output format (default txt) |
| `-cd <path>` | Change export directory |
| `-k <service@key>` | Add API key (e.g. `shodan@<key>`) |

---

## Usage

```bash
# Headers and WHOIS only
./finalrecon.py --headers --whois --url http://<target>

# Headers, WHOIS, SSL, and DNS
./finalrecon.py --headers --whois --sslinfo --dns --url http://<target>

# Subdomain enumeration
./finalrecon.py --sub --url http://<target>

# Crawl + Wayback
./finalrecon.py --crawl --wayback --url http://<target>

# Full recon (all modules)
./finalrecon.py --full --url http://<target>

# Directory search with custom wordlist and extensions
./finalrecon.py --dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -e php,txt,html --url http://<target>
```

Output is saved to `~/.local/share/finalrecon/dumps/` by default.

---

## Other Recon Frameworks

| Tool | Strengths |
|------|----------|
| **Recon-ng** | Modular framework, database-backed, good for large engagements. Modules for DNS, subdomain discovery, port scanning, and more |
| **theHarvester** | Focused on emails, subdomains, hosts, and employee names from public sources (Google, Bing, Shodan, PGP servers) |
| **SpiderFoot** | Automated OSINT across many data sources -- IPs, domains, emails, social media. GUI and CLI |
| **OSINT Framework** | Not a tool -- a curated collection of links to OSINT resources organised by category. `https://osintframework.com` |
