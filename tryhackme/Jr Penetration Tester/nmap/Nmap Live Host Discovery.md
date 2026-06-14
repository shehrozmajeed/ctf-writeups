# TryHackMe — Nmap Live Host Discovery

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Category](https://img.shields.io/badge/Category-Network%20Security-blue)
![Topic](https://img.shields.io/badge/Topic-Nmap%20%7C%20Host%20Discovery-informational)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Nmap Live Host Discovery |
| URL | tryhackme.com/room/nmap01 |
| Tasks Completed | Task 1 — Task 9 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Subnetworks | ✅ Done |
| Task 3 | Understanding Hosts Discovery through TCP/IP Layer | ✅ Done |
| Task 4 | Enumerating Targets | ✅ Done |
| Task 5 | Nmap Host Discovery Using ARP | ✅ Done |
| Task 6 | Nmap Host Discovery Using ICMP | ✅ Done |
| Task 7 | Nmap Host Discovery Using TCP and UDP | ✅ Done |
| Task 8 | Using Reverse-DNS Lookup | ✅ Done |
| Task 9 | Summary | ✅ Done |

---

## What is Host Discovery?

Before exploiting anything, you need to know what is alive on the network. Host discovery is the first phase of every penetration test — identifying which IP addresses have live systems responding.

**Why it matters:**
- Scanning every port on every IP in a large range wastes time and creates noise
- Host discovery narrows scope to only live targets before deeper enumeration begins
- Different techniques are needed depending on network position (same subnet vs remote)

---

## Task 2 — Subnetworks

Understanding subnets is fundamental to scoping a pentest correctly.

**Key concepts:**
- A subnet is a logical division of an IP network
- CIDR notation defines the range: `192.168.1.0/24` = 254 usable hosts
- `/24` = 256 addresses, `/16` = 65,536 addresses, `/8` = 16.7M addresses
- Host discovery must cover the full target subnet, not just known IPs

**Common subnet ranges encountered in pentests:**
```
192.168.1.0/24   →  192.168.1.1 – 192.168.1.254  (254 hosts)
10.0.0.0/8       →  10.0.0.1 – 10.255.255.254    (16M hosts)
172.16.0.0/12    →  172.16.0.1 – 172.31.255.254  (1M hosts)
```

---

## Task 3 — TCP/IP Layer Host Discovery

Different layers of the TCP/IP model are used for different discovery techniques:

| Layer | Protocol | Nmap technique | Range |
|-------|----------|----------------|-------|
| Layer 2 (Data Link) | ARP | ARP scan | Same subnet only |
| Layer 3 (Network) | ICMP | Ping scan | Any routable target |
| Layer 4 (Transport) | TCP/UDP | TCP/UDP probes | Any routable target |

**Key insight:** ARP is the most reliable for same-subnet discovery because it cannot be blocked by host-level firewalls. ICMP and TCP probes are needed for remote targets.

---

## Task 4 — Enumerating Targets

Nmap accepts targets in multiple formats:

```bash
nmap 192.168.1.1              # Single host
nmap 192.168.1.1-20           # Range
nmap 192.168.1.0/24           # Subnet
nmap -iL targets.txt          # From file
nmap scanme.nmap.org          # Hostname
```

**Useful flags before scanning:**
```bash
nmap -sL 192.168.1.0/24       # List targets without scanning (verify scope)
nmap -n 192.168.1.0/24        # No DNS resolution (faster)
nmap -R 192.168.1.0/24        # Force reverse DNS lookup
```

---

## Task 5 — Host Discovery Using ARP

ARP (Address Resolution Protocol) resolves IP addresses to MAC addresses on the local network.

**How it works for discovery:**
- Nmap sends ARP requests to every IP in the range
- Any live host on the same subnet must respond with its MAC address
- No firewall can block ARP at Layer 2 — most reliable local discovery method

```bash
nmap -PR -sn 192.168.1.0/24
```

| Flag | Meaning |
|------|---------|
| `-PR` | ARP ping — use ARP for host discovery |
| `-sn` | Ping scan only — no port scan after discovery |

**Output shows:** IP address + MAC address + vendor of each live host.

**What I learned:** ARP scanning is the go-to technique when you are on the same network segment. It is faster, more reliable, and less detectable than ICMP or TCP probes on a local network.

---

## Task 6 — Host Discovery Using ICMP

ICMP (Internet Control Message Protocol) is used for ping-based discovery across routable networks.

**Three ICMP techniques:**

```bash
nmap -PE -sn 192.168.1.0/24   # ICMP Echo Request (standard ping)
nmap -PP -sn 192.168.1.0/24   # ICMP Timestamp Request
nmap -PM -sn 192.168.1.0/24   # ICMP Address Mask Request
```

| Type | Use case |
|------|---------|
| Echo Request (`-PE`) | Most common — blocked by many firewalls |
| Timestamp (`-PP`) | Bypasses firewalls that block echo requests |
| Address Mask (`-PM`) | Rarely used, legacy systems |

**What I learned:** ICMP Echo is often blocked by Windows hosts and enterprise firewalls. Timestamp requests frequently bypass these filters and reach hosts that appear dead to a standard ping.

---

## Task 7 — Host Discovery Using TCP and UDP

When ICMP is blocked entirely, TCP and UDP probes confirm liveness.

**TCP SYN Ping:**
```bash
nmap -PS22,80,443 -sn 192.168.1.0/24
```
Sends a SYN packet to specified ports. A SYN/ACK or RST response confirms the host is alive — even if the port is closed.

**TCP ACK Ping:**
```bash
nmap -PA80 -sn 192.168.1.0/24
```
Sends an ACK packet. Stateless firewalls often allow ACK packets through, making this useful for discovering hosts behind firewalls.

**UDP Ping:**
```bash
nmap -PU53,161 -sn 192.168.1.0/24
```
Sends to typically open UDP ports (DNS:53, SNMP:161). An ICMP Port Unreachable response confirms the host is alive.

**What I learned:** Combining multiple techniques (`-PS -PE -PP`) gives the most complete picture. A host that doesn't respond to ICMP may respond to TCP probes on port 80.

---

## Task 8 — Reverse-DNS Lookup

Reverse DNS resolves IP addresses back to hostnames — revealing information about a host's role.

```bash
nmap -R 192.168.1.0/24        # Force reverse DNS on all hosts
nmap -n 192.168.1.0/24        # Disable DNS (faster scans)
nmap --dns-servers 8.8.8.8 192.168.1.0/24  # Use specific DNS server
```

**Why hostnames matter in a pentest:**
- `dc01.corp.local` → likely a Domain Controller (high value target)
- `webserver-prod.company.com` → production web server (in scope check)
- `backup01.internal` → backup server (often misconfigured, high value)

**What I learned:** Never skip reverse DNS on internal engagements. Hostnames reveal network architecture and target priority far faster than banner grabbing individual services.

---

## Key Takeaways

- **Host discovery before port scanning is non-negotiable.** Running `-p-` against a /24 without first identifying live hosts wastes hours and creates unnecessary noise
- **No single technique works everywhere.** ARP for local, ICMP for remote, TCP/UDP when ICMP is blocked — know all three and combine them
- **`-sn` flag is your best friend** for pure host discovery without triggering port scan alerts
- **Hostnames from reverse DNS reveal more than the IP address alone** — always run `-R` on internal network engagements
- **Scope creep starts here.** Always use `-sL` to list and verify targets before scanning — one wrong CIDR range can put you outside legal scope

---

## Essential Nmap Host Discovery Commands

```bash
# Quick local discovery (ARP)
nmap -PR -sn 192.168.1.0/24

# Remote discovery (ICMP + TCP fallback)
nmap -PE -PS22,80,443 -sn 10.10.10.0/24

# Stealth — no ICMP, TCP ACK only
nmap -PA80,443 -sn 192.168.1.0/24

# Full combo — most thorough
nmap -PE -PP -PS21,22,23,25,80,443 -PA80,443 -PU53,161 -sn 192.168.1.0/24

# With reverse DNS + verbose output
nmap -R -v -sn 192.168.1.0/24
```

---

## Resources

- [Nmap Book — Host Discovery chapter (free)](https://nmap.org/book/man-host-discovery.html)
- [TryHackMe Nmap series — nmap01, nmap02, nmap03](https://tryhackme.com)

---

## My Progress

- [x] Guided Pentest: Web Application
- [x] Guided Pentest: Infrastructure
- [x] SQL Injection
- [x] CSRF Introduction
- [x] Nmap Live Host Discovery ← *this writeup*
- [x] Nmap Basic Port Scans
- [ ] XSS, Authentication — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
