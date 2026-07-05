# TryHackMe — Content Discovery

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Web%20Enumeration-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Content Discovery |
| Path | Jr Penetration Tester → Web Application Security Fundamentals |
| Tasks | Task 1 — Task 8 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Manual Discovery - Common Files |
| Task 3 | Manual Discovery - Headers & Framework Stack |
| Task 4 | OSINT - Search Engines & Web Tools |
| Task 5 | OSINT - Repositories & Archives |
| Task 6 | Automated Discovery - Gobuster Fundamentals |
| Task 7 | Automated Discovery - Subdomains & Virtual Hosts |
| Task 8 | Conclusion |

---

## Task 2 — Manual Discovery: Common Files

Before running any tool, manually check for files that commonly exist on web servers:

```
/robots.txt        → Tells search engines what not to index — often reveals hidden paths
/sitemap.xml       → Full site structure for SEO — maps all pages
/favicon.ico       → Can fingerprint the framework (IDOR'd favicon hashes)
/.well-known/      → ACME challenge files, security.txt
/crossdomain.xml   → Flash-era policy file — sometimes still present
/security.txt      → Contact info for responsible disclosure
```

**robots.txt is always the first manual check** — developers frequently list sensitive directories to exclude from search engine indexing, inadvertently advertising them to attackers.

```
User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /api/internal/
```

Every `Disallow` entry is a target to investigate.

---

## Task 3 — Manual Discovery: Headers & Framework Stack

HTTP response headers reveal the technology stack without any active scanning:

```bash
curl -I https://target.com

# Sample response headers:
Server: Apache/2.4.41 (Ubuntu)          → Web server + version
X-Powered-By: PHP/7.4.3                 → Backend language
X-Generator: WordPress 6.1              → CMS
X-Frame-Options: SAMEORIGIN             → Security header present
Strict-Transport-Security: max-age=...  → HSTS configured
```

**Missing security headers are findings:**

| Missing Header | Risk |
|----------------|------|
| `X-Frame-Options` | Clickjacking possible |
| `Content-Security-Policy` | XSS harder to mitigate |
| `X-Content-Type-Options` | MIME sniffing attacks |
| `Strict-Transport-Security` | SSL stripping possible |

**Framework fingerprinting without headers:**
- Page source comments: `<!-- WordPress 6.1 -->`
- Default CSS/JS paths: `/wp-content/themes/`, `/vendor/laravel/`
- Error page signatures — framework-specific 404 pages
- Cookie names: `PHPSESSID` (PHP), `JSESSIONID` (Java), `ASP.NET_SessionId` (.NET)

---

## Task 4 — OSINT: Search Engines & Web Tools

**Google Dorking** — advanced search operators to find hidden content:

```
site:target.com               → All indexed pages
site:target.com filetype:pdf  → PDF documents
site:target.com inurl:admin   → URLs containing "admin"
site:target.com intitle:login → Pages with "login" in title
site:target.com ext:sql       → SQL files
"target.com" "internal use only"  → Accidentally indexed internal pages
cache:target.com              → Google's cached version
```

**Other OSINT tools:**
- **Wayback Machine (web.archive.org):** Old versions of pages — sometimes reveals removed content, old credentials, or deprecated endpoints still running
- **crt.sh:** SSL certificate transparency logs — finds subdomains without scanning the target
- **VirusTotal:** URL scan history, sometimes reveals subdomains

---

## Task 5 — OSINT: Repositories & Archives

**GitHub reconnaissance:**
```
site:github.com target.com         → Code mentioning the target
"target.com" password               → Leaked credentials
"target.com" api_key                → Exposed API keys
"target.com" secret                 → Configuration secrets
```

**What to look for in repositories:**
- Hardcoded credentials in config files
- Internal API endpoints referenced in code
- Old branches or commits containing removed sensitive data (`git log --all`)
- `.env` files accidentally committed

**Wayback Machine for content discovery:**
```
web.archive.org/web/*/target.com/*

→ Find old sitemap versions
→ Discover removed pages that may still be live on the server
→ Find previously exposed files
```

---

## Task 6 — Automated Discovery: Gobuster

Gobuster brute-forces URLs against a target using wordlists:

```bash
# Directory and file discovery
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# With file extensions
gobuster dir -u http://target.com \
  -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,txt,bak,old

# Common useful flags
-t 50          # 50 threads (faster)
-o results.txt # Save output
-b 404,403     # Ignore these status codes
-k             # Skip SSL verification
```

**Recommended wordlists (from SecLists):**
```
Discovery/Web-Content/common.txt           → Quick scan
Discovery/Web-Content/directory-list-2.3-medium.txt  → Thorough scan
Discovery/Web-Content/big.txt              → Comprehensive
```

**What Gobuster finds:**
- Hidden admin panels (`/admin`, `/administrator`, `/wp-admin`)
- Backup files (`/backup.zip`, `/db.sql`, `/config.bak`)
- Development artifacts (`/.git`, `/.env`, `/test`, `/dev`)
- API endpoints not linked from the frontend

---

## Task 7 — Automated Discovery: Subdomains & Virtual Hosts

**Subdomain enumeration:**
```bash
# DNS-based subdomain brute force
gobuster dns -d target.com \
  -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# Virtual host enumeration (same IP, different Host header)
gobuster vhost -u http://target.com \
  -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# Alternative tools
subfinder -d target.com
amass enum -d target.com
```

**DNS vs Virtual Host difference:**
- **DNS subdomain brute force:** queries DNS to find real subdomains
- **Virtual host brute force:** sends HTTP requests with different `Host:` headers to the same IP — finds applications not in DNS at all

---

## Key Takeaways

- **Content discovery precedes exploitation** — you can't attack what you can't find
- **robots.txt and response headers are free intelligence** — always check manually before running tools
- **Google dorking finds what scanners miss** — indexed pages, accidentally exposed files, and cached old versions
- **GitHub recon often yields the highest-value findings** — hardcoded credentials and API keys in public repos are extremely common
- **Always use SecLists, not just default wordlists** — the quality of content discovery depends entirely on the quality of the wordlist

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
