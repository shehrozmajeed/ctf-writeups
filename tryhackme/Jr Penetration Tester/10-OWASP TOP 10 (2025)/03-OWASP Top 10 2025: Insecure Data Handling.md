# TryHackMe — OWASP Top 10 2025: Insecure Data Handling

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-OWASP%20Top%2010-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | OWASP Top 10 2025: Insecure Data Handling |
| Path | Jr Penetration Tester → OWASP Top 10 (2025) |
| Covers | A04, A05, A08 |
| Tasks | Task 1 — Task 4 (all, 100%) |

---

## Tasks Completed

| Task | OWASP Category |
|------|----------------|
| Task 1 | Introduction |
| Task 2 | A04: Cryptographic Failures |
| Task 3 | A05: Injection |
| Task 4 | A08: Software or Data Integrity Failures |

---

## Task 2 — A04: Cryptographic Failures

**OWASP Rank: #4** — sensitive data exposed because it's transmitted or stored without proper encryption, or with weak encryption.

### Where sensitive data gets exposed

**In transit (transmission):**
```
HTTP instead of HTTPS         → credentials intercepted in plaintext
Weak TLS cipher suites        → encrypted but breakable
No HSTS header               → SSL stripping attack possible
Mixed content                → HTTP resources on HTTPS page
```

**At rest (storage):**
```
Plaintext passwords in database       → single breach = all passwords exposed
MD5/SHA1 password hashes             → rainbow tables crack instantly
Unencrypted PII in database          → GDPR/data protection violation
Credentials in config files          → exposed via LFI, git, backups
API keys hardcoded in source         → exposed via GitHub, decompilation
```

**In transit within the application:**
```
Sensitive data in GET parameters  → logged in server/proxy/browser history
?password=abc123&token=xyz        → leaked in Referer header to third parties
Session token in URL              → leaked everywhere a URL appears
```

### Testing cryptographic failures

```bash
# Check TLS quality
testssl.sh https://target.com
sslscan target.com

# Look for weak hashes in database exposure (SQLi, data breach)
# MD5: 32 hex characters  e.g. 5f4dcc3b5aa765d61d8327deb882cf99
# SHA1: 40 hex characters e.g. 5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
# Both crackable immediately with hashcat + rockyou.txt

# Check for sensitive data in URLs
# Proxy through Burp → search HTTP history for tokens in URL parameters

# Check cookie security attributes
# Missing Secure flag = cookie sent over HTTP
# Missing HttpOnly = JavaScript can read the cookie
```

### Cryptographic failures in APIs

```http
# Sensitive data in response that shouldn't be there
GET /api/v1/users/5
Response:
{
  "id": 5,
  "username": "alice",
  "password_hash": "5f4dcc3b5aa765d61d8327deb882cf99",  ← MD5, crackable
  "credit_card": "4111111111111111",                      ← PCI violation
  "ssn": "123-45-6789"                                    ← PII exposure
}
```

---

## Task 3 — A05: Injection

**OWASP Rank: #5** — untrusted data sent to an interpreter as part of a command or query.

### Injection categories

| Type | Interpreter | Classic example |
|------|------------|-----------------|
| **SQL Injection** | SQL database | `' OR 1=1--` |
| **NoSQL Injection** | MongoDB, Redis | `{"$gt": ""}` |
| **Command Injection** | OS shell | `; whoami` |
| **LDAP Injection** | LDAP directory | `*)(uid=*))(|(uid=*` |
| **XPath Injection** | XML database | `' or '1'='1` |
| **Template Injection (SSTI)** | Template engines | `{{7*7}}` → 49 |
| **HTML Injection** | Browser rendering | `<h1>Injected</h1>` |

### Server-Side Template Injection (SSTI) — often overlooked

SSTI occurs when user input is embedded directly into a template string that gets evaluated server-side.

```python
# Vulnerable Flask/Jinja2 code
template = "Hello " + request.args.get('name')
return render_template_string(template)
```

**Detection payloads (each templating engine has different syntax):**
```
{{7*7}}           → if response shows 49 → Jinja2/Twig injection confirmed
${7*7}            → FreeMarker, Velocity
<%= 7*7 %>        → ERB (Ruby)
#{7*7}            → Pebble
*{7*7}            → Thymeleaf
```

**Escalating SSTI to RCE (Jinja2):**
```python
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```

### LDAP Injection

LDAP directories (Active Directory) are queried with LDAP filter syntax. Unsanitised input breaks the filter.

```
# Normal LDAP query:
(&(uid=alice)(password=Secret123))

# Injection:
Username: alice)(uid=*))(|(uid=*
# Modified query:
(&(uid=alice)(uid=*))(|(uid=*)(password=anything))
# → Authentication bypassed
```

**Relevance:** Active Directory environments (common in Pakistani enterprise) use LDAP for authentication. LDAP injection is a high-value finding in corporate network pentests.

### Universal injection testing approach

```
1. Identify every input field, URL parameter, and API parameter
2. Submit a single quote: '
   → Does the response change? Error? Different content? → injection likely
3. Submit injection-specific syntax for the suspected interpreter
4. Confirm with a benign payload (time delay, mathematical operation)
5. Escalate to data extraction or command execution
```

---

## Task 4 — A08: Software or Data Integrity Failures

**OWASP Rank: #8** — code and infrastructure that doesn't protect against integrity violations.

### Two subtypes

**Software integrity failures:**
- Applications download updates/plugins without verifying the signature or hash
- CI/CD pipelines execute unsigned/unverified code
- Deserialisation of untrusted data executes arbitrary code

**Data integrity failures:**
- JWTs and session tokens that can be tampered with (from IAAA room)
- Client-side data trusted without server-side validation
- Critical business logic enforced only client-side

### Insecure Deserialisation

Deserialisation converts stored/transmitted data back into an object. If the data can be modified before deserialisation, the reconstructed object can contain malicious logic.

**Languages commonly vulnerable:**
- Java: `ObjectInputStream.readObject()`
- PHP: `unserialize()`
- Python: `pickle.loads()`
- Ruby: `Marshal.load()`

**Testing for deserialisation:**
```
1. Look for base64-encoded data in cookies or parameters
   rO0AB... → Java serialised object (starts with rO0)
   O:4:"User" → PHP serialised object

2. Decode and examine the structure
3. Modify fields and observe how the application responds
4. Use ysoserial (Java) or phpggc (PHP) to generate malicious payloads
```

**PHP deserialisation example:**
```php
// Serialised user object in cookie
O:4:"User":2:{s:4:"name";s:5:"alice";s:4:"role";s:4:"user";}

// Modified to admin:
O:4:"User":2:{s:4:"name";s:5:"alice";s:4:"role";s:5:"admin";}
```

### CI/CD Pipeline Integrity

```
Attack scenario:
1. Attacker gains write access to a dependency repository
2. Malicious code committed to package
3. CI/CD pipeline pulls the dependency automatically
4. Malicious code executes during build with pipeline permissions
5. Built artifacts distributed to all users of the application

Real-world: SolarWinds attack (2020)
```

**What to check in source code repos:**
- Are dependency versions pinned exactly? (`lodash@4.17.21` not `lodash@^4`)
- Are package hashes verified? (`npm install --require-lockfile`)
- Are CI/CD pipeline configurations (`.github/workflows/`, `Jenkinsfile`) protected from external contributions?

---

## OWASP 2025 Data Handling — Combined Picture

```
A04 Cryptographic Failures  → Data exposed in transit or at rest
A05 Injection               → Untrusted data executed by an interpreter
A08 Integrity Failures      → Unsigned/unverified code or data executes
```

**How they chain:**
```
A04: Login credentials stored as MD5 → cracked easily
→ A05: Attacker logs in → finds SQLi on internal page
→ A08: Internal CI/CD pipeline has no code signing
→ Attacker injects malicious build step → compromises deployment
```

---

## Key Takeaways

- **SSTI is underrated** — it's less known than SQLi but equally severe, and many developers have never heard of it. Finding `{{7*7}}` evaluating to 49 in a response is an immediate RCE vector
- **LDAP injection is enterprise-specific** — less common in bug bounty but extremely high-value in corporate network pentests where Active Directory is the authentication backbone
- **Insecure deserialisation requires source code context** — you need to know the language and serialisation format to craft effective payloads; this is an intermediate skill worth developing further
- **A08 is about trust chains** — the question is always "where does this data/code come from, and is its integrity verified at every step?"
- **A04 + A05 together cover the majority of real breach scenarios** — weak password storage combined with any injection vulnerability is the textbook path to full database compromise

---

## Complete OWASP Top 10 2025 — My Coverage

| # | Category | Room completed |
|---|----------|----------------|
| A01 | Broken Access Control | ✅ IAAA Failures room |
| A02 | Security Misconfigurations | ✅ Design Flaws room |
| A03 | Software Supply Chain | ✅ Design Flaws room |
| A04 | Cryptographic Failures | ✅ Both OWASP rooms |
| A05 | Injection | ✅ This room + SQLi + Command Injection rooms |
| A06 | Insecure Design | ✅ Design Flaws room |
| A07 | Authentication Failures | ✅ IAAA Failures + Broken Auth rooms |
| A08 | Integrity Failures | ✅ This room |
| A09 | Logging & Alerting | ✅ IAAA Failures room |
| A10 | SSRF | ✅ Intro to SSRF room |

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
