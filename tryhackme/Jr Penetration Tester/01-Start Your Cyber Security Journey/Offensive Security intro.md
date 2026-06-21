# TryHackMe — Offensive Security Intro

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Offensive Security Intro |
| Path | Jr Penetration Tester → Start Your Cyber Security Journey |
| Tasks | Task 1 — Task 4 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Think like a Hacker! |
| Task 2 | Starting the Lab |
| Task 3 | Find Hidden Pages |
| Task 4 | Attack the Admin Page |

---

## What I Learned

My first hands-on offensive exercise — hacking a website legally in a safe lab environment.

**Thinking like a hacker:** An attacker looks beyond what a website shows on the surface and asks what else might be there — hidden pages, unlinked admin panels, files not meant to be public.

**Finding hidden pages:** Used directory enumeration to discover pages not linked anywhere on the visible site. A page can exist on a server without being referenced by any link — the only way to find it is to actively probe for common paths.

```bash
gobuster dir -u http://target.thm -w /usr/share/wordlists/dirb/common.txt
```

**Attacking the admin page:** Once the hidden admin login was found, I tested it for weaknesses — default credentials, common username/password combinations.

---

## Key Takeaway

Security through obscurity (hiding a page instead of properly protecting it) is not real security. If a page exists on the server, it can be found through directory brute forcing — and "hidden" is never a substitute for "authenticated and authorised."

---

## My Progress

- [x] Offensive Security Intro ← *this writeup*
- [x] Defensive Security Intro
- [x] Search Skills

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
