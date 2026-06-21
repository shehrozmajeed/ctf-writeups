# TryHackMe — Search Skills

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Search Skills |
| Path | Jr Penetration Tester → Start Your Cyber Security Journey |
| Tasks | Task 1 — Task 6 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Shodan (TryScanMe) |
| Task 3 | VirusTotal (TryDetectMe) |
| Task 4 | Vulnerability Databases (CVE) |
| Task 5 | Technical Documentation (MAN) |
| Task 6 | GitHub |

---

## What I Learned

A practical tour of the everyday research tools every pentester relies on.

**Shodan:** Search engine for internet-connected devices — finds what's running on an IP without touching it directly. Already used this in the Passive Reconnaissance room.

**VirusTotal:** Scans files and URLs against dozens of antivirus engines simultaneously. Useful for checking if a downloaded file or suspicious link is known malware.

**CVE Databases:** Once a service version is identified (via Nmap `-sV`), CVE databases (cve.mitre.org, nvd.nist.gov) tell you exactly what known vulnerabilities exist for that version.

```bash
# Workflow: version found → search CVE database
nmap -sV target.com
# → Apache 2.4.29 found
# → search "Apache 2.4.29 CVE" on nvd.nist.gov
```

**Man pages:** Linux's built-in documentation for every command-line tool. Faster than a web search when working offline or wanting authoritative flag syntax.

```bash
man nmap
man hydra
nmap --help    # quick reference alternative
```

**GitHub:** Source of exploit PoCs, security tools, and CVE writeups. Searching `CVE-XXXX-XXXXX exploit` on GitHub often surfaces a working proof-of-concept.

---

## Key Takeaway

This room formalised a workflow I was already doing instinctively — find a service version, then research it. The explicit lesson is to treat research tools (Shodan, CVE databases, GitHub, man pages) as part of the core toolkit, not an afterthought. Knowing where to look is as important as knowing what to look for.

---

## My Progress

- [x] Offensive Security Intro
- [x] Defensive Security Intro
- [x] Search Skills ← *this writeup*

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
