# TryHackMe — Web Server Attacks I

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Web%20Server%20Security-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Web Server Attacks - I |
| Path | Jr Penetration Tester → Web Application Security Fundamentals |
| Tasks | Task 1 — Task 8 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Identifying Web Servers |
| Task 3 | Python HTTP Server |
| Task 4 | Apache2 |
| Task 5 | Node.js (Express) |
| Task 6 | Nginx |
| Task 7 | Common Misconfigurations Across Servers |
| Task 8 | Conclusion |

---

## Task 2 — Identifying Web Servers

Fingerprinting the web server is always the first step — different servers have different vulnerabilities and misconfigurations to look for.

```bash
# Banner grabbing via curl headers
curl -I http://target.com

# Nmap service detection
nmap -sV -p 80,443 target.com

# Whatweb — automated fingerprinting
whatweb http://target.com
```

**Fingerprinting without headers (when headers are hidden):**
- Default error page styling (Apache vs Nginx have distinct 404 pages)
- Default file locations (`/server-status` for Apache, `/nginx_status` for Nginx)
- Response behaviour to malformed requests differs by server

---

## Task 3 — Python HTTP Server

Python's built-in HTTP server (`python3 -m http.server`) is commonly found running in development environments accidentally exposed to the internet or internal networks.

**Default behaviour when misconfigured:**
- Serves the entire current directory — including parent directories via path traversal
- No authentication whatsoever
- Logs all requests to stdout (easily missed by developers)

**What to test when found:**
```
http://target:8000/            → Lists directory contents
http://target:8000/../../../   → Path traversal attempt
http://target:8000/.env        → Environment file with credentials
http://target:8000/config.py   → Configuration file
```

**Finding exposed Python servers:** Shodan search `"SimpleHTTP" port:8000` or `"Python" port:8080`

---

## Task 4 — Apache2

Apache is the most widely deployed web server globally.

**Key misconfigurations to test:**

```bash
# Directory listing enabled (no index.html)
http://target.com/uploads/     → If directory listing is on, lists all files

# Server status page (internal metrics exposed)
http://target.com/server-status
http://target.com/server-info

# .htaccess file readable
http://target.com/.htaccess    → Sometimes contains rewrite rules, auth config

# Default test files still present
http://target.com/phpinfo.php  → Full PHP configuration including file paths
http://target.com/test.php
```

**Apache-specific CVEs to check after version detection:**
- Apache 2.4.49/2.4.50: Path traversal + RCE (CVE-2021-41773, CVE-2021-42013) — critical
- mod_status information disclosure
- .htaccess bypass techniques

**Version from header:**
```
Server: Apache/2.4.41 (Ubuntu)
→ Search "Apache 2.4.41 CVE" on nvd.nist.gov
```

---

## Task 5 — Node.js (Express)

Node.js applications often run with Express framework and expose different attack surface than traditional servers.

**Fingerprinting Node.js:**
```
X-Powered-By: Express          → Response header
Cookie: connect.sid=...        → Express session cookie name
```

**Common Node.js/Express misconfigurations:**

```bash
# Debug mode enabled in production — exposes routes and stack traces
GET /debug
GET /api/debug
POST any endpoint with invalid JSON → Detailed error with file paths

# Prototype pollution candidates
# Express apps using merge/extend without sanitisation
POST /api/settings {"__proto__": {"admin": true}}

# Path traversal in static file serving
GET /static/../../../etc/passwd
GET /public/../config/database.json
```

**npm packages to check after identifying the stack:**
- `package.json` sometimes accessible: `http://target.com/package.json`
- Reveals all dependencies → check each against known CVEs

---

## Task 6 — Nginx

Nginx is commonly used as a reverse proxy in front of application servers.

**Key misconfigurations:**

```bash
# Alias traversal (path confusion bug)
# If nginx config: location /files/ { alias /var/www/uploads/; }
GET /files../etc/passwd        → Traverses out of the alias

# nginx status page
http://target.com/nginx_status

# Open redirect via proxy misconfiguration
# Nginx proxy_pass with trailing slash differences

# Default nginx page still showing
http://target.com/             → "Welcome to nginx!" = unconfigured server
```

**Nginx alias traversal — the most impactful Nginx-specific finding:**
```nginx
# Vulnerable nginx config
location /static {
    alias /var/www/app/static/;
}

# Attack: missing trailing slash in location → traversal possible
GET /static../config.py        → reads /var/www/app/config.py
```

---

## Task 7 — Common Misconfigurations Across All Servers

Regardless of which server is running, these misconfigurations appear everywhere:

| Misconfiguration | How to find it | Impact |
|-----------------|----------------|--------|
| **Directory listing** | Browse to directories without index.html | File enumeration, source code disclosure |
| **Default credentials** | Try admin:admin, admin:password | Admin panel access |
| **Version disclosure** | Read Server: header | CVE research starting point |
| **Backup files** | Gobuster with .bak, .old, .orig extensions | Source code, credentials |
| **Development files** | Check for .env, .git, phpinfo.php | Credentials, full source |
| **HTTP methods** | `OPTIONS /` → check for PUT, DELETE | File upload, resource deletion |
| **Exposed admin panels** | Gobuster → /admin, /manager, /console | Admin access |

**Checking allowed HTTP methods:**
```bash
curl -X OPTIONS http://target.com/ -I

# If response includes:
Allow: GET, POST, PUT, DELETE, OPTIONS
→ PUT and DELETE being allowed is a critical finding
→ May allow file upload or deletion without authentication
```

---

## Key Takeaways

- **Always identify the server before testing** — Apache, Nginx, Node.js, and Python servers all have distinct misconfigurations and CVE histories
- **phpinfo.php is a critical finding when present** — exposes the entire server configuration including paths, PHP settings, and environment variables
- **Directory listing is still extremely common** — check every subdirectory, especially `/uploads`, `/backup`, `/files`, `/assets`
- **Nginx alias traversal is subtle but high-impact** — the missing trailing slash in a location block is easy to miss in a code review but straightforward to exploit
- **Version disclosure in Server header = CVE research starting point** — every version should be checked against NVD immediately

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
