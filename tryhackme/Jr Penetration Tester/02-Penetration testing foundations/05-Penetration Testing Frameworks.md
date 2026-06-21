# TryHackMe — Penetration Testing Frameworks

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Pentest%20Frameworks-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Penetration Testing Frameworks |
| URL | tryhackme.com/room/penetrationtestingframeworks |
| Path | Jr Penetration Tester → Penetration Testing Foundations |
| Tasks Completed | Task 1 — Task 10 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | OSSTMM | ✅ Done |
| Task 3 | OWASP WSTG | ✅ Done |
| Task 4 | NIST SP 800-115 | ✅ Done |
| Task 5 | PTES | ✅ Done |
| Task 6 | ISSAF | ✅ Done |
| Task 7 | MITRE ATT&CK | ✅ Done |
| Task 8 | Other Notable Frameworks | ✅ Done |
| Task 9 | Choosing the Right Framework | ✅ Done |
| Task 10 | Conclusion | ✅ Done |

---

## Why Frameworks Matter

A framework is a structured, repeatable methodology for conducting a penetration test. Without one, tests are inconsistent, results are incomparable, and reports miss critical areas.

**Key benefits:**
- Ensures nothing is missed
- Provides a common language between pentesters and clients
- Makes engagements legally defensible
- Enables comparison of results across different tests over time

---

## Task 2 — OSSTMM (Open Source Security Testing Methodology Manual)

**Created by:** ISECOM  
**Focus:** Scientific, measurable approach to security testing  
**Website:** isecom.org/OSSTMM.3.pdf

### What makes OSSTMM unique

OSSTMM does not just test technical systems — it tests the full security posture across multiple channels.

**5 Channels OSSTMM covers:**

| Channel | What it tests |
|---------|---------------|
| **Human** | Social engineering, phishing, physical access |
| **Physical** | Locks, access cards, cameras, guards |
| **Wireless** | WiFi, Bluetooth, RFID |
| **Telecommunications** | Phone systems, VoIP |
| **Data Networks** | Traditional network and application testing |

### Key concept: RAV (Risk Assessment Value)

OSSTMM produces a quantitative security score called RAV — not just a list of vulnerabilities. This makes it useful for comparing security posture over time.

**Best used for:** Comprehensive, scientific engagements where the client wants a measurable security score, not just a vulnerability list.

---

## Task 3 — OWASP WSTG (Web Security Testing Guide)

**Created by:** OWASP (Open Web Application Security Project)  
**Focus:** Web application security testing  
**Website:** owasp.org/www-project-web-security-testing-guide/  
**Current version:** WSTG v4.2

### Why OWASP WSTG is essential

OWASP WSTG is the most widely referenced web application testing guide in the industry. It is free, maintained by the community, and maps directly to the OWASP Top 10.

### Structure — 12 testing categories

| Category | Code | What it covers |
|----------|------|----------------|
| Information Gathering | OTG-INFO | Recon, fingerprinting, Google dorking |
| Configuration Testing | OTG-CONF | Server config, cloud storage, HTTP methods |
| Identity Management | OTG-IDENT | Account enumeration, username policy |
| Authentication | OTG-AUTHN | Login bypass, brute force, MFA testing |
| Authorization | OTG-AUTHZ | IDOR, privilege escalation, path traversal |
| Session Management | OTG-SESS | Cookie security, CSRF, session fixation |
| Input Validation | OTG-INPVAL | SQLi, XSS, XXE, SSRF, command injection |
| Error Handling | OTG-ERR | Error messages, stack traces |
| Cryptography | OTG-CRYPT | Weak ciphers, SSL/TLS testing |
| Business Logic | OTG-BUSLOGIC | Logic flaws, workflow bypass |
| Client-Side | OTG-CLIENT | DOM XSS, HTML injection, clickjacking |
| API Testing | OTG-API | REST/GraphQL security, auth, rate limiting |

**Best used for:** Any web application or API penetration test. This guide is the standard reference for bug bounty hunters and professional web pentesters.

**My use:** OWASP WSTG directly maps to my PortSwigger Web Security Academy labs. I use it as the checklist for what to test on every web target.

---

## Task 4 — NIST SP 800-115

**Created by:** NIST (National Institute of Standards and Technology)  
**Focus:** Government and compliance-driven engagements  
**Website:** nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf

### NIST SP 800-115 Phases

| Phase | Activities |
|-------|-----------|
| **Planning** | Define objectives, scope, rules of engagement, legal review |
| **Discovery** | Passive and active reconnaissance, vulnerability scanning |
| **Attack** | Exploitation, privilege escalation, lateral movement |
| **Reporting** | Document findings, risk ratings, remediation recommendations |

### Who uses NIST SP 800-115

- US government agencies (mandatory for federal systems)
- Companies that must comply with FISMA
- Financial institutions under regulatory scrutiny
- Healthcare organisations (HIPAA-adjacent compliance)

**Best used for:** Compliance-driven engagements where the client needs to demonstrate security testing to a regulator. Less practical than PTES for offensive testing.

**Pakistan relevance:** SBP (State Bank of Pakistan) cybersecurity frameworks reference NIST standards. Knowledge of NIST is directly useful for banking sector roles.

---

## Task 5 — PTES (Penetration Testing Execution Standard)

**Created by:** Community of pentesters  
**Focus:** Practical, comprehensive offensive testing  
**Website:** pentest-standard.org

### PTES Phases

| Phase | What happens |
|-------|-------------|
| **Pre-engagement** | Scope, ROE, contracts, legal authorisation |
| **Intelligence Gathering** | Recon — OSINT, active scanning |
| **Threat Modelling** | Identify what assets are at risk and from whom |
| **Vulnerability Analysis** | Identify and validate weaknesses |
| **Exploitation** | Active attack — gain access |
| **Post-Exploitation** | What can be done with access — pivot, escalate, exfiltrate |
| **Reporting** | Document everything with evidence and remediation |

**Best used for:** Full-scope penetration tests where the pentester has significant freedom. PTES is the most practically useful framework for professional offensive security work.

**Why I focus on PTES:** The Jr Penetration Tester path on TryHackMe and my PortSwigger labs follow the PTES structure — recon, vuln analysis, exploitation, post-exploitation, reporting.

---

## Task 6 — ISSAF (Information Systems Security Assessment Framework)

**Created by:** OISSG  
**Focus:** Detailed, phase-by-phase assessment methodology

ISSAF is highly detailed — it breaks each phase into specific techniques and tools. Less commonly referenced today but useful as a technical checklist.

**Phases:** Planning → Assessment → Reporting → Clean Up → Destroy Evidence of Test

**Best used for:** Reference for specific techniques within a phase. Not typically used as a standalone engagement framework today.

---

## Task 7 — MITRE ATT&CK

**Created by:** MITRE Corporation  
**Focus:** Real-world attacker tactics, techniques, and procedures (TTPs)  
**Website:** attack.mitre.org

### What makes MITRE ATT&CK different

Unlike other frameworks, ATT&CK is built from real-world attack observations — every technique in it has been used by actual threat actors in real incidents.

### Structure

```
Tactics (What the attacker wants to achieve)
    └── Techniques (How they achieve it)
            └── Sub-techniques (Specific implementation)
                        └── Procedures (Real-world examples from threat groups)
```

### 14 Tactics in ATT&CK Enterprise

| # | Tactic | Goal |
|---|--------|------|
| TA0043 | Reconnaissance | Gather target information |
| TA0042 | Resource Development | Build attack infrastructure |
| TA0001 | Initial Access | Get into the target |
| TA0002 | Execution | Run malicious code |
| TA0003 | Persistence | Maintain foothold |
| TA0004 | Privilege Escalation | Gain higher permissions |
| TA0005 | Defense Evasion | Avoid detection |
| TA0006 | Credential Access | Steal credentials |
| TA0007 | Discovery | Explore the environment |
| TA0008 | Lateral Movement | Move through the network |
| TA0009 | Collection | Gather data to exfiltrate |
| TA0011 | Command and Control | Communicate with compromised systems |
| TA0010 | Exfiltration | Steal data out |
| TA0040 | Impact | Destroy, disrupt, or encrypt |

### Example ATT&CK entry

```
Tactic:        Persistence (TA0003)
Technique:     Boot or Logon Autostart Execution (T1547)
Sub-technique: Registry Run Keys (T1547.001)
Procedure:     APT29 used this technique in the SolarWinds attack
Detection:     Monitor HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

**Best used for:** Threat intelligence, SOC detection engineering, red team exercise planning, mapping defences to known threat actor behaviours.

**Why ATT&CK matters for SOC roles:** When a SIEM alert fires, SOC analysts map the behaviour to an ATT&CK technique. This tells them what the attacker is trying to do and what the next likely step is. ATT&CK knowledge is expected in SOC L2+ interviews.

---

## Task 9 — Choosing the Right Framework

| Scenario | Best Framework |
|----------|---------------|
| Web application pentest | OWASP WSTG |
| Full network + physical + social pentest | OSSTMM |
| Government / compliance-driven test | NIST SP 800-115 |
| Practical offensive engagement | PTES |
| SOC detection engineering | MITRE ATT&CK |
| Threat intelligence / red team planning | MITRE ATT&CK |
| Bug bounty hunting | OWASP WSTG |

**In practice:** Professional pentesters combine frameworks. Start with PTES for structure, use OWASP WSTG as the web testing checklist, and map findings to MITRE ATT&CK for the report.

---

## Key Takeaways

- **No single framework covers everything** — professionals combine them based on engagement type and client needs
- **OWASP WSTG is the most practical for my current focus** — it is the direct reference for everything I am doing in PortSwigger labs and bug bounty
- **MITRE ATT&CK is the language of SOC teams** — knowing it makes you immediately useful in a SOC analyst role
- **PTES is how real pentests are structured** — pre-engagement through reporting, every phase has clear deliverables
- **NIST matters in Pakistan's banking sector** — SBP references NIST frameworks; knowing this signals awareness of the local regulatory environment

---

## Resources

- [OWASP WSTG (free)](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK (free)](https://attack.mitre.org/)
- [PTES (free)](http://www.pentest-standard.org/)
- [NIST SP 800-115 (free PDF)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf)
- [OSSTMM (free)](https://www.isecom.org/OSSTMM.3.pdf)

---

## My Progress

- [x] Dive Into Pentesting
- [x] Cyber Kill Chain
- [x] Penetration Testing Frameworks ← *this writeup*
- [ ] Network Services rooms — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
