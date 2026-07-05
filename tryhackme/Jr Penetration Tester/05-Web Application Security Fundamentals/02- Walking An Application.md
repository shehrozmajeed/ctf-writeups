# TryHackMe — Walking An Application

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Manual%20Recon-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Walking An Application |
| Path | Jr Penetration Tester → Web Application Security Fundamentals |
| Tasks | Task 1 — Task 8 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Walking An Application |
| Task 2 | Exploring The Website |
| Task 3 | Viewing The Page Source |
| Task 4 | Developer Tools - Inspector |
| Task 5 | Developer Tools - Debugger |
| Task 6 | Developer Tools - Network |
| Task 7 | Developer Tools - Storage |
| Task 8 | Conclusion |

---

## What This Room Teaches

Manual application review using only a browser — no tools, no scans, no automation. The goal is building the habit of understanding what an application is doing before touching any attack tooling.

---

## Task 2 — Exploring The Website

Before testing anything, walk through the application as a normal user:

```
- Click every link and button
- Submit every form (with dummy data)
- Register an account if possible
- Log in and explore authenticated functionality
- Note every page, parameter, and feature
```

**What to document:**
- Input fields (potential injection points)
- File upload functionality (potential shell upload)
- User-controlled content displayed to others (potential XSS/stored)
- URL parameters that reference IDs or filenames (potential IDOR/traversal)
- Authentication and access control boundaries (what changes when logged in)

---

## Task 3 — Viewing Page Source

`Ctrl+U` or right-click → View Page Source reveals what the server actually sent — before the browser renders it.

**What to look for:**

```html
<!-- Developer comment with sensitive info -->
<!-- Admin panel at /secret-admin-page -->

<!-- Hardcoded credentials -->
<input type="hidden" name="admin_token" value="abc123secret">

<!-- Framework/version disclosure -->
<meta name="generator" content="WordPress 6.1">

<!-- Internal paths -->
<script src="/var/www/internal/js/admin.js"></script>

<!-- Disabled form fields still sent to server -->
<input type="hidden" name="role" value="user">

<!-- API keys in JavaScript -->
const API_KEY = "sk_live_abc123xyz";
```

**Key habit:** Search page source for: `password`, `secret`, `key`, `token`, `admin`, `TODO`, `FIXME`, `<!--`

---

## Task 4 — Developer Tools: Inspector

`F12` → Inspector (Elements tab) shows the live DOM — the page as rendered and modified by JavaScript, not just the raw source.

**What Inspector reveals that source doesn't:**
- JavaScript-modified content (hidden divs shown/hidden dynamically)
- Values added by JS after page load
- Real-time changes as you interact with the page

**Practical use:**
```
1. Find a hidden form field: <input type="hidden" name="role" value="user">
2. In Inspector: double-click the value → change "user" to "admin"
3. Submit the form — does the server accept the modified value?
```

This is the manual equivalent of what Burp Intruder automates — directly editing the DOM before submission.

---

## Task 5 — Developer Tools: Debugger

The Debugger tab shows all JavaScript files loaded by the page and allows setting breakpoints.

**Security uses:**

```javascript
// Find API endpoints hardcoded in JS
fetch('/api/v1/internal/users')

// Find authentication logic client-side
if (user.role === 'admin') { showAdminPanel(); }
// → if role check is only client-side, can bypass by modifying JS

// Find obfuscated or minified JS with secrets
// → Pretty-print with {} button → search for "key", "secret", "password"
```

**Source map files:** If `app.js.map` exists, the original unminified source is accessible — much easier to read than minified production JS.

---

## Task 6 — Developer Tools: Network

The Network tab captures every HTTP request the browser makes — including AJAX requests to APIs that aren't visible in the URL bar.

**Why this matters:** Modern web apps make dozens of background API calls that never appear in the address bar. These hidden requests are often less tested and more vulnerable.

```
1. Open Network tab (F12 → Network)
2. Interact with the application normally
3. Watch every request appear in real-time
4. Click any request → see full headers, payload, and response
5. Look for:
   → API endpoints: /api/v1/users, /api/internal/config
   → Authentication tokens in request headers
   → Sensitive data in responses (more than the UI shows)
   → Parameters that reference IDs or usernames (IDOR candidates)
```

**Filter options:** XHR/Fetch shows only AJAX calls, hiding page loads and static assets — cleaner view for API hunting.

---

## Task 7 — Developer Tools: Storage

The Storage tab shows everything the browser stores locally for the application:

```
Cookies          → Session tokens, auth flags, tracking
localStorage     → Persistent key-value data (survives browser close)
sessionStorage   → Temporary key-value data (cleared on tab close)
IndexedDB        → Structured database in browser
Cache Storage    → Service worker cached files
```

**What to check:**

```javascript
// Check if role or user data is stored client-side
localStorage.getItem('userRole')     // → "admin"?
localStorage.getItem('authToken')    // → JWT?

// Modify and see if server accepts it
localStorage.setItem('userRole', 'admin')
// Reload page — does the application behave differently?
```

**Session cookie attributes to check:**
```
HttpOnly: true   → Good (JS can't read it)
Secure: true     → Good (HTTPS only)
SameSite: Strict → Good (CSRF protection)

Missing any of these = finding
```

---

## Key Takeaways

- **Manual review before tools** — understanding what the app does legitimately makes attack attempts more targeted and less noisy
- **Page source and DevTools find what scanners miss** — hardcoded tokens, hidden fields, and client-side logic flaws require manual review
- **Network tab is where API vulnerabilities live** — background AJAX calls to undocumented endpoints are frequently the highest-value findings
- **Client-side role checks are always bypassable** — if the application makes access decisions based on localStorage or cookie values without server-side validation, those decisions can be overridden
- **This workflow is the foundation of every web pentest** — before Burp, before SQLmap, before any tool: walk the application manually first

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
