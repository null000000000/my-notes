[← Back to Home](../README.md)

# 09 — Windows Enumeration & Situational Awareness

The goal of this stage: after landing a Windows shell, work through this checklist **in order** before attempting any privilege escalation.

```
Current Situation
        |
        ↓
Understand Host
        |
        ↓
Identify Network
        |
        ↓
Identify Protections
        |
        ↓
Choose PrivEsc Path
```

You're looking for: other reachable networks, other hosts on the network, domain information, AV/EDR presence, AppLocker restrictions, and which tools you're actually able to run.

## 1) User Enumeration

**Who am I?**
```cmd
whoami
```
Example output: `WIN01\john` → machine `WIN01`, user `john`.

**Full identity details**
```cmd
whoami /all
```
Shows SID, groups, privileges, and token information.

## 2) User Privileges

```cmd
whoami /priv
```

Focus on these privileges specifically:
```
SeImpersonatePrivilege
SeAssignPrimaryTokenPrivilege
SeDebugPrivilege
SeBackupPrivilege
SeRestorePrivilege
```

| Privilege | Use |
|-----------|-----|
| `SeImpersonatePrivilege` | Token impersonation |
| `SeDebugPrivilege` | Control over other processes |
| `SeBackupPrivilege` | Read protected files |
| `SeRestorePrivilege` | Write to protected files |

## 3) Group Membership

```cmd
whoami /groups
```

Look for: `BUILTIN\Administrators`, `Domain Admins`, `Backup Operators`, `Remote Desktop Users`.

**Enumerate members of a specific local group:**
```cmd
net localgroup "Group Name"
```
Examples:
```cmd
net localgroup administrators
net localgroup "Backup Operators"
net localgroup "Remote Desktop Users"
```

## 4) System Enumeration

```cmd
systeminfo
```
Reveals OS version, build number, hotfixes, domain, and architecture. Use the build number to search for a matching privilege-escalation exploit, e.g. "Windows Server 2016 privilege escalation exploit".

```cmd
hostname
wmic os get Caption,Version,BuildNumber
```

## 5) Architecture

```cmd
echo %PROCESSOR_ARCHITECTURE%
```
`AMD64` means 64-bit.

## 6) Patch Enumeration

```cmd
wmic qfe
```
or
```powershell
Get-HotFix
```
Note any `KBxxxxxxx` identifiers and search for `KBxxxxxxx exploit`.

## 7) Running Processes

```cmd
tasklist
tasklist /svc
```
For each interesting process, ask: is it an outdated version? Does it run as SYSTEM? Does its config file contain credentials?

## 8) Services

```cmd
sc query
net start
```
```powershell
Get-Service
```
**Details on a specific service:**
```cmd
sc qc ServiceName
```
Check `BINARY_PATH_NAME` and `SERVICE_START_NAME` — a `SERVICE_START_NAME` of `NT AUTHORITY\SYSTEM` is significant.

## 9) Network Enumeration

```cmd
netstat -ano
```
Example: `0.0.0.0:21 LISTENING PID 1234` → FTP is running under PID 1234. Identify the process:
```cmd
tasklist /FI "PID eq 1234"
```

**Find the service listening on a specific port:**
```cmd
netstat -ano | findstr :PORT
tasklist /svc /FI "PID eq PID"
```
The service name (not the executable name) is what you're after — e.g. `Tomcat9`, not `java.exe`.

**PowerShell alternative:**
```powershell
Get-NetTCPConnection -LocalPort PORT
Get-CimInstance Win32_Service | Where-Object {$_.ProcessId -eq PID}
```

**Network interfaces**
```cmd
ipconfig /all
```
Look for more than one network adapter and note internal IP ranges.

## 10) Environment Variables

```cmd
set
echo %PATH%
```
If a folder in `PATH` is user-writable, that's a potential DLL hijacking vector.

```cmd
echo %USERPROFILE%
```
Check `Desktop`, `Documents`, `Downloads`, and `AppData` under the user's profile.

## 11) Installed Software

```cmd
wmic product get name
```
```powershell
Get-WmiObject -Class Win32_Product
```
Watch for: FileZilla, PuTTY, WinSCP, SQL Server, Java.

## 12) Logged-In Users

```cmd
query user
quser
```

## 13) User Accounts

```cmd
net user
net user <username>
```
The per-user detail view shows group membership, last login, and password info.

## 14) Local Groups

```cmd
net localgroup
net localgroup administrators
```

## 15) Password Policy

```cmd
net accounts
```
Shows minimum password length, lockout policy, and expiration settings.

## 16) Scheduled Tasks

Very important for privilege escalation.
```cmd
schtasks /query
schtasks /query /fo LIST /v
```
Look for tasks running as SYSTEM with a script you can modify.

## 17) Credential Hunting

```cmd
findstr /si password *.txt
dir /s *pass*
```
Check high-value locations:
```
C:\Users\*\Desktop
C:\Users\*\Documents
C:\ProgramData
C:\Windows\Temp
```

## 18) File Permissions

```cmd
icacls filename
```
`Everyone:(F)` means anyone can modify the file.

## 19) Drive Enumeration

```cmd
wmic logicaldisk get name
fsutil fsinfo drives
```

## 20) Registry Enumeration

```cmd
reg query HKLM
reg query HKLM\Software
```

## 21) Startup Programs

```cmd
dir "C:\Users\%username%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
```

## 22) PowerShell Version

```powershell
$PSVersionTable
```

---

## Network & Situational Awareness (Beyond the Local Host)

Before running heavier tools (winPEAS, Seatbelt, LaZagne, PowerUp), build a picture of the wider environment.

### Network Interfaces, IP & DNS

```cmd
ipconfig /all
```

| Item | Why it matters |
|------|-----------------|
| Hostname | The machine's name |
| IPv4 | The host's address |
| DNS servers | Network context |
| Default gateway | The network's gateway |
| DNS suffix | Indicates domain membership |
| Multiple interfaces | Possible dual-homed host |

**Dual-homed host example:**
```
Ethernet0 → 10.129.43.8
Ethernet1 → 192.168.20.56
```
This machine is connected to two networks — after compromising it, you may be able to pivot into the second one.

### ARP Enumeration

```cmd
arp -a
```
Shows recently-contacted hosts:
```
10.129.43.12    xx-xx-xx
10.129.43.13    xx-xx-xx
```
After obtaining credentials, these hosts might be admin workstations, domain controllers, or other servers reachable via RDP, WinRM, or SMB.

### Routing Table

```cmd
route print
```
Reveals which networks the host can reach. Pay special attention to the default route (`0.0.0.0`, with a gateway like `10.129.0.1`).

**Overall network-awareness flow:**
```
ipconfig /all → find interfaces → arp -a → find known hosts → route print → find reachable networks
```

### Windows Defender Status

```powershell
Get-MpComputerStatus
```
Key fields:
- `AntivirusEnabled: True` — Defender is active.
- `RealTimeProtectionEnabled: True` — real-time protection is on.
- `BehaviorMonitorEnabled` — behavioral monitoring is active.

### AppLocker Enumeration

AppLocker is application whitelisting — it blocks users from running certain executables.

```powershell
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```
Shows allowed/denied rules, paths, and affected users. Example: `PathConditions: {%WINDIR%\*}` with `Action: Allow` means everything under `C:\Windows\` is permitted.

**Save the policy as XML:**
```powershell
Get-AppLockerPolicy -Effective -Xml > AppLockerPolicy.xml
```

**Test whether a specific binary is allowed:**
```powershell
Test-AppLockerPolicy -XmlPolicy .\AppLockerPolicy.xml -Path C:\Path\file.exe -User Everyone
```
Example — testing `cmd.exe` → `Denied`; testing `powershell.exe` → `Allowed`.

| Output | Meaning |
|--------|---------|
| Allowed | The file can run |
| Denied | The file is blocked |
| PathConditions | Where the rule applies |
| UserOrGroupSid | Who the rule applies to |
| Action | Allow / Deny |

Common paths: `%WINDIR%\*` (`C:\Windows\*`), `%PROGRAMFILES%\*` (`C:\Program Files\*`).

## Quick Enumeration Checklist

```
[ ] whoami / whoami /priv / whoami /groups
[ ] hostname / systeminfo / ipconfig /all
[ ] tasklist /svc / netstat -ano
[ ] net user / net localgroup administrators
[ ] wmic product get name
[ ] schtasks /query
[ ] Get-Service
[ ] search for stored passwords
[ ] check writable files/paths
```

## The CPTS/eJPT Mental Model

```
1. Who am I?
2. What network am I in?
3. What other networks exist?
4. What protections exist?
5. What tools can I run?
6. Start privilege-escalation enumeration
```

This stage is called **Situational Awareness** because it stops you from acting randomly — you build a complete picture of the system before committing to a privilege-escalation path.

## Quick Command Reference

| Command | Shell | Use |
|---------|-------|-----|
| `ipconfig /all` | CMD | Network interfaces, IP, DNS |
| `arp -a` | CMD | ARP cache & known hosts |
| `route print` | CMD | Routing table & reachable networks |
| `systeminfo` | CMD | OS information & patches |
| `Get-MpComputerStatus` | PowerShell | Windows Defender status |
| `Get-AppLockerPolicy` | PowerShell | AppLocker rules |
| `Get-Service` | PowerShell | Service status |

## The Top 5 Things to Check Fast (CPTS/eJPT)

```
1. SeImpersonatePrivilege
2. User in the Administrators group
3. Weak service permissions
4. Stored credentials
5. Scheduled task misconfiguration
```

Memorize this flow — in most HTB/CPTS labs, roughly 70% of the enumeration follows this exact order.

## Related Pages

- Turn these findings into root/SYSTEM → [Windows Privilege Escalation](10-Windows-Privilege-Escalation.md)
- Process/pipe-based escalation → [Windows Process Communication & Access Tokens](11-Windows-Process-Communication-and-Access-Tokens.md)
- Same checklist for Linux → [Linux Privilege Escalation](12-Linux-Privilege-Escalation.md)
