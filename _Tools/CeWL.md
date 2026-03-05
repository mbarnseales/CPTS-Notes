
# CeWL

Spiders a target URL and generates a custom wordlist from the words found. Used to build targeted wordlists for password attacks against the same organisation.

---

## Common Usage

```bash
# Basic spider (depth 2 by default)
cewl http://<target>

# Set depth and save to file
cewl -d 3 -w wordlist.txt http://<target>

# Set minimum word length (default is 3)
cewl -m 6 -w wordlist.txt http://<target>

# Include email addresses in output
cewl -e --email_file emails.txt http://<target>

# Include metadata from documents found on the site
cewl -a --meta_file meta.txt http://<target>

# Show word frequency count
cewl -c http://<target>
```

---

## Key Flags

| Flag | Description |
|------|-------------|
| `-d <x>` | Depth to spider (default: 2) |
| `-m <x>` | Minimum word length (default: 3) |
| `-w <file>` | Write wordlist to file |
| `-o` | Allow spidering offsite links |
| `-e` | Extract email addresses |
| `--email_file <file>` | Save emails to separate file |
| `-a` | Extract metadata from documents |
| `--meta_file <file>` | Save metadata to separate file |
| `-c` | Show count for each word |
| `-u <agent>` | Set custom user agent |
| `-n` | Don't output wordlist (use with `-e` or `-a` only) |

---

## Authentication & Proxy

```bash
# Basic or digest auth
cewl --auth_type basic --auth_user <user> --auth_pass <pass> http://<target>

# Through a proxy
cewl --proxy_host 127.0.0.1 --proxy_port 8080 http://<target>
```

---

## Docker

```bash
docker run -it --rm -v "${PWD}:/host" ghcr.io/digininja/cewl [OPTIONS] <url>
```

---

## See Also

- [CeWL GitHub](https://github.com/digininja/CeWL)
