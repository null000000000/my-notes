[← Back to Home](../README.md)

# 02 — Web Reconnaissance, Fuzzing & Proxies

## WHOIS & DNS

```bash
whois domain.com
dig domain.com A MX NS TXT ANY
dig axfr @ns1.domain.com domain.com   # Zone transfer = jackpot if it works
host -t MX domain.com
```

## Subdomain Enumeration

```bash
# Passive
curl -s "https://crt.sh/?q=domain.com&output=json" | jq -r '.[].name_value'

# Active
dnsenum --enum domain.com -f wordlist.txt
gobuster vhost -u http://IP -w wordlist.txt --append-domain
ffuf -u http://IP -H "Host: FUZZ.domain.com" -w wordlist.txt
```

## Fingerprinting

```bash
curl -I http://target
whatweb target.com
nikto -h target.com
wafw00f target.com
```

## Directory & File Fuzzing (ffuf)

```bash
# Directories
ffuf -u http://target/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt

# Files with extensions
ffuf -u http://target/FUZZ -w wordlist.txt -e .php,.txt,.html,.bak

# Recursive
ffuf -u http://target/FUZZ -w wordlist -recursion -recursion-depth 1 -e .php

# Virtual host fuzzing
ffuf -u http://target/ -H "Host: FUZZ.target.com" -w wordlist -fs 900

# Parameter fuzzing (GET)
ffuf -u "http://target/page.php?FUZZ=value" -w params.txt -fs xxx

# Parameter fuzzing (POST)
ffuf -u http://target/page.php -X POST -d 'FUZZ=value' -H 'Content-Type: application/x-www-form-urlencoded' -w params.txt -fs xxx
```

## Web Proxies

```bash
# Burp Suite / OWASP ZAP default listener
127.0.0.1:8080

# Route curl traffic through proxychains → Burp
# In /etc/proxychains.conf: http 127.0.0.1 8080
proxychains curl http://target

# Curl directly through a proxy
curl -x http://127.0.0.1:8080 http://target
```

## Related Pages

- Network-level recon → [Network Enumeration & Footprinting](01-Network-Enumeration-and-Footprinting.md)
- Turning findings into exploits → [Service Exploitation Deep-Dive](03-Service-Exploitation-Deep-Dive.md)
- Automated vulnerability scanning → [Vulnerability Assessment](04-Vulnerability-Assessment.md)
