
# cURL

## Banner Grabbing / Server Headers
```bash
# Fetch headers only (follows redirects)
curl -IL https://<target>
```

Headers can reveal: server software, framework, PHP version, auth options, misconfigurations.
