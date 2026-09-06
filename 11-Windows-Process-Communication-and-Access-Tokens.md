[← Back to Home](../README.md)

# 11 — Windows Process Communication & Access Tokens

This is one of the most important parts of Windows privilege escalation in eJPT/CPTS, because the question is rarely "run this pre-built exploit" — it's **enumerate → find a weak process/service → exploit the misconfiguration.**

## 1. The Core Idea

During Windows privilege escalation, don't only look at users, services, and scheduled tasks — also look at **processes that communicate with each other**.

> A low-privilege process may communicate with a high-privilege process in an insecure way → leading to privilege escalation.

Examples:
- A web server running as SYSTEM.
- A service holding a sensitive token.
- A named pipe with weak permissions.

## 2. Access Tokens

An **Access Token** is the security context that defines:
- User identity
- Privileges
- Group memberships
- Permissions

Every process in Windows owns an access token. Example:
```
Process: Tomcat.exe
Running User: SYSTEM
Token: SYSTEM privileges
```

If you can control or improperly interact with this process:
```
Low-Priv User → Vulnerable Process → SYSTEM
```

## 3. Enumerating Network Services

Goal: find services running on ports that can be exploited.

```cmd
netstat -ano
```
Shows port, state, and PID. Example: `TCP 0.0.0.0:8080 LISTENING 2276` → port `8080`, PID `2276`.

## 4. Mapping a Port to a Process

```cmd
tasklist /FI "PID eq 2276"
```
Example result: `Tomcat8.exe`, PID `2276`.

```
Port → PID → Process
```

## 5. What to Look For in `netstat`

**Localhost-only services** — anything bound to `127.0.0.1` or `::1`:
```
TCP 127.0.0.1:14147 LISTENING
```
These are assumed safe by developers because they're not exposed to the network — but once you're on the box, you can reach them too.

**Worked example — FileZilla admin interface** on `127.0.0.1:14147`. This administrative interface may allow you to:
- Extract stored FTP passwords
- Create new FTP shares
- Execute commands with elevated privileges

**Interesting ports to check:**

| Port | Service |
|------|---------|
| 21 | FTP |
| 80 | HTTP |
| 135 | RPC |
| 139 | NetBIOS |
| 445 | SMB |
| 1433 | MSSQL |
| 3389 | RDP |
| 5985 | WinRM |
| 8080 | Web apps |

## 6. Named Pipes

A named pipe is a communication channel between processes, formatted as:
```
\\.\pipe\PipeName
```
Example: `\\.\pipe\SQLLocal\SQLEXPRESS01` — used by SQL Server, other services, and security software.

**Why they matter for privilege escalation:**
```
High-Privilege Process
        |
    Named Pipe
        |
Low-Privilege User
```
If the low-privilege user can **write to** or **modify permissions on** the pipe, they may be able to take control of the high-privilege process behind it.

## 7. Enumerating Named Pipes

```powershell
gci \\.\pipe\
# or
Get-ChildItem \\.\pipe\
```
Example output:
```
SQLLocal\SQLEXPRESS01
MSSQL$SQLEXPRESS01\sql\query
WindscribeService
```

**Sysinternals PipeList:**
```cmd
pipelist.exe /accepteula
```
Shows pipe name, current instances, and max instances.

## 8. Checking Pipe Permissions

```cmd
accesschk.exe -accepteula -v \\.\pipe\PipeName
```
Example:
```cmd
accesschk.exe -accepteula -v \\.\pipe\WindscribeService
```

**Key permissions to look for:**

| Permission | Meaning |
|------------|---------|
| `FILE_WRITE_DATA` | Can write data to the pipe |
| `FILE_ALL_ACCESS` | Full control |
| `WRITE_DAC` | Can modify the object's permissions (very significant) |

**Why `WRITE_DAC` matters:** before, `Administrator: Full Access` / `User: Read`. With `WRITE_DAC`, the user can rewrite the ACL itself and grant themselves full access.

## 9. DACL (Discretionary Access Control List)

The DACL defines who has what permissions on an object. Example:
```
Pipe Owner: NT SERVICE\MSSQL$SQLEXPRESS01
Permissions:
  Everyone: Read
  Admin: Full Access
```

**The owner matters:** whoever owns the object can modify its DACL — so `Owner ≈ WRITE_DAC capability`.

## 10. Hunting for Vulnerable Pipes

```cmd
accesschk.exe -w \pipe\* -v
```
Focus on principals like `Everyone`, `Authenticated Users`, `Users` combined with `WRITE`, `FILE_ALL_ACCESS`, or `WRITE_DAC`.

## 11. Full Attack Workflow

```
Step 1 — Enumerate processes:      netstat -ano
Step 2 — Map PID to process:       tasklist /FI "PID eq <PID>"
Step 3 — Check localhost services: 127.0.0.1:<PORT>
Step 4 — Enumerate named pipes:    gci \\.\pipe\
Step 5 — Check permissions:        accesschk.exe -accepteula -v \\.\pipe\<name>
Step 6 — Look for:                 WRITE, WRITE_DAC, FILE_ALL_ACCESS, Everyone
```

## 12. Worked Lab Example

Pipe: `\\.\pipe\SQLLocal\SQLEXPRESS01`
```powershell
accesschk.exe -accepteula -l \\.\pipe\SQLLocal\SQLEXPRESS01
```
Output:
```
OWNER: NT SERVICE\MSSQL$SQLEXPRESS01
```
Conclusion: the owner `NT SERVICE\MSSQL$SQLEXPRESS01` can modify the DACL, and therefore effectively has `WRITE_DAC`.

## Quick Exam Checklist

```
[ ] Run netstat -ano
[ ] Find localhost-only services
[ ] Map PID → process
[ ] Check the process's running user
[ ] Enumerate named pipes
[ ] Find weak permissions
[ ] Look for: WRITE, WRITE_DAC, FILE_ALL_ACCESS
[ ] Identify the account behind it
```

These notes are worth reviewing before CPTS/eJPT because they connect **enumeration → analysis → exploitation path**, rather than just memorizing isolated commands.

## Related Pages

- Broader Windows PrivEsc context → [Windows Privilege Escalation](10-Windows-Privilege-Escalation.md)
- Situational awareness before you get here → [Windows Enumeration & Situational Awareness](09-Windows-Enumeration-and-Situational-Awareness.md)
