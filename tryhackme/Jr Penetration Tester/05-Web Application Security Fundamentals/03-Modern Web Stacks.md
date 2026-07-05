# TryHackMe — Modern Web Stacks

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Web%20Stack%20Security-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Modern Web Stacks |
| Path | Jr Penetration Tester → Web Application Security Fundamentals |
| Tasks | Task 1 — Task 7 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | MERN Stack |
| Task 3 | React / Next.js |
| Task 4 | Django |
| Task 5 | LAMP |
| Task 6 | Automation |
| Task 7 | Conclusion |

---

## Why Stack Identification Matters

Knowing which technology stack an application uses determines:
- Which CVEs to check first
- Which attack surface is most likely
- Which tools and payloads to prioritise
- Where configuration mistakes most commonly appear

**Fingerprinting approach:** Headers → cookies → default paths → error pages → response behaviour → job listings (developers mention their stack).

---

## Task 2 — MERN Stack

**MERN = MongoDB + Express + React + Node.js**

All-JavaScript stack — frontend and backend both in JS.

**Fingerprinting signals:**
```
X-Powered-By: Express              → Node.js/Express backend
Cookie: connect.sid=...            → Express session
Content-Type: application/json     → REST API (common in MERN)
```

**Common MERN vulnerabilities:**

| Vulnerability | Where it appears |
|--------------|-----------------|
| **NoSQL Injection** | MongoDB queries with unsanitised input |
| **Prototype Pollution** | JavaScript object merge operations |
| **JWT weaknesses** | MERN apps heavily use JWTs |
| **Mass assignment** | Mongoose model save with user-supplied fields |

**NoSQL Injection (MongoDB specific):**
```javascript
// Vulnerable Node.js/MongoDB query
db.users.find({ username: req.body.username, password: req.body.password })

// Bypass with MongoDB operators:
POST /login
{"username": "admin", "password": {"$gt": ""}}
// → password greater than empty string = always true → login bypass
```

---

## Task 3 — React / Next.js

React is a frontend library; Next.js adds server-side rendering (SSR) and API routes.

**Fingerprinting:**
```
/__next/static/           → Next.js static files
/_next/                   → Next.js build artifacts
/api/                     → Next.js API routes
<div id="__NEXT_DATA__">  → Next.js data hydration in page source
```

**Security-relevant Next.js features:**

```javascript
// API routes in /pages/api/ — server-side code
// Commonly missed in security review since they look like frontend files
GET /api/users             → might return all users
GET /api/admin/settings    → admin endpoint with no auth check

// Server-side props can leak sensitive data if misconfigured
// Check __NEXT_DATA__ in page source for sensitive state
<script id="__NEXT_DATA__" type="application/json">
  {"props":{"pageProps":{"user":{"email":"admin@company.com","role":"admin"}}}}
</script>
```

**Exposed source maps:**
```
/static/js/main.chunk.js.map    → Full original source code
/_next/static/chunks/app.js.map → Deobfuscated Next.js source
```

---

## Task 4 — Django

Python web framework — common in data science companies, startups, and increasingly in Pakistan's tech sector.

**Fingerprinting:**
```
Cookie: csrftoken=...             → Django's CSRF cookie
Cookie: sessionid=...             → Django session cookie
X-Frame-Options: SAMEORIGIN      → Django default security header
Server: WSGIServer/...            → Python WSGI server
```

**Django debug mode (critical misconfiguration):**
```
DEBUG = True in production → Full error pages with:
- Stack traces showing full file paths
- Local variable values at each frame
- Django settings (DATABASES, SECRET_KEY, etc.)
```

**Testing for debug mode:**
```
GET /nonexistent-path        → 500 error with full stack trace? → DEBUG=True
POST with invalid data       → Verbose error? → DEBUG=True
```

**Django admin panel:**
```
/admin/                 → Django admin login — always test default/weak credentials
/admin/login/           → Alternate path
```

**Django SECRET_KEY exposure:** If `settings.py` is exposed (via LFI, git, or debug mode), the SECRET_KEY can be used to forge session cookies and CSRF tokens.

---

## Task 5 — LAMP Stack

**LAMP = Linux + Apache + MySQL + PHP**

The most common web stack globally — WordPress, Drupal, Joomla, and most legacy PHP applications run on LAMP.

**Fingerprinting:**
```
Server: Apache/2.4.x
X-Powered-By: PHP/7.x
Cookie: PHPSESSID=...
```

**Common LAMP vulnerabilities:**

| Vuln | Location |
|------|----------|
| SQL Injection | MySQL queries with PHP string concatenation |
| LFI/RFI | PHP `include`/`require` with user input |
| File upload → RCE | PHP upload handling without type checking |
| phpinfo() exposed | `/phpinfo.php`, `/info.php`, `/test.php` |
| .env exposed | Laravel/Symfony `.env` with database credentials |

**WordPress-specific (most common LAMP target):**
```bash
wpscan --url http://target.com --enumerate u,p,t,vt
# u = users, p = plugins, t = themes, vt = vulnerable themes
```

---

## Task 6 — Automation

**WhatWeb — stack fingerprinting:**
```bash
whatweb http://target.com
whatweb -a 3 http://target.com     # Aggressive mode
```

**Wappalyzer (browser extension):** Passively fingerprints technology stack as you browse — no requests sent, instant results.

**Stack-specific vulnerability scanners:**
```bash
# WordPress
wpscan --url http://target.com

# Drupal
droopescan scan drupal -u http://target.com

# Joomla
joomscan -u http://target.com

# Django/Rails/general
nikto -h http://target.com
```

---

## Stack Comparison Summary

| Stack | Language | DB | Common vulns | Fingerprint |
|-------|---------|-----|-------------|-------------|
| MERN | JavaScript | MongoDB | NoSQL injection, prototype pollution | `X-Powered-By: Express` |
| Next.js | JavaScript | Varies | Exposed source maps, API route auth | `/_next/` paths |
| Django | Python | PostgreSQL/MySQL | Debug mode, SECRET_KEY exposure | `csrftoken` cookie |
| LAMP | PHP | MySQL | SQLi, LFI, file upload | `PHPSESSID`, `X-Powered-By: PHP` |

---

## Key Takeaways

- **Identify the stack before testing** — NoSQL injection syntax is completely different from SQL injection; the wrong payload wastes time and creates noise
- **Debug mode is always a critical finding** — Django debug, PHP error display, Node.js verbose errors all expose sensitive internal data that dramatically accelerates further exploitation
- **Source maps are free source code** — React and Next.js apps frequently ship source maps to production, turning a black-box test into a white-box one
- **LAMP is still the majority of the internet** — WordPress alone powers ~43% of all websites; knowing LAMP attack surface covers the widest real-world scope
- **Stack fingerprinting is fast and passive** — WhatWeb and Wappalyzer identify the stack in seconds with zero active scanning

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
