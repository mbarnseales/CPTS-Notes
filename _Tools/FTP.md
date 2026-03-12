
# FTP

Default port: `21`

**Always check for anonymous login**  -  a common misconfiguration.

## Commands
```bash
# Connect
ftp -p <target>

# Login as anonymous
Name: anonymous

# Basic navigation
ls
cd <dir>
get <file>
exit
```

## Nmap Check
```bash
nmap -sC -sV -p21 <target>
```
Look for `ftp-anon: Anonymous FTP login allowed` in output.
