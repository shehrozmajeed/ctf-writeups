# TryHackMe — Python: Building Scripts

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Python%20Scripting-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Python: Building Scripts |
| Path | Jr Penetration Tester → Python Scripting Basics |
| Tasks | Task 1 — Task 7 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Functions |
| Task 3 | Error Handling |
| Task 4 | Reading and Writing Files |
| Task 5 | Libraries and Pip |
| Task 6 | Putting It All Together: Password Strength Checker |
| Task 7 | Conclusion |

---

## Task 2 — Functions

Functions make scripts reusable, readable, and testable — essential for any tool beyond a few lines.

```python
# Basic function structure
def scan_port(ip, port, timeout=2):
    """Check if a port is open on the given IP."""
    import socket
    try:
        sock = socket.socket()
        sock.settimeout(timeout)
        result = sock.connect_ex((ip, port))
        sock.close()
        return result == 0   # True if open
    except Exception:
        return False

# Call the function
if scan_port("10.10.10.1", 80):
    print("Port 80 is open")

# Function with multiple return values
def check_response(url):
    import requests
    r = requests.get(url, timeout=3)
    return r.status_code, len(r.content)

code, size = check_response("http://target.thm/admin")
print(f"Status: {code}, Size: {size} bytes")
```

**Why functions matter in security scripting:** Without functions, a port scanner that checks 1000 ports would repeat the same connection logic 1000 times. A function wraps the logic once — the loop just calls it. This also makes it easy to test individual components and reuse code across multiple tools.

---

## Task 3 — Error Handling

Security tools connect to live systems that may be slow, unreachable, or actively blocking. Error handling prevents scripts from crashing on the first failure.

```python
import socket
import requests

# try/except — the core pattern
def safe_request(url):
    try:
        r = requests.get(url, timeout=3)
        return r.status_code
    except requests.exceptions.ConnectionError:
        return None    # host unreachable
    except requests.exceptions.Timeout:
        return None    # timed out
    except Exception as e:
        print(f"Unexpected error: {e}")
        return None

# Multiple exception types
def connect_port(ip, port):
    try:
        sock = socket.socket()
        sock.settimeout(2)
        sock.connect((ip, port))
        banner = sock.recv(1024).decode("utf-8", errors="ignore")
        sock.close()
        return banner
    except socket.timeout:
        return None    # port filtered
    except ConnectionRefusedError:
        return None    # port closed
    except Exception:
        return None

# finally — always runs, even if exception occurs
def scan_with_cleanup(ip, port):
    sock = None
    try:
        sock = socket.socket()
        sock.connect((ip, port))
        return True
    except:
        return False
    finally:
        if sock:
            sock.close()   # always close the socket
```

**Why this matters:** A port scanner hitting 1000 ports will encounter timeouts, refused connections, and unreachable hosts on every run. Without try/except, the script crashes on the first failure. With it, the script logs the failure and continues to the next target.

---

## Task 4 — Reading and Writing Files

File I/O is how security scripts consume wordlists and save results.

```python
# Reading a wordlist — the standard pattern
def load_wordlist(filepath):
    with open(filepath, "r") as f:
        return [line.strip() for line in f if line.strip()]

wordlist = load_wordlist("/usr/share/wordlists/rockyou.txt")

# Writing scan results to file
def save_results(filename, results):
    with open(filename, "w") as f:
        for item in results:
            f.write(item + "\n")

# Appending to an existing file (for ongoing scans)
with open("found_paths.txt", "a") as f:
    f.write("/admin\n")

# Reading/writing JSON — useful for structured results
import json

results = {
    "target": "10.10.10.1",
    "open_ports": [22, 80, 443],
    "findings": ["SQLi on /login", "RCE via /upload"]
}

with open("scan_results.json", "w") as f:
    json.dump(results, f, indent=2)

# Read it back
with open("scan_results.json", "r") as f:
    data = json.load(f)
    print(data["open_ports"])
```

**Security scripting pattern:** Every tool follows the same file pattern — `load_wordlist()` at the start, `save_results()` at the end, and logging discovered items with append mode during the scan so nothing is lost if the script is interrupted.

---

## Task 5 — Libraries and Pip

Python's strength for security scripting comes from its library ecosystem.

```bash
# Install a library
pip install requests
pip install scapy
pip install paramiko

# List installed packages
pip list

# Install from requirements file
pip install -r requirements.txt
```

### Essential libraries for security scripting

| Library | Purpose | Install |
|---------|---------|---------|
| `requests` | HTTP requests — web scanning, API testing | `pip install requests` |
| `socket` | Raw TCP/UDP connections — port scanning | Built-in |
| `scapy` | Packet crafting and network analysis | `pip install scapy` |
| `paramiko` | SSH connections and brute forcing | `pip install paramiko` |
| `argparse` | Command-line arguments for tools | Built-in |
| `threading` | Run multiple scans simultaneously | Built-in |
| `re` | Regular expressions — parsing output | Built-in |

```python
# Argparse — makes tools usable from command line
import argparse

parser = argparse.ArgumentParser(description="Port Scanner")
parser.add_argument("target", help="Target IP address")
parser.add_argument("-p", "--ports", default="1-1000", help="Port range")
parser.add_argument("-t", "--timeout", type=float, default=2.0)
args = parser.parse_args()

print(f"Scanning {args.target} on ports {args.ports}")
```

---

## Task 6 — Password Strength Checker (Putting It All Together)

Built a complete tool combining all concepts: functions, error handling, file I/O, and string analysis.

```python
import re

def check_password_strength(password):
    score = 0
    feedback = []

    if len(password) >= 12:
        score += 1
    else:
        feedback.append("Use at least 12 characters")

    if re.search(r"[A-Z]", password):
        score += 1
    else:
        feedback.append("Add uppercase letters")

    if re.search(r"[a-z]", password):
        score += 1
    else:
        feedback.append("Add lowercase letters")

    if re.search(r"\d", password):
        score += 1
    else:
        feedback.append("Add numbers")

    if re.search(r"[!@#$%^&*(),.?\":{}|<>]", password):
        score += 1
    else:
        feedback.append("Add special characters")

    ratings = {5: "Strong", 4: "Good", 3: "Moderate", 2: "Weak", 1: "Very weak"}
    rating = ratings.get(score, "Very weak")

    return rating, feedback

# Test it
password = "MyP@ssw0rd2026!"
rating, tips = check_password_strength(password)
print(f"Rating: {rating}")
for tip in tips:
    print(f"  - {tip}")
```

---

## Key Takeaways

- **Functions are the single most important scripting habit** — every reusable action becomes a function; scripts become a list of function calls
- **try/except on every network operation** — security tools connect to live, unreliable systems; defensive error handling is not optional
- **`with open()` for all file operations** — automatically handles closing the file even if an error occurs
- **`argparse` makes tools professional** — a tool that accepts command-line arguments is immediately more useful and more impressive in a portfolio than a script with hardcoded values

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
