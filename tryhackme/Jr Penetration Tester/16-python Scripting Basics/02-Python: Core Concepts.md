# TryHackMe — Python: Core Concepts

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Python%20Scripting-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Python: Core Concepts |
| Path | Jr Penetration Tester → Python Scripting Basics |
| Tasks | Task 1 — Task 7 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Quick Review: Hello World, Variables, and Conditionals |
| Task 3 | Working with Strings |
| Task 4 | Lists and Dictionaries |
| Task 5 | Arithmetic and Membership Operators |
| Task 6 | Loops: for and while |
| Task 7 | Conclusion |

---

## Task 3 — Working with Strings

Strings are everywhere in security scripting — URLs, payloads, responses, file paths.

```python
target = "http://10.10.10.1/login?user=admin"

# Common string methods
print(target.upper())              # uppercase
print(target.lower())              # lowercase
print(target.replace("admin", "' OR 1=1--"))  # payload injection
print(len(target))                 # length

# Splitting and joining — useful for parsing responses
header = "Content-Type: application/json"
key, value = header.split(": ")
print(key)    # Content-Type
print(value)  # application/json

# Checking for substrings — useful for response filtering
response = "Welcome admin! You are logged in."
if "logged in" in response:
    print("Login successful")

# Stripping whitespace — common when reading wordlists
word = "  admin  \n"
print(word.strip())                # "admin"
```

**Security scripting use:** When parsing Nmap output, Burp responses, or wordlist files, string methods are used constantly — split to parse, strip to clean, in to filter, replace to build payloads.

---

## Task 4 — Lists and Dictionaries

### Lists — ordered collections

```python
# Port list for scanning
open_ports = [22, 80, 443, 8080]
open_ports.append(3306)           # add discovered port
open_ports.remove(8080)           # remove if closed

# Wordlist loaded from file
wordlist = ["admin", "login", "backup", "config", "test"]

for word in wordlist:
    print(f"Testing: /{word}")

# List comprehension — filter ports under 1024
privileged = [p for p in open_ports if p < 1024]
```

### Dictionaries — key-value pairs

```python
# Store scan results
scan_results = {
    "22": "OpenSSH 7.6p1",
    "80": "Apache 2.4.41",
    "443": "nginx 1.18.0"
}

# Access and iterate
for port, service in scan_results.items():
    print(f"Port {port}: {service}")

# Store found credentials
creds = {
    "username": "admin",
    "password": "Password123!",
    "url": "http://target.thm/login"
}
```

**Why dictionaries matter:** Security tool output is naturally key-value — port:service, username:password, finding:severity. Storing results in dictionaries makes them easy to process, filter, and write to reports.

---

## Task 5 — Arithmetic and Membership Operators

```python
# Arithmetic — useful for calculating scan progress
total_ports = 65535
scanned = 1024
progress = (scanned / total_ports) * 100
print(f"Progress: {progress:.1f}%")

# Membership operators — filter and check
ports_to_scan = [21, 22, 80, 443, 8080]
high_value = [22, 443, 3389]

for port in ports_to_scan:
    if port in high_value:
        print(f"Port {port} — high value target")

# Check if result not in known list
common_errors = [404, 403, 500]
response_code = 200
if response_code not in common_errors:
    print("Interesting response — investigate further")
```

---

## Task 6 — Loops

```python
# for loop — iterate wordlist
with open("common.txt") as f:
    for line in f:
        word = line.strip()
        print(f"Testing: {word}")

# while loop — retry with backoff
import time
retries = 0
while retries < 5:
    try:
        # attempt connection
        break
    except:
        retries += 1
        time.sleep(1)

# enumerate — loop with index (useful for progress reporting)
targets = ["10.10.10.1", "10.10.10.2", "10.10.10.3"]
for i, ip in enumerate(targets, 1):
    print(f"[{i}/{len(targets)}] Scanning {ip}")
```

---

## Key Takeaways

- **Strings, lists, and dictionaries are the three data structures used most in security scripts** — master these and you can build most tools
- **`strip()` when reading wordlists** — every line from a file has a trailing newline that breaks comparisons without it
- **Dictionaries for results** — store findings as `{port: service}` or `{url: status_code}` for easy filtering and output
- **`enumerate()` for progress** — always show the user what number out of total the script is on — security tools often run for minutes against large wordlists

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
