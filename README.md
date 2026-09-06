# 🛡️ Offensive Security Field Notes — eJPT / Penetration Testing Wiki

> A structured, exam-ready knowledge base covering network enumeration, service exploitation, brute forcing, privilege escalation, and post-exploitation — built while preparing for the **eJPT** certification and organized for long-term reference.

[![Made for eJPT](https://img.shields.io/badge/exam-eJPT-blue)]() [![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Status](https://img.shields.io/badge/status-active-brightgreen)]()

---

## ⚠️ Legal & Ethical Use Notice

This repository is an educational reference for **authorized penetration testing, CTFs, lab environments (HTB, TryHackMe, PWK/OSCP-style labs), and certification study (eJPT/CPTS)**. Every technique documented here must only be used against systems you own or are explicitly authorized to test in writing. Unauthorized access to computer systems is illegal in most jurisdictions. The author and contributors accept no liability for misuse.

---

## 📖 How This Wiki Is Organized

Each page below is self-contained but cross-linked to related material — think of it as a personal Notion-style knowledge base ported to Markdown so it renders cleanly on GitHub. Start from **Methodology** if you're new, or jump straight to the topic you need.

| # | Page | What's Inside |
|---|------|----------------|
| 00 | [Methodology & Golden Rules](docs/00-Methodology-and-Golden-Rules.md) | The mental checklist to run on *every* engagement; exam mindset tips |
| 01 | [Network Enumeration & Footprinting](docs/01-Network-Enumeration-and-Footprinting.md) | Nmap mastery, port states, per-service footprinting |
| 02 | [Web Reconnaissance & Fuzzing](docs/02-Web-Recon-and-Fuzzing.md) | WHOIS/DNS, subdomain enum, `ffuf`, proxies |
| 03 | [Service Exploitation Deep-Dive](docs/03-Service-Exploitation-Deep-Dive.md) | SMB, FTP, MSSQL, MySQL, RDP, DNS, Email — theory + attacks + CVEs |
| 04 | [Vulnerability Assessment](docs/04-Vulnerability-Assessment.md) | Nessus, OpenVAS, CVSS scoring |
| 05 | [Brute Force & Credential Attacks](docs/05-Brute-Force-and-Credential-Attacks.md) | Attack theory, Hydra, Medusa, custom wordlists, scripting |
| 06 | [Password Cracking & Hash Attacks](docs/06-Password-Cracking-and-Hash-Attacks.md) | John the Ripper, Hashcat, Pass-the-Hash, Pass-the-Ticket |
| 07 | [Shells, Payloads & Metasploit](docs/07-Shells-Payloads-and-Metasploit.md) | Reverse shells, `msfvenom`, Metasploit workflow |
| 08 | [Pivoting & Tunneling](docs/08-Pivoting-and-Tunneling.md) | SSH pivoting, port forwarding, proxychains |
| 09 | [Windows Enumeration & Situational Awareness](docs/09-Windows-Enumeration-and-Situational-Awareness.md) | Post-shell checklist, network/AV/AppLocker awareness |
| 10 | [Windows Privilege Escalation](docs/10-Windows-Privilege-Escalation.md) | SeImpersonate, weak permissions, UAC bypass, kernel exploits |
| 11 | [Windows Process Communication & Access Tokens](docs/11-Windows-Process-Communication-and-Access-Tokens.md) | Named pipes, access tokens, DACL abuse |
| 12 | [Linux Privilege Escalation](docs/12-Linux-Privilege-Escalation.md) | SUID, sudo abuse, cron/wildcard abuse, kernel exploits |
| 13 | [Post-Exploitation](docs/13-Post-Exploitation.md) | Hash dumping, credential harvesting, pivoting from a compromised host |
| 14 | [Quick Reference — Ports & Services](docs/14-Quick-Reference-Ports-and-Services.md) | One-page cheat table for every major service |

---

## 🗺️ Suggested Reading Paths

- **Studying for eJPT?** → 00 → 01 → 02 → 03 → 05 → 09 → 10 → 12 → 14
- **Mid-engagement, got a shell?** → 09 (Windows) or 12 (Linux) → 10/11 → 13
- **Need a specific service attack right now?** → 03 or 14 → find your service → follow the link

---

## 🤝 Contributing

These are personal study notes cleaned up for public use. Corrections, additional CVEs, or better one-liners are welcome via pull request.

## 📄 License

Released under the [MIT License](LICENSE) — use freely, attribute if useful, hack responsibly.
