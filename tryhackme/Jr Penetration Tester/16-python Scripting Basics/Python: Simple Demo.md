# TryHackMe — Python: Simple Demo

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Python%20Scripting-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Python: Simple Demo |
| Path | Jr Penetration Tester → Python Scripting Basics |
| Tasks | Task 1 — Task 5 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Variables |
| Task 3 | Conditional Statements |
| Task 4 | Iterations |
| Task 5 | Conclusion |

---

## Task 2 — Variables

Variables store data that the script can reference and manipulate.

```python
# Basic variable types used in security scripts
target_ip = "10.10.10.1"          # string — target address
port = 80                          # integer — port number
is_open = False                    # boolean — port state
timeout = 2.5                      # float — connection timeout

# Type checking
print(type(target_ip))             # <class 'str'>
print(type(port))                  # <class 'int'>

# String formatting — essential for building dynamic commands/URLs
url = f"http://{target_ip}:{port}/admin"
print(url)                         # http://10.10.10.1:80/admin
```

**Why this matters for security scripting:** Every tool you build — port scanners, subdomain finders, brute forcers — uses variables to hold target IPs, ports, wordlists, and results. Getting comfortable with f-strings is particularly important since URL and command construction relies on them constantly.

---

## Task 3 — Conditional Statements

Conditionals make scripts respond differently based on what they find.

```python
response_code = 200

if response_code == 200:
    print("Page found — potential target")
elif response_code == 403:
    print("Forbidden — exists but restricted")
elif response_code == 404:
    print("Not found")
else:
    print(f"Unexpected response: {response_code}")
```

**Security scripting application:**
```python
port = 22
is_open = True

if is_open and port == 22:
    print("SSH is open — attempt key auth or brute force")
elif is_open and port == 80:
    print("HTTP open — begin web enumeration")
```

---

## Task 4 — Iterations

Loops let scripts repeat actions across lists of targets, ports, or wordlist entries.

```python
# Loop through a list of ports to scan
ports = [21, 22, 80, 443, 3306, 8080]

for port in ports:
    print(f"Scanning port {port}...")

# Loop through a wordlist
wordlist = ["admin", "login", "dashboard", "backup", "config"]

for path in wordlist:
    url = f"http://10.10.10.1/{path}"
    print(f"Testing: {url}")

# While loop — retry until condition met
attempts = 0
max_attempts = 3

while attempts < max_attempts:
    print(f"Attempt {attempts + 1}")
    attempts += 1
```

**Why iterations are the core of security scripting:** A port scanner loops through ports. A subdomain finder loops through a wordlist. A brute forcer loops through passwords. Every security tool at its core is a loop doing something to each item in a list.

---

## Key Takeaways

- **Variables, conditionals, and loops are the building blocks of every security tool** — a port scanner is a loop (iterate ports) inside a conditional (if open, print it)
- **f-strings make URL and command building clean** — `f"http://{ip}:{port}/{path}"` is the pattern used in almost every web security script
- **Python's readability** makes it the dominant language for security tooling — most tools in the field (Impacket, sqlmap, Metasploit modules) are Python

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
