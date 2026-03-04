
# Searchsploit

Local CLI tool for searching the Exploit DB. Useful for offline exploit research during engagements.

## Install
```bash
sudo apt install exploitdb -y
```

## Usage
```bash
# Search by app name and version
searchsploit <application> <version>

# Example
searchsploit openssh 7.2
```

Output shows exploit title and file path. Cross-reference with online sources:
- [Exploit DB](https://www.exploit-db.com/)
- [Rapid7 DB](https://www.rapid7.com/db/)
- [Vulnerability Lab](https://www.vulnerability-lab.com/)
