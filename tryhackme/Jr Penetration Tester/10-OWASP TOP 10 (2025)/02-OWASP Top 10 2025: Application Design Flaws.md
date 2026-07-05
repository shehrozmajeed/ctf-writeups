# TryHackMe — OWASP Top 10 2025: Application Design Flaws

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-OWASP%20Top%2010-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | OWASP Top 10 2025: Application Design Flaws |
| Path | Jr Penetration Tester → OWASP Top 10 (2025) |
| Covers | A02, A03, A06, A10 |
| Tasks | Task 1 — Task 6 (all, 100%) |

---

## Tasks Completed

| Task | OWASP Category |
|------|----------------|
| Task 1 | Introduction |
| Task 2 | A02: Security Misconfigurations |
| Task 3 | A03: Software Supply Chain Failures |
| Task 4 | A04: Cryptographic Failures |
| Task 5 | A06: Insecure Design |
| Task 6 | Conclusion |

---

## Task 2 — A02: Security Misconfigurations

**OWASP Rank: #2** — the most widespread category, covering a huge range of "someone didn't configure this properly" findings.

**Common misconfigurations to test:**

| Misconfiguration | Test | Finding |
|-----------------|------|---------|
| Default credentials | Try admin:admin, admin:password, root:root | Authentication bypass |
| Unnecessary features enabled | OPTIONS request → PUT/DELETE allowed | File upload, resource deletion |
| Verbose error messages | Trigger errors → check for stack traces | Path/version disclosure |
| Missing security headers | curl -I → check for CSP, HSTS, X-Frame-Options | XSS, clickjacking, MITM |
| Directory listing | Browse directories without index file | File enumeration |
| Debug mode enabled | Deliberate 404/500 → check response detail | Full source paths, config |
| Cloud storage public | S3 bucket/Azure blob accessible | Data exposure |

**Security headers checklist (every one missing is a finding):**
```
Content-Security-Policy
X-Frame-Options
X-Content-Type-Options
Strict-Transport-Security
Referrer-Policy
Permissions-Policy
```

---

## Task 3 — A03: Software Supply Chain Failures

**OWASP Rank: #3** — attacking the libraries and dependencies an application uses, rather than the application itself.

**How supply chain attacks work:**
```
1. Attacker compromises a legitimate npm/PyPI/Maven package
2. Application uses that package
3. Malicious code executes within the trusted application context
4. Attacker gains same access as the application itself
```

**Real-world examples:**
- **SolarWinds (2020):** Build pipeline compromised → malicious update pushed to 18,000 organisations
- **Log4Shell (CVE-2021-44228):** Vulnerability in Log4j library affected thousands of Java applications
- **left-pad npm incident:** Single package removal broke thousands of apps — dependency fragility

**Testing for supply chain vulnerabilities:**
```bash
# Check npm packages for known vulnerabilities
npm audit

# Check Python packages
pip-audit
safety check

# Check Java
dependency-check --project myapp --scan .

# Search for outdated packages with CVEs
# Identify package versions → check snyk.io or osv.dev
```

**Finding exposed dependency files:**
```
/package.json          → Node.js dependencies + versions
/package-lock.json     → Exact locked versions
/requirements.txt      → Python dependencies
/pom.xml               → Java Maven dependencies
/composer.json         → PHP dependencies
/Gemfile               → Ruby dependencies
```

---

## Task 4 — A04: Cryptographic Failures

**OWASP Rank: #4** — sensitive data exposed due to weak, missing, or incorrectly implemented encryption.

**Common cryptographic failures:**

| Failure | Example | Impact |
|---------|---------|--------|
| **Data transmitted in plaintext** | HTTP instead of HTTPS | Credential interception |
| **Weak hashing algorithm** | MD5 or SHA1 for passwords | Fast cracking with rainbow tables |
| **No salting** | Same password = same hash | Hash comparison attacks |
| **Hardcoded keys/secrets** | `SECRET_KEY = "abc123"` in source | Full compromise |
| **Weak cipher suites** | SSLv3, TLS 1.0, RC4 | Decryption of captured traffic |
| **Sensitive data in URL** | `/reset?token=abc123` | Logged in server/proxy logs |
| **Weak random number generation** | `Math.random()` for tokens | Predictable tokens |

**Testing for weak crypto:**
```bash
# Check TLS configuration
sslscan target.com
testssl.sh target.com
nmap --script ssl-enum-ciphers -p 443 target.com

# Check for weak password hashing (look for short hashes)
# MD5: 32 hex chars
# SHA1: 40 hex chars
# bcrypt: starts with $2b$ — acceptable
# Argon2: starts with $argon2 — good

# Check certificate validity
curl -I https://target.com         # Certificate error? → expired or self-signed
```

**Password storage — good vs bad:**
```
BAD:  MD5("password") = 5f4dcc3b5aa765d61d8327deb882cf99
BAD:  SHA1("password") = 5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
BAD:  SHA256 without salt — same password = same hash always

GOOD: bcrypt($password, cost=12)
GOOD: Argon2id($password, memory=64MB, iterations=3)
```

---

## Task 5 — A06: Insecure Design

**OWASP Rank: #6** — vulnerabilities baked into the system architecture and design decisions, not just implementation mistakes.

**The key distinction:**
```
Security Misconfiguration (A02): The developer configured something wrong
Insecure Design (A06):           The fundamental design has no way to be secure
```

**Examples of insecure design:**

| Design Flaw | Why it can't be simply "fixed" |
|-------------|-------------------------------|
| Password reset by security questions | Questions are guessable/public — no implementation fixes this |
| "Remember me" via predictable cookie | Token-based design requires redesign |
| Rate limiting bolted on after the fact | If architecture doesn't support it centrally, bypasses are easy |
| Trust user-supplied data for pricing | Any implementation that trusts client-side price data is flawed by design |
| Single-factor auth for financial transfers | The design needs MFA — patching doesn't fix a design decision |

**Testing for insecure design:**
- Business logic testing: can you buy a £100 item for £0 by manipulating cart values?
- Can you skip required steps in a workflow (checkout without address, password reset without verification)?
- Can you perform actions in the wrong order (pay before adding to cart)?
- Are there race conditions in the design (two users both claiming the same coupon simultaneously)?

---

## OWASP 2025 Design Flaws — Combined Picture

```
A02 Security Misconfiguration  → Something was misconfigured (fixable)
A03 Supply Chain Failures      → A dependency was compromised (systemic risk)
A04 Cryptographic Failures     → Data exposed through weak/missing encryption
A06 Insecure Design            → The design itself is the vulnerability (requires redesign)
```

---

## Key Takeaways

- **A02 is the most common finding in real pentests** — misconfigured servers, default credentials, and missing security headers appear in almost every engagement
- **Supply chain is the fastest-growing attack vector** — a single compromised dependency can affect thousands of downstream applications with no code changes required
- **MD5/SHA1 for passwords = automatic Critical finding** — these are computationally trivial to crack; bcrypt or Argon2 is the only acceptable alternative
- **A06 requires design review, not just code review** — insecure design findings often require the most expensive remediation (architectural changes, not patches)
- **These four categories often chain together** — misconfigured server (A02) + plaintext data (A04) + predictable reset token (A06) = complete account takeover

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
