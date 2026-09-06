[← Back to Home](../README.md)

# 00 — Methodology & Golden Rules

This is the checklist to run through your head on every box, every lab, and every exam question. It's less about memorizing exploits and more about not skipping steps.

## 🎯 The 60-Second Pre-Flight Checklist

```
□ nmap -sV -sC -p- <target>
□ Try FTP anonymous / SMB null session / SNMP public community
□ sudo -l (Linux) or whoami /priv (Windows)
□ find / -perm -4000 2>/dev/null (Linux SUID binaries)
□ netstat -ano / ss -tulpn
□ cat /etc/crontab or schtasks /query
□ searchsploit <service> <version>
□ Always test default credentials first!
```

## 🧠 15 Golden Rules

1. **Start with `nmap -sV -sC -p-`** — map every service before touching anything else.
2. **Default credentials first** — try `admin:admin`, `root:root`, `anonymous:anonymous` on *every* service you find.
3. **Anonymous / guest access is common in training labs** — FTP, SMB, and SNMP are frequently left open.
4. **Found a username? Try it as the password too** — credential reuse is extremely common.
5. **Methodology beats blind exploitation** — certification exams (eJPT/CPTS) test whether you follow a repeatable process, not whether you know a single exploit by heart.
6. **`sudo -l` / `whoami /priv`** — the very first command to run after landing any shell.
7. **`SeImpersonatePrivilege` present?** → jump straight to a Potato-family exploit (Windows).
8. **SUID binaries / writable cron jobs** are usually the fastest path to root on Linux.
9. **`proxychains` only supports TCP connect scans (`-sT`)** — don't forget this when pivoting.
10. **Meterpreter beats a raw SSH/shell session** when you need to pivot — use `autoroute` + `socks_proxy`.
11. **Base64 encode/decode** shows up constantly in exam scenarios — know the one-liners cold.
12. **New internal domain discovered?** Add it to `/etc/hosts` immediately.
13. **Prefer ZAP's fuzzer over Burp Community's Intruder** — Burp's free tier throttles heavily.
14. **`rockyou.txt`** is still the single most useful wordlist: `/usr/share/wordlists/rockyou.txt`.
15. **Impacket is essential for Windows targets** — `secretsdump.py`, `psexec.py`, `wmiexec.py`, `ntlmrelayx.py` should be second nature.

## Related Pages

- Next step after recon → [Network Enumeration & Footprinting](01-Network-Enumeration-and-Footprinting.md)
- Got a shell already? → [Windows Enumeration](09-Windows-Enumeration-and-Situational-Awareness.md) · [Linux PrivEsc](12-Linux-Privilege-Escalation.md)
