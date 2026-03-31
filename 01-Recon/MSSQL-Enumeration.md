
# MSSQL

Layer 3 enumeration. Microsoft SQL Server is a Windows-native relational database management system. Commonly found on Windows targets in enterprise environments, often integrated with Active Directory via Windows Authentication. High value target -- credentials can open doors into the database, the OS via `xp_cmdshell`, and potentially the domain.

Port: `1433` TCP

> [!tip] Saved Credentials
> SQL Server Management Studio (SSMS) is a client application that can be installed on any machine. If you find SSMS on a compromised host, check for saved credentials -- they may connect directly to a database server elsewhere on the network.

---

## Workflow

1. Nmap on port 1433 with `ms-sql-*` scripts
2. Run Metasploit `mssql_ping` for additional instance info
3. Try default `sa` account with empty password
4. If you have credentials, connect with `mssqlclient.py`
5. Enumerate databases, users, and permissions
6. Check if `xp_cmdshell` is enabled -- OS command execution from SQL
7. Check for Windows Authentication -- compromised domain accounts may have DB access

---

## Nmap

```bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes \
  --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER \
  -sV -p1433 <target>
```

Returns: version, instance name, hostname, named pipe path, DAC port, and attempts empty password check on `sa`.

---

## Metasploit - mssql_ping

Quick instance enumeration without needing credentials:

```bash
msf6 > use auxiliary/scanner/mssql/mssql_ping
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts <target>
msf6 auxiliary(scanner/mssql/mssql_ping) > run
```

Returns: server name, instance name, version, TCP port, and named pipe path.

---

## Connecting - mssqlclient.py

```bash
# Windows Authentication (domain or local account)
python3 mssqlclient.py <domain>/<user>@<target> -windows-auth

# SQL Authentication (sa or other SQL user)
python3 mssqlclient.py <user>@<target>

# With password
python3 mssqlclient.py Administrator@<target> -windows-auth -p 'P4SSw0rd'
```

---

## T-SQL Enumeration Commands

Once connected:

| Command | Description |
|---------|-------------|
| `select name from sys.databases;` | List all databases |
| `use <database>;` | Select a database |
| `select table_name from information_schema.tables;` | List tables in current database |
| `select column_name from information_schema.columns where table_name='<table>';` | List columns |
| `select * from <table>;` | Dump table |
| `select @@version;` | MSSQL version |
| `select system_user;` | Current SQL login |
| `select is_srvrolemember('sysadmin');` | Check if current user is sysadmin (1 = yes) |
| `select name, password_hash from sys.sql_logins;` | SQL login hashes (requires sysadmin) |
| `exec sp_configure;` | Show server configuration options |

---

## Default System Databases

| Database | Contents |
|----------|----------|
| `master` | All system info for the SQL instance -- users, configs, linked servers |
| `model` | Template for all new databases. Changes here apply to every new DB created |
| `msdb` | SQL Server Agent jobs and alerts |
| `tempdb` | Temporary objects. Rebuilt on each restart |
| `resource` | Read-only. System objects bundled with SQL Server |

Start with `master` -- it contains logins, server config, and linked server definitions.

---

## xp_cmdshell

`xp_cmdshell` is a stored procedure that executes OS commands as the SQL Server service account. Disabled by default in modern installs but commonly re-enabled by admins for convenience.

```sql
-- Check if enabled (1 = enabled)
select value from sys.configurations where name = 'xp_cmdshell';

-- Enable it (requires sysadmin)
exec sp_configure 'show advanced options', 1; reconfigure;
exec sp_configure 'xp_cmdshell', 1; reconfigure;

-- Execute a command
exec xp_cmdshell 'whoami';
exec xp_cmdshell 'net user';
```

> [!danger] xp_cmdshell
> If enabled and you have sysadmin, this is direct OS code execution as the service account -- often `NT SERVICE\MSSQLSERVER` or in misconfigured environments, `NT AUTHORITY\SYSTEM`. Treat as full host compromise.

---

## Dangerous Settings

| Setting / Condition | Risk |
|---------------------|------|
| `sa` account enabled with empty or default password | Immediate sysadmin access |
| `xp_cmdshell` enabled | OS command execution from SQL |
| Unencrypted connections | Credentials interceptable on the network |
| Self-signed certificates in use | Certificate spoofing possible |
| Named pipes enabled | Alternative attack path for lateral movement |
| Windows Authentication with over-privileged domain accounts | Compromised AD account may inherit DB sysadmin |
| SSMS installed on non-server hosts with saved credentials | Credential harvesting opportunity |

---

## What to Look For

- **Port 1433 externally accessible** -- same as MySQL, databases should not be publicly exposed. Immediate finding.
- **Empty or default `sa` password** -- Nmap `ms-sql-empty-password` script checks this. Verify manually.
- **sysadmin role on your current user** -- run `select is_srvrolemember('sysadmin');` first after connecting. Determines what you can do.
- **xp_cmdshell enabled** -- check with `sp_configure`. If enabled, escalate to OS immediately.
- **Password hashes in sys.sql_logins** -- crack offline with [[07-Passwords/Hash-Cracking|Hash Cracking]].
- **Windows Authentication in use** -- means domain accounts have DB access. A compromised domain user may already have sysadmin depending on group membership.
- **Linked servers** -- check with `select * from sys.servers;`. Linked servers can be used to pivot SQL access to other database servers on the network.
- **Named pipes enabled** -- noted in Nmap output as `\\<host>\pipe\sql\query`. Relevant for relay and lateral movement paths.
- **MSSQL version** -- older versions (2012, 2014, 2016) have known privilege escalation paths. Note and cross-reference CVEs.
