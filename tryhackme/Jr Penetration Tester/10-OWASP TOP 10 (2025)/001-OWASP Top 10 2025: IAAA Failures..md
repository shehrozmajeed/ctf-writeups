# TryHackMe — OWASP Top 10 2025: IAAA Failures

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-OWASP%20Top%2010-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | OWASP Top 10 2025: IAAA Failures |
| URL | tryhackme.com/room/owasptopten2025one |
| Tasks Completed | Task 1 — Task 6 (all, 100%) |
| Focus | A01: Broken Access Control, A07: Authentication Failures, A09: Logging & Alerting Failures |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | What is IAAA? | ✅ Done |
| Task 3 | A01: Broken Access Control | ✅ Done |
| Task 4 | A07: Authentication Failures | ✅ Done |
| Task 5 | A09: Logging & Alerting Failures | ✅ Done |
| Task 6 | Conclusion | ✅ Done |

---

## Task 2 — What is IAAA?

IAAA is the security model that governs how systems control who can do what.

| Letter | Concept | What it means |
|--------|---------|---------------|
| **I** | Identification | Who are you? (username, email) |
| **A** | Authentication | Prove it (password, MFA, biometric) |
| **A** | Authorisation | What are you allowed to do? (roles, permissions) |
| **A** | Accountability | What did you do? (logs, audit trail) |

**Why IAAA matters for pentesting:**
Every access control vulnerability maps to a failure in one of these four layers. Identifying which layer is broken tells you where to attack and what the business impact is.

---

## Task 3 — A01: Broken Access Control

**OWASP Rank:** #1 — most critical web application vulnerability category.

### What is Broken Access Control?

When a user can perform actions or access resources beyond their intended permissions.

**Real-world examples:**
- Regular user accesses admin panel by navigating to `/admin`
- User views another user's data by changing `?user_id=1` to `?user_id=2` (IDOR)
- Unpublished content accessible via direct URL
- API returns full data objects when only subset should be visible

### Attack types under A01

| Attack | How it works | Example |
|--------|-------------|---------|
| **IDOR** (Insecure Direct Object Reference) | Change an ID in URL/body to access another user's object | `GET /invoice?id=1001` → change to `id=1002` |
| **Forced browsing** | Navigate directly to restricted URL without going through auth flow | Visit `/admin/users` directly |
| **Privilege escalation** | Perform actions reserved for higher-privilege roles | Regular user calls `DELETE /api/users/5` |
| **Path traversal** | Navigate outside intended directory | `../../etc/passwd` |
| **Missing function-level access control** | UI hides admin buttons but API endpoint unprotected | Call API directly without going through UI |

### How to find Broken Access Control

```
1. Log in as a low-privilege user
2. Note all resource IDs visible (user IDs, order IDs, document IDs)
3. Try accessing IDs that don't belong to you
4. Try accessing admin endpoints directly (/admin, /dashboard, /config)
5. Intercept requests in Burp — change role/permission parameters
6. Test horizontal (same role, different user) AND vertical (different role) escalation
```

### Prevention

- Deny by default — everything blocked unless explicitly permitted
- Enforce access control server-side, never client-side
- Log and alert on access control failures
- Rate limit API endpoints to reduce automated IDOR testing

---

## Task 4 — A07: Authentication Failures

**OWASP Rank:** #7 — previously called "Broken Authentication" in 2017 list.

### What is Authentication Failure?

When the mechanism that verifies user identity can be bypassed, brute-forced, or exploited.

### Common authentication failures

| Failure | Description | Attack |
|---------|-------------|--------|
| **Weak passwords** | No password policy enforced | Brute force with rockyou.txt |
| **No account lockout** | Unlimited login attempts allowed | Credential stuffing, brute force |
| **Credential exposure** | Passwords in URLs, logs, source code | Information disclosure |
| **Weak session tokens** | Predictable or short session IDs | Session prediction/hijacking |
| **No MFA on sensitive accounts** | Single factor only | Stolen password = full access |
| **Insecure "remember me"** | Long-lived, unrevocable tokens | Stolen cookie = persistent access |
| **Weak password reset** | Predictable reset tokens, no expiry | Account takeover via reset flow |

### Testing authentication

```bash
# Brute force login with Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"

# Username enumeration — different response for valid vs invalid user
# Valid user:   "Incorrect password"
# Invalid user: "User not found"
# → enumerate valid usernames first, then target password

# Check for no lockout — send 100 requests, if no lockout triggered = vulnerable
```

### Prevention

- Enforce strong password policy (min 12 chars, complexity)
- Implement account lockout after 5–10 failed attempts
- Use MFA on all admin and sensitive accounts
- Generate cryptographically random session tokens
- Set session token expiry and invalidate on logout
- Never expose session tokens in URLs

---

## Task 5 — A09: Logging & Alerting Failures

**OWASP Rank:** #9 — previously "Insufficient Logging and Monitoring" in 2017.

### What is a Logging & Alerting Failure?

When a system fails to record security-relevant events or fails to alert on them in time — allowing attackers to operate undetected.

**The core problem:** You cannot defend what you cannot see.

### What should be logged

| Event | Why it matters |
|-------|----------------|
| Failed login attempts | Detect brute force |
| Successful logins from new locations | Detect account compromise |
| Access control failures (403s) | Detect IDOR/forced browsing attempts |
| Input validation failures | Detect injection attempts |
| Admin actions | Detect privilege abuse |
| Password changes / account modifications | Detect account takeover |
| API calls with unusual parameters | Detect automated attacks |

### What bad logging looks like

```
# No logging — attacker brute forces 10,000 passwords, no record exists

# Insufficient logging — login recorded but no failed attempt logging
# Attacker tries 9,999 wrong passwords — no alert fires

# Logged but not monitored — logs exist but no one reviews them
# Attacker has access for 3 months before discovery
```

### What good logging looks like

```
# Structured log entry
{
  "timestamp": "2026-06-15T14:23:01Z",
  "event": "AUTH_FAILURE",
  "user": "admin@company.com",
  "ip": "185.220.101.45",
  "attempt": 47,
  "action": "ACCOUNT_LOCKED"
}

# Alert fired: 10+ failed logins from same IP in 60 seconds → SOC notified
```

### Why this matters for SOC roles

A09 is directly relevant to SOC analyst work. Every SIEM tool (Splunk, ELK, Sentinel) is designed to address A09 failures. Learning what should be logged and how to query for it is core SOC analyst skill.

**Splunk query to detect brute force:**
```spl
index=auth_logs event_type=login_failure
| stats count by src_ip, user
| where count > 10
| sort -count
```

### Prevention

- Log all authentication events — success AND failure
- Log all access control decisions — especially denials
- Use structured logging (JSON) for machine-readable output
- Send logs to a centralised SIEM — not local files only
- Set alerts for suspicious patterns (>10 failures in 60s, login from new country)
- Protect logs from tampering — write-once storage, separate log server
- Define and test incident response playbooks

---

## IAAA Failures — Combined Attack Scenario

Understanding how A01, A07, and A09 chain together:

```
Step 1 — A07 failure: No account lockout on login page
         → Attacker brute forces admin password with Hydra

Step 2 — A01 failure: Admin panel accessible to any authenticated user
         → Attacker accesses /admin without admin role check

Step 3 — A01 failure: IDOR on user management API
         → Attacker changes other users' roles: ?user_id=5&role=admin

Step 4 — A09 failure: None of the above was logged or alerted
         → Attacker operates for weeks before discovery (if ever)
```

This is not a theoretical chain — it is the actual attack pattern in the majority of real data breaches.

---

## Key Takeaways

- **A01 is #1 for a reason** — access control failures are the most common finding in real web application pentests. Every parameter that references a resource is a potential IDOR target
- **A07 failures are preventable at zero cost** — account lockout, MFA, and strong passwords cost nothing to implement but eliminate the majority of authentication attacks
- **A09 is what lets attackers stay hidden** — a well-configured SIEM running on even basic log sources would catch the majority of real-world attack chains before significant damage occurs
- **IAAA is the lens, not the checklist** — when you find a vulnerability, ask: which layer of IAAA failed? That question leads directly to the root cause and the remediation

---

## Connection to My Learning Path

| This Room | Where it appears again |
|-----------|----------------------|
| A01 Broken Access Control | PortSwigger Access Control labs, bug bounty IDOR hunting |
| A07 Auth Failures | PortSwigger Authentication labs, Hydra tool weekend |
| A09 Logging Failures | Splunk (week 3 tool), SOC Level 1 THM path, Wazuh (week 7) |

---

## Resources

- [OWASP Top 10 2025](https://owasp.org/www-project-top-ten/)
- [PortSwigger Access Control Labs](https://portswigger.net/web-security/access-control)
- [PortSwigger Authentication Labs](https://portswigger.net/web-security/authentication)
- [OWASP Testing Guide — Access Control](https://owasp.org/www-project-web-security-testing-guide/)

---

## My Progress

- [x] Guided Pentest: Web Application
- [x] Guided Pentest: Infrastructure
- [x] SQL Injection
- [x] CSRF Introduction
- [x] Nmap Live Host Discovery
- [x] Nmap Basic Port Scans
- [x] Metasploit: The Basics
- [x] OWASP Top 10 2025: IAAA Failures ← *this writeup*
- [ ] OWASP Top 10 2025 — remaining categories
- [ ] XSS, Authentication — continuing PortSwigger

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
