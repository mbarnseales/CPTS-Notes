
# SMB

Default port: `445`

Common attack surface on Windows. Check for sensitive data in shares, credential exposure, and RCE (e.g. EternalBlue).

## smbclient
```bash
# List shares (no password)
smbclient -N -L \\\\<target>

# Connect to share (guest)
smbclient \\\\<target>\\<share>

# Connect with credentials
smbclient -U <user> \\\\<target>\\<share>

# Basic navigation
ls
cd <dir>
get <file>
exit
```

## Nmap
```bash
# OS discovery via SMB
nmap --script smb-os-discovery.nse -p445 <target>

# Aggressive scan
nmap -A -p445 <target>
```
