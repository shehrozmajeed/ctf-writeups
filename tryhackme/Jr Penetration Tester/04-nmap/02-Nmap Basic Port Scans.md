# TryHackMe — Nmap Basic Port Scans

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Category](https://img.shields.io/badge/Category-Network%20Security-blue)
![Topic](https://img.shields.io/badge/Topic-Nmap%20%7C%20Port%20Scanning-informational)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Nmap Basic Port Scans |
| URL | tryhackme.com/room/nmap02 |
| Tasks Completed | Task 1 — Task 8 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | TCP and UDP Ports | ✅ Done |
| Task 3 | TCP Flags | ✅ Done |
| Task 4 | TCP Connect Scan | ✅ Done |
| Task 5 | TCP SYN Scan | ✅ Done |
| Task 6 | UDP Scan | ✅ Done |
| Task 7 | Fine-Tuning Scope and Performance | ✅ Done |
| Task 8 | Summary | ✅ Done |

---

## What is Port Scanning?

After identifying live hosts, port scanning determines which services are running and accessible. Every open port is a potential attack surface.

**Port states Nmap reports:**

| State | Meaning |
|-------|---------|
| `open` | A service is actively accepting connections |
| `closed` | Port is reachable but no service is listening |
| `filtered` | Firewall or filter blocking — Nmap cannot determine state |
| `unfiltered` | Reachable but state unknown (ACK scan result) |
| `open\|filtered` | Cannot determine if open or filtered (UDP, FIN, NULL, Xmas) |

---

## Task 2 — TCP and UDP Ports

**65,535 ports exist.** They are divided into:

| Range | Name | Examples |
|-------|------|---------|
| 0–1023 | Well-known ports | 22 SSH, 80 HTTP, 443 HTTPS, 21 FTP |
| 1024–49151 | Registered ports | 3306 MySQL, 5432 PostgreSQL, 8080 HTTP-alt |
| 49152–65535 | Dynamic/ephemeral | Temporary client-side connections |

**TCP vs UDP:**
- TCP — connection-oriented, three-way handshake, reliable delivery
- UDP — connectionless, no handshake, faster but no confirmation

**What I learned:** In a pentest, focus on well-known ports first. A service running on an unusual port (e.g. SSH on 2222, HTTP on 8888) is worth investigating — misconfiguration or intentional obfuscation.

---

## Task 3 — TCP Flags

TCP flags control the state of a connection. Understanding them is essential for interpreting scan results and crafting custom probes.

| Flag | Full name | Purpose in scanning |
|------|-----------|-------------------|
| `SYN` | Synchronise | Initiates connection — used in SYN scans |
| `ACK` | Acknowledge | Confirms receipt — used in ACK scans |
| `RST` | Reset | Abruptly closes connection — confirms port closed |
| `FIN` | Finish | Gracefully closes connection — used in FIN scans |
| `URG` | Urgent | Marks data as urgent |
| `PSH` | Push | Sends data immediately without buffering |

**In context of scanning:**
- `SYN` sent → `SYN/ACK` received = port **open**
- `SYN` sent → `RST` received = port **closed**
- `SYN` sent → no response = port **filtered**

---

## Task 4 — TCP Connect Scan

The TCP Connect scan completes the full three-way handshake for each port tested.

```bash
nmap -sT 10.10.10.1
```

**Handshake flow:**
```
Nmap → SYN →      Target
Nmap ← SYN/ACK ← Target   (port open)
Nmap → ACK →      Target
Nmap → RST →      Target   (close connection)
```

**Characteristics:**

| Aspect | Detail |
|--------|--------|
| Privileges required | None (no raw socket access needed) |
| Detectability | High — full connection logged by target |
| Reliability | Very high — definitive open/closed result |
| Speed | Slower than SYN scan |

**When to use:** When you don't have root/administrator privileges on your scanning machine.

**What I learned:** TCP Connect is the fallback when SYN scan requires root. It is louder — every connection attempt will appear in the target's logs — making it unsuitable for stealth engagements.

---

## Task 5 — TCP SYN Scan (Stealth Scan)

The SYN scan is Nmap's default and most commonly used scan type. It never completes the handshake.

```bash
nmap -sS 10.10.10.1          # Requires root / sudo
sudo nmap 10.10.10.1         # -sS is default with root
```

**Handshake flow:**
```
Nmap → SYN →      Target
Nmap ← SYN/ACK ← Target   (port open — confirmed)
Nmap → RST →      Target   (drop the connection, never complete)
```

**Characteristics:**

| Aspect | Detail |
|--------|--------|
| Privileges required | Root / Administrator |
| Detectability | Lower — connection never fully established |
| Reliability | Very high |
| Speed | Faster than Connect scan |
| Also called | Half-open scan, stealth scan |

**Why it is stealthier:** Because the connection is never fully established, older IDS systems and some firewalls do not log incomplete connections. Modern IDS tools do detect SYN scans, so "stealth" is relative.

**What I learned:** Always use `-sS` (with sudo) as the default port scan. The speed improvement over `-sT` is significant on large ranges, and reduced logging is always preferable during an authorized engagement.

---

## Task 6 — UDP Scan

UDP services (DNS, SNMP, DHCP) are frequently overlooked but are critical attack surface.

```bash
sudo nmap -sU 10.10.10.1
```

**How UDP scanning works:**
- Nmap sends a UDP packet to each port
- No response → port is `open|filtered` (no confirmation possible with UDP)
- ICMP Port Unreachable received → port is `closed`
- Service-specific response → port is `open`

**Why UDP scanning is slow:**
- No handshake to confirm receipt
- Nmap waits for timeout before marking as filtered
- Rate limiting on ICMP responses slows bulk scanning

**Critical UDP ports to always check:**

| Port | Service | Why it matters |
|------|---------|----------------|
| 53 | DNS | Zone transfer, DNS amplification |
| 161/162 | SNMP | Community strings, device info leakage |
| 67/68 | DHCP | Rogue DHCP, IP exhaustion |
| 69 | TFTP | Unauthenticated file access |
| 123 | NTP | Amplification attacks |
| 500 | IKE/VPN | VPN fingerprinting |

**What I learned:** Never skip UDP scanning. Services running on UDP are frequently unpatched because they are invisible to TCP-only scans. SNMP with default community string `public` is one of the most common findings on internal network pentests.

---

## Task 7 — Fine-Tuning Scope and Performance

### Port specification:
```bash
nmap -p 80                    # Single port
nmap -p 80,443,8080           # Multiple ports
nmap -p 1-1024                # Range
nmap -p-                      # All 65,535 ports
nmap -F                       # Fast — top 100 ports only
nmap --top-ports 1000         # Top 1000 most common ports
```

### Timing templates:
```bash
nmap -T0    # Paranoid   — slowest, most evasive
nmap -T1    # Sneaky     — slow, evades some IDS
nmap -T2    # Polite     — reduced bandwidth
nmap -T3    # Normal     — default
nmap -T4    # Aggressive — faster, may miss filtered ports
nmap -T5    # Insane     — fastest, unreliable on slow networks
```

### Practical combinations:
```bash
# Fast initial scan — top ports, aggressive timing
sudo nmap -sS -T4 --top-ports 1000 10.10.10.1

# Thorough — all ports, service detection
sudo nmap -sS -sV -sC -p- -T4 10.10.10.1

# Stealth — slow timing, SYN scan
sudo nmap -sS -T1 -p- 10.10.10.1

# UDP top ports + TCP SYN combined
sudo nmap -sS -sU --top-ports 100 10.10.10.1
```

**What I learned:** Start with `-T4 --top-ports 1000` for speed, then follow up with `-p-` on interesting targets. Never use `-T5` on real engagements — too many false negatives on congested networks.

---

## Key Takeaways

- **SYN scan (`-sS`) is the default for a reason** — faster than Connect scan and produces less log noise on the target
- **Always scan UDP** — DNS, SNMP, and DHCP vulnerabilities are missed entirely by TCP-only scans
- **Timing matters in two directions** — too fast (`-T5`) misses filtered ports, too slow (`-T0`) wastes engagement time
- **Port state interpretation requires context** — `filtered` means a firewall exists, not that the service doesn't — probe from multiple positions
- **`-p-` on every target, always** — services on non-standard ports (SSH on 2222, web on 8888) are found only by scanning all 65,535 ports

---

## My Nmap Reference Card

```bash
# Discovery only (no port scan)
sudo nmap -sn 192.168.1.0/24

# Standard pentest opening scan
sudo nmap -sS -sV -sC -O -p- -T4 -oN scan.txt 10.10.10.1

# Quick first look
sudo nmap -sS -T4 --top-ports 1000 10.10.10.1

# UDP + TCP combined
sudo nmap -sS -sU -T4 --top-ports 200 10.10.10.1

# Stealth scan with output
sudo nmap -sS -T2 -p- --open -oN stealth_scan.txt 10.10.10.1
```

---

## Resources

- [Nmap Book — Port Scanning chapter (free)](https://nmap.org/book/man-port-scanning-techniques.html)
- [TryHackMe Nmap series: nmap01, nmap02, nmap03, nmap04](https://tryhackme.com)

---

## My Progress

- [x] Guided Pentest: Web Application
- [x] Guided Pentest: Infrastructure
- [x] SQL Injection
- [x] CSRF Introduction
- [x] Nmap Live Host Discovery
- [x] Nmap Basic Port Scans ← *this writeup*
- [ ] Nmap Advanced — nmap03, nmap04
- [ ] XSS, Authentication — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
