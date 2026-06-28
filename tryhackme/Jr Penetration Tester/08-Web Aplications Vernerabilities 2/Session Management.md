# TryHackMe — Session Management

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Session%20Security-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Session Management |
| Path | Jr Penetration Tester → Web Application Vulnerabilities |
| Tasks | Task 1 — Task 7 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | What is Session Management? |
| Task 3 | Authentication vs Authorisation |
| Task 4 | Cookies vs Tokens |
| Task 5 | Securing the Session Lifecycle |
| Task 6 | Exploiting Insecure Session Management |
| Task 7 | Conclusion |

---

## Task 2 — What is Session Management?

HTTP is stateless — every request is independent, with no memory of previous ones. Session management solves this by creating a persistent identity across multiple requests after login.

**How it works:**
```
1. User logs in with credentials
2. Server creates a session → assigns a unique session ID
3. Session ID sent to browser (as cookie or token)
4. Browser includes session ID in every subsequent request
5. Server looks up session ID → knows who the user is
```

If an attacker obtains a valid session ID, they can impersonate that user without ever knowing their password.

---

## Task 3 — Authentication vs Authorisation

The most commonly confused pair in security interviews:

| Concept | Question it answers | Example |
|---------|---------------------|---------|
| **Authentication** | Who are you? | Username + password login |
| **Authorisation** | What are you allowed to do? | Admin can delete users, regular user cannot |

**Attack relevance:**
- Broken Authentication → attacker logs in as someone else
- Broken Authorisation → attacker accesses resources beyond their permission level (IDOR)
- Both can exist independently — a properly authenticated user can still access things they shouldn't

---

## Task 4 — Cookies vs Tokens

Two main mechanisms for carrying session identity after login:

| | Cookies | Tokens (JWT) |
|--|---------|-------------|
| **Storage** | Browser cookie jar | localStorage, sessionStorage, or cookie |
| **Server-side storage** | Session data stored on server | All data encoded in the token itself (stateless) |
| **Validation** | Server looks up session ID in database | Server verifies token signature |
| **Revocation** | Easy — delete session from server | Hard — token valid until expiry |
| **Vulnerable to** | XSS (cookie theft), CSRF, session fixation | JWT algorithm confusion, signature bypass, weak secrets |

### Cookie security attributes

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Path=/
```

| Attribute | What it does |
|-----------|-------------|
| `HttpOnly` | JavaScript cannot read the cookie — blocks XSS-based cookie theft |
| `Secure` | Cookie only sent over HTTPS — never over HTTP |
| `SameSite=Strict` | Cookie not sent on cross-site requests — blocks CSRF |
| `Path=/` | Cookie scope limited to the application root |

**Missing any of these is a finding** — especially missing `HttpOnly` (enables XSS cookie theft) and missing `SameSite` (enables CSRF).

---

## Task 5 — Securing the Session Lifecycle

A secure session has three distinct phases, each with specific requirements:

### Creation (after login)
- Generate a cryptographically random session ID — minimum 128 bits
- Never reuse session IDs from before login (prevents session fixation)
- Store server-side: session ID → user data mapping

### Maintenance (during use)
- Regenerate session ID after privilege escalation (e.g. logging in, password change)
- Set appropriate expiry — short for sensitive apps, longer for low-risk
- Implement idle timeout — expire sessions after inactivity
- Bind session to IP/User-Agent (optional — tradeoff with usability)

### Termination (logout)
- Invalidate session server-side immediately on logout
- Clear the cookie from the browser
- Never rely only on cookie deletion — the session must be killed server-side too

**Common mistake:** Logging out only deletes the cookie client-side without invalidating the server-side session. The old session ID remains valid and can be replayed by anyone who captured it.

---

## Task 6 — Exploiting Insecure Session Management

### Session Hijacking

Stealing a valid session ID and using it to impersonate the victim.

**Methods to steal session IDs:**
- XSS: `document.cookie` exfiltration (blocked by `HttpOnly`)
- Network sniffing on HTTP (blocked by `Secure` flag)
- Predictable session IDs (sequential, timestamp-based)
- Session fixation — attacker sets victim's session ID before login

### Session Fixation Attack

```
1. Attacker gets a pre-login session ID from the server
2. Attacker tricks victim into using that session ID
   (via URL parameter: ?sessionid=ATTACKER_KNOWN_ID)
3. Victim logs in — server authenticates them under that session ID
4. Attacker already knows the session ID → hijacks the now-authenticated session
```

**Prevention:** Always generate a new session ID immediately after successful login.

### Cookie Manipulation

Testing for weak session tokens using Burp Suite:

```
1. Log in — capture the session cookie in Burp
2. Decode it: Base64? URL-encoded? JWT?
3. If readable: identify if it contains predictable data (username, role, timestamp)
4. Modify: change role=user → role=admin, re-encode
5. Resend with modified cookie — does the server accept it?
```

**What I did:** Used Burp Repeater to intercept session cookies, decoded them, identified that role information was stored client-side in the cookie, modified the value, and confirmed the server accepted the manipulated cookie without server-side validation.

---

## Key Takeaways

- **Session IDs must be unpredictable and long** — sequential or short IDs can be guessed or brute-forced
- **Always regenerate session ID on login** — prevents session fixation
- **HttpOnly + Secure + SameSite together** — the three cookie attributes that must always be present
- **Logout must invalidate server-side** — client-side cookie deletion alone is insufficient
- **Client-side stored roles are always a finding** — if the application trusts role data stored in a cookie without server-side verification, it is exploitable

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
