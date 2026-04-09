# Protected File Transfers

> Always encrypt sensitive data before transfer — NTDS.dit, credential files, recon output, etc.
> Never exfiltrate real PII/financial/trade secret data. Use dummy data for DLP testing.

---

## Windows — AES Encryption (PowerShell)

Uses `Invoke-AESEncryption.ps1` — AES-256-CBC, outputs `.aes` file.

```powershell
# Import the module (transfer the script first)
Import-Module .\Invoke-AESEncryption.ps1

# Encrypt a file
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
# Output: scan-results.txt.aes

# Decrypt a file
Invoke-AESEncryption -Mode Decrypt -Key "p4ssw0rd" -Path .\scan-results.txt.aes

# Encrypt a string
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Text "Secret Text"

# Decrypt a string
Invoke-AESEncryption -Mode Decrypt -Key "p4ssw0rd" -Text "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs="
```

> Use a **unique strong password per engagement** — a single leaked/cracked password shouldn't expose all client data.

---

## Linux — OpenSSL Encryption

```bash
# Encrypt
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc

# Decrypt
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```

| Flag | Purpose |
|---|---|
| `-aes256` | AES-256-CBC cipher |
| `-iter 100000` | 100k iterations — hardens against brute-force |
| `-pbkdf2` | PBKDF2 key derivation — recommended over legacy default |
| `-d` | Decrypt mode |

---

## Transfer Method Priority

Prefer encrypted transport when available:

1. HTTPS
2. SFTP / SCP (SSH)
3. Encrypted payload over plain HTTP (e.g., OpenSSL-encrypted file via wget/curl)
