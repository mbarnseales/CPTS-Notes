
# Metasploit (msfconsole)

Framework for finding and running public exploits. Also includes recon scripts, vulnerability verification, post-exploitation, and pivoting tools.

## Start
```bash
msfconsole
```

## Basic Workflow
```bash
# Search for an exploit
search exploit <term>
search cve:<year> type:exploit   # filtered search

# Load a module
use <module path>

# View required options
show options

# Set options
set RHOSTS <target IP>
set LHOST <your IP or interface e.g. tun0>

# Check if target is vulnerable (if supported)
check

# Run the exploit
exploit
# or
run
```

## Meterpreter (post-exploitation shell)
```bash
getuid          # show current user
shell           # drop into interactive shell
```

## Key Options
| Option | Description |
|--------|-------------|
| `RHOSTS` | Target IP(s), CIDR range, or file |
| `RPORT` | Target port |
| `LHOST` | Your attack host IP or interface |
| `LPORT` | Your listener port |

> Don't rely solely on MSF. Know why exploits fail and when to pivot to manual techniques.
