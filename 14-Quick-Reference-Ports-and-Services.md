[← Back to Home](../README.md)

# 14 — Quick Reference: Ports & Services

A single-page lookup table — every major service, its port, and the one detail worth remembering first.

| Service | Port | Key point |
|---------|------|-----------|
| FTP | 21 | Anonymous login |
| SSH | 22 | `ssh-audit`, key-based auth |
| DNS | 53 | Zone transfer (`dig axfr`) |
| SMTP | 25 | `VRFY` user enumeration |
| SMB | 445 | Null session, `enum4linux`, `psexec` |
| NFS | 2049 | `showmount`, `no_root_squash` |
| SNMP | 161/UDP | `public`/`private` community strings, `snmpwalk` |
| MySQL | 3306 | Empty root password |
| MSSQL | 1433 | `xp_cmdshell`, `sa` account |
| Oracle | 1521 | SID brute forcing, `odat.py` |
| IPMI | 623/UDP | Default creds, RAKP hash |
| RDP | 3389 | `xfreerdp`, check NLA |
| WinRM | 5985 | `evil-winrm` |
| HTTP/HTTPS | 80/443 | `ffuf`, `nikto`, `gobuster` |

## Related Pages

- Full footprinting detail for each service → [Network Enumeration & Footprinting](01-Network-Enumeration-and-Footprinting.md)
- Full attack chains → [Service Exploitation Deep-Dive](03-Service-Exploitation-Deep-Dive.md)
- Back to the start → [Methodology & Golden Rules](00-Methodology-and-Golden-Rules.md)
