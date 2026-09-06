[← Back to Home](../README.md)

# 01 — Network Enumeration & Footprinting

## Nmap Cheat Sheet

```bash
# Full comprehensive scan (the most important one)
sudo nmap -sS -sV -sC -O -p- <target> -oA fullscan

# Quick scans
sudo nmap -F <target>
sudo nmap --top-ports 100 <target>

# Fast UDP scan
sudo nmap -sU -F <target>

# Ping sweep across a subnet
sudo nmap -sn <network>/24

# ACK scan (firewall rule mapping)
sudo nmap -sA <target>

# Decoy scan
sudo nmap -D RND:5 <target>

# NSE vulnerability scripts
nmap --script vuln <target>
nmap --script smb-os-discovery <target>
```

### Port States You Must Know Cold

| State | Meaning |
|-------|---------|
| `open` | An active connection was established |
| `closed` | Target replied with RST |
| `filtered` | A firewall is dropping/blocking probes |
| `unfiltered` | Reachable via ACK scan, but open/closed status unknown |

## Per-Service Footprinting

### FTP (21)
```bash
ftp <IP>                          # try anonymous / any password
wget -m --no-passive ftp://anonymous:anonymous@IP
nmap -sV -p21 --script ftp-anon,ftp-syst
```

### SMB (445)
```bash
smbclient -N -L //IP
smbmap -H IP
rpcclient -U "" IP
enum4linux-ng.py IP -A
crackmapexec smb IP --shares -u '' -p ''
```

### NFS (2049)
```bash
showmount -e IP
sudo mount -t nfs IP:/mnt/nfs ./target -o nolock
# no_root_squash on an exported share = golden privilege-escalation path
```

### SMTP (25)
```bash
# User enumeration
VRFY user / EXPN user / RCPT TO:user
nmap -p25 --script smtp-open-relay
```

### SNMP (161/UDP)
```bash
snmpwalk -v2c -c public IP
onesixtyone -c snmp.txt IP
nmap -sU -p161 --script snmp*
```

### MySQL (3306)
```bash
mysql -u root -p -h IP
# An empty root password is common in training environments
```

### MSSQL (1433)
```bash
mssqlclient.py user@IP -windows-auth
# Look for xp_cmdshell, xp_dirtree hash stealing (see Service Exploitation Deep-Dive)
```

### RDP (3389)
```bash
xfreerdp /v:IP /u:user /p:pass
nmap -p3389 --script rdp*
```

### WinRM (5985)
```bash
evil-winrm -i IP -u user -p pass
```

### SSH (22)
```bash
ssh-audit.py IP
ssh user@host -o PreferredAuthentications=password
```

## Related Pages

- Web-specific recon → [Web Reconnaissance & Fuzzing](02-Web-Recon-and-Fuzzing.md)
- Full attack chains per service → [Service Exploitation Deep-Dive](03-Service-Exploitation-Deep-Dive.md)
- Port/service quick lookup → [Quick Reference](14-Quick-Reference-Ports-and-Services.md)
