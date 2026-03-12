
# SNMP

Default port: `161`

Versions 1 and 2c use plaintext community strings  -  `public` and `private` are often left as default. Version 3 added encryption and authentication.

Can expose: process parameters, credentials, routing info, software versions, bound services.

## snmpwalk
```bash
# Query with public string
snmpwalk -v 2c -c public <target> 1.3.6.1.2.1.1.5.0

# Query with private string
snmpwalk -v 2c -c private <target>
```

## onesixtyone (brute force community strings)
```bash
onesixtyone -c dict.txt <target>
```
