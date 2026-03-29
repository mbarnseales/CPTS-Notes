
# Hash Cracking

See Also: [[Credential-Hunting]] | [[GTFOBins]]

---

## Identifying Hash Types

Before cracking, identify the hash format. Common tells:

| Hash | Length | Example | Notes |
|------|--------|---------|-------|
| MD5 | 32 chars | `5b5c3ac3a1c897c94caad48e6c71fdec` | No salt  -  instantly crackable with rockyou |
| SHA1 | 40 chars | `da39a3ee5e6b4b0d...` | Common in older apps |
| SHA256 | 64 chars |  -  | Stronger, still crackable offline |
| NTLM | 32 chars |  -  | Windows  -  looks like MD5 but different format |
| bcrypt | 60 chars | `$2a$10$...` | Slow by design  -  expensive to crack |
| SHA512crypt |  -  | `$6$...` | Linux `/etc/shadow` format |

```bash
# hashid can identify hash format
hashid <hash>

# hash-identifier (alternative)
hash-identifier
```

---

## John the Ripper

### Basic Usage

```bash
# Crack with rockyou wordlist
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Specify format explicitly (faster and more accurate)
john hash.txt --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt

# Show cracked passwords
john hash.txt --show

# List supported formats
john --list=formats
```

### Common Format Flags

| Hash Type | John Format Flag |
|-----------|-----------------|
| MD5 (raw) | `--format=raw-md5` |
| SHA1 (raw) | `--format=raw-sha1` |
| SHA256 (raw) | `--format=raw-sha256` |
| NTLM | `--format=NT` |
| bcrypt | `--format=bcrypt` |
| Linux shadow (SHA512) | `--format=sha512crypt` |
| Linux shadow (MD5) | `--format=md5crypt` |

### Preparing Hash Files

```bash
# Single hash
echo "5b5c3ac3a1c897c94caad48e6c71fdec" > hash.txt

# username:hash format (for shadow files)
echo "root:$6$salt$hash..." > hash.txt

# Unshadow (combine /etc/passwd and /etc/shadow)
unshadow /etc/passwd /etc/shadow > hashes.txt
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## Hashcat

```bash
# Basic syntax
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt

# Common modes
# -m 0    = MD5
# -m 100  = SHA1
# -m 1000 = NTLM
# -m 1800 = SHA512crypt (Linux shadow)
# -m 3200 = bcrypt

# Example  -  MD5
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# Show cracked
hashcat -m 0 hash.txt --show
```

---

## Wordlists

| Wordlist | Path | Use |
|----------|------|-----|
| rockyou | `/usr/share/wordlists/rockyou.txt` | Default first attempt |
| SecLists | `/usr/share/seclists/` | Broad coverage across many categories |

```bash
# Decompress rockyou if needed
gunzip /usr/share/wordlists/rockyou.txt.gz
```

---

## Seen In

| Box | Hash Type | Tool | Notes |
|-----|-----------|------|-------|
| [[Conversor]] | MD5 (unsalted) | John | Extracted from SQLite DB with `strings`  -  see [[Credential-Hunting]] |
| [[CCTV]] | bcrypt (`$2y$`) | Hashcat `-m 3200` | Dumped from ZoneMinder DB via SQLi  -  only `mark`'s hash cracked (rockyou) |
