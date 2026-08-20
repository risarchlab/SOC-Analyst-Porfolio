# 🛡️ SOC Level 1 — Log Analysis Methodology

![Status](https://img.shields.io/badge/Path-TryHackMe%20SOC%20Level%201-blueviolet)
![Focus](https://img.shields.io/badge/Focus-Log%20Analysis%20with%20SIEM-blue)
![Tool](https://img.shields.io/badge/Tool-Splunk-brightgreen)

---

## 📘 Introduction

This repository documents my analytical approach to **SIEM-based log analysis**, developed through the TryHackMe SOC Level 1 path. Rather than reproducing lab answers, these notes capture *how I triage and investigate* across the core log sources a Tier 1 analyst works with daily — Windows, Linux, and web server logs — using **Splunk** as the SIEM.

The goal is to demonstrate analyst thinking: knowing which log source and event to pivot on for a given alert, and why.

---

## 🎯 Objectives

- Investigate common alert types across Windows, Linux, and web logs
- Map alerts to the right log source and event codes
- Build repeatable triage workflows for a Tier 1 SOC queue
- Recognize attacker techniques: persistence, privilege escalation, brute force, web shells

---

## 🧰 Log Sources & Tools

| Source / Tool        | Purpose                                                  |
| -------------------- | -------------------------------------------------------- |
| Splunk               | SIEM platform for searching and correlating logs         |
| Windows Event Logs   | Authentication, account changes, service events          |
| Sysmon               | Deep endpoint telemetry (process, network, registry)     |
| Linux auth.log       | SSH logins, sudo usage, privilege changes                |
| Linux syslog         | Services, cron jobs, system-level events                 |
| Web access logs      | HTTP requests — brute force, web shells, DDoS patterns   |

---

## 🪟 Windows Log Analysis

### Suspicious PowerShell execution
- **Pivot on:** Sysmon Event ID 1 (process creation)
- **Look for:** encoded-command arguments — a common evasion technique
- **Why:** examining parent/child process relationships reveals when a legitimate binary (e.g. `cmd.exe`) is abused to spawn an encoded payload

### Suspicious outbound network connection
- **Pivot on:** Sysmon Event ID 3 (network connection), scoped to the host (Use EventCode=3)
- **Look for:** unusual destination ports and processes running from non-standard paths (e.g. binaries in `Temp`)
- **Next step:** enrich the destination IP against threat-intelligence platforms

### Unauthorized account creation (persistence)
- **Pivot on:** Security Event IDs 4720 (account created) / 4722 (account enabled)
- **Why:** a backup user account created by an already-compromised admin is a classic persistence move — identifying *who* performed the action is key

### Service-based privilege escalation
- **Pivot on:** System Event IDs 7045 (service installed) / 7036 (service state change)
- **Red flag:** a new service launching a binary from `Temp` under the `SYSTEM` account

---

## 🐧 Linux Log Analysis

### Unusual login activity (brute force)
- **Source:** `auth.log`, filtered to `sshd`
- **Look for:** many `Failed password` events followed by an `Accepted password` — the signature of a successful brute force
- **Action:** escalate confirmed brute-force success to Tier 2

### Privilege escalation
- **Source:** `auth.log`, filtered on `su` / sudo activity
- **Why:** tracks a user transitioning to `root`; note that auth.log alone shows *that* it happened, not always *how* — correlate with other sources

### Persistence via cron
- **Source:** `syslog`, filtered for cron entries invoking interpreters (`bash`, `python`, `perl`, `nc`)
- **Red flag:** a script in `/tmp` executing on a short repeating interval, or a reverse-shell one-liner phoning home

---

## 🌐 Web Server Log Analysis

### Brute-force login attempts
- **Look for:** high volume of `POST` requests to a login endpoint (e.g. `/wp-login.php`) from a single client IP in a short window
- **Tell:** the `User-Agent` may reveal an attack tool rather than a browser

### Possible web shell
- **Look for:** `GET`/`POST` requests to script file types (`.php`, `.asp`, `.jsp`) returning `200`, especially oddly-named files generating repeated hits
- **Why:** a successful web shell shows up as a small cluster of successful requests to an unexpected script

### DDoS activity
- **Look for:** a spike of `503` responses and abnormally high request counts per IP over a short interval

---

## 📸 Screenshots

> Images show my investigation *process* — queries and tool navigation — with specific answer values (IPs, hashes, task solutions) cropped or redacted to avoid spoiling the room.

| Description                                              | Screenshot                          |
| ------------------------------------------------------- | ----------------------------------- |
| Pivoting on Sysmon Event ID 3 for network connections                  | <img width="1881" height="794" alt="Screenshot 2026-08-19 180931" src="https://github.com/user-attachments/assets/b3a8b8bd-f507-462f-82bb-d78e58152978" /> |
| Linux brute-force investigation   | <img width="949" height="566" alt="attempted brute force" src="https://github.com/user-attachments/assets/64c1b0d8-952a-4002-a9c4-f4b009f9bd10" />|
| Web log analysis — brute-force POST pattern              | <img width="951" height="365" alt="threat actor tool" src="https://github.com/user-attachments/assets/cc415504-50c2-4e3c-a2ac-d47999eb4c8e" />|

---

## 🧠 Key Takeaways

- 🔍 Learned to map each alert type to the correct log source and event code
- 🪟 Practiced Windows triage with Sysmon and Security/System event logs
- 🐧 Investigated Linux persistence and privilege escalation via auth.log and syslog
- 🌐 Detected web-based attacks — brute force, web shells, DDoS — from access logs
- 🧩 Built repeatable triage workflows suited to a Tier 1 SOC queue

---

## 📎 References

- [TryHackMe — SOC Level 1 Path](https://tryhackme.com/path/outline/soclevel1)
- [Splunk Search Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)
- [Sysmon — Microsoft Learn](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

---

## 📬 About Me

👋 I'm a career-changer building hands-on experience toward a SOC analyst role, backed by a B.S. in Cybersecurity and CompTIA certifications (Security+, CySA+, PenTest+). These notes are part of my self-directed study.

🔗 [LinkedIn](https://www.linkedin.com/in/risaono/) · 🔍 [Return to Projects on GitHub](https://github.com/risarchlab)
