# TryHackMe — Nmap Post Port Scans

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Nmap%20%7C%20Service%20Enumeration-informational)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Nmap Post Port Scans |
| URL | tryhackme.com/room/nmappostportscans |
| Path | Jr Penetration Tester → Nmap |
| Tasks Completed | Task 1 — Task 6 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Service Detection | ✅ Done |
| Task 3 | OS Detection and Traceroute | ✅ Done |
| Task 4 | Nmap Scripting Engine (NSE) | ✅ Done |
| Task 5 | Saving the Output | ✅ Done |
| Task 6 | Summary | ✅ Done |

---

## What I Learned (My Own Words)

In this room I learned how to interpret and analyze scan results after discovering open ports. I practiced identifying service versions using `-sV`, performing OS detection with `-O`, and using default NSE scripts with `-sC` to gather deeper information like service banners, SSH keys, HTTP titles, and SMTP/IMAP capabilities. I also learned how Nmap determines operating systems based on network behavior and how to use additional options like `--reason`, verbosity levels, and traceroute to better understand scan results and network paths. Finally, I explored different output formats such as normal, grepable, XML, and script kiddie, which help in organizing and processing scan data efficiently for further analysis.

---

## Why This Room Matters — The Missing Link

My earlier Nmap rooms (Live Host Discovery, Basic Port Scans, Advanced Port Scans) all answer **"is this port open, and is there a firewall in the way?"**

This room answers a completely different question: **"now that I know a port is open — what exactly is running there, and how do I extract maximum information before moving to exploitation?"**

This is the step most beginners skip — they find an open port and jump straight to searching for exploits. This room teaches the discipline of fully enumerating before attacking.

---

## Task 2 — Service Detection

Service detection identifies exactly what software (and version) is running behind an open port — not just that the port responds.

```bash
nmap -sV 10.10.10.1                  # Basic version detection
nmap -sV --version-intensity 9 10.10.10.1   # Maximum intensity (0-9, slower but thorough)
nmap -sV --version-light 10.10.10.1  # Faster, less thorough (intensity 2)
```

### How -sV works internally

Nmap sends a series of probes to the open port and compares the response against a database of thousands of known service signatures (`nmap-service-probes` file). Unlike simple banner grabbing (just reading what the service announces), `-sV` actively probes the service with different requests to confirm the version, since not all services announce themselves clearly on connection.

### Sample output

```
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.29 ((Ubuntu))
3306/tcp open  mysql       MySQL 5.7.29-0ubuntu0.18.04.1
```

**Why this matters for exploitation:** Every line in that output is searchable. `OpenSSH 7.6p1` and `Apache httpd 2.4.29` go straight into a CVE search. Without `-sV`, I'd only know "port 22 is open" — with it, I know exactly what to research.

**What I learned:** Version detection intensity is a tradeoff. Higher intensity (`--version-intensity 9`) sends more probes and gets more accurate results but takes longer and is noisier on the network. For a stealthy engagement, lower intensity makes sense; for a thorough internal assessment, maximum intensity is worth the time.

---

## Task 3 — OS Detection and Traceroute

### OS Detection

```bash
nmap -O 10.10.10.1                   # OS detection
nmap -O --osscan-guess 10.10.10.1    # More aggressive guessing for unclear results
nmap -O --osscan-limit 10.10.10.1    # Only attempt OS detection on promising hosts
```

### How Nmap determines the OS

Nmap doesn't ask the target what OS it's running — it infers this by analyzing subtle differences in how different operating systems implement the TCP/IP stack. This connects directly to what I learned in the Advanced Port Scans room about RFC 793 implementation quirks.

**Signals Nmap analyzes:**

| Signal | What it reveals |
|--------|-----------------|
| **TTL (Time To Live)** | Linux defaults to 64, Windows to 128, some network devices to 255 |
| **TCP window size** | Different OSes use different default window sizes |
| **TCP options ordering** | The order and presence of TCP options varies by OS implementation |
| **ICMP response quirks** | How the OS responds to malformed/unusual ICMP packets |
| **IP ID sequence generation** | Predictable vs randomized — also relevant to the Idle/Zombie scan from the previous room |

### Sample output

```
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.4
Network Distance: 2 hops
```

### Traceroute integration

```bash
nmap -O --traceroute 10.10.10.1
```

Combining OS detection with traceroute shows both what the target is running AND how many network hops separate me from it — useful for understanding whether I'm on the same local network or scanning across the internet/internal routing.

**What I learned:** OS detection confidence varies — Nmap will sometimes give a single confident guess and other times list multiple possibilities with percentage confidence. This is honest uncertainty, not a flaw — TCP/IP fingerprinting is inherently probabilistic, especially against hardened or unusual network stacks.

---

## Task 4 — Nmap Scripting Engine (NSE)

NSE is what transforms Nmap from a port scanner into a full reconnaissance and light vulnerability-assessment tool.

### Running default scripts

```bash
nmap -sC 10.10.10.1                  # Run default safe scripts
nmap -sV -sC 10.10.10.1              # Combine with version detection (very common pairing)
```

### NSE script categories

| Category | Purpose | Example use |
|----------|---------|-------------|
| `default` | Safe, commonly useful scripts (`-sC` runs this set) | General enumeration |
| `vuln` | Checks for known vulnerabilities | `--script vuln` |
| `auth` | Authentication-related testing | Checking for default credentials |
| `brute` | Brute-forcing credentials | Login attacks (use carefully) |
| `discovery` | Extra information gathering | DNS, SNMP, more |
| `safe` | Scripts that won't crash or disrupt the target | Safe for production scanning |

### Specific NSE scripts I used

```bash
# Grab SSH host keys
nmap --script ssh-hostkey -p 22 10.10.10.1

# Get the HTTP page title
nmap --script http-title -p 80 10.10.10.1

# Check SMTP capabilities
nmap --script smtp-commands -p 25 10.10.10.1

# Check IMAP capabilities
nmap --script imap-capabilities -p 143 10.10.10.1

# Run a vulnerability-focused scan
nmap --script vuln 10.10.10.1

# Run multiple specific scripts together
nmap --script "http-title,http-headers,http-methods" -p 80 10.10.10.1
```

### What each script revealed

| Script | Information extracted |
|--------|----------------------|
| `ssh-hostkey` | SSH server's public key fingerprint — useful for detecting MITM later, or identifying the same host across different IPs |
| `http-title` | The `<title>` tag of a web page without opening a browser — quick way to identify what application is running |
| `smtp-commands` | Lists every SMTP command the mail server supports — reveals VRFY/EXPN availability (relevant to my Protocols and Servers room) |
| `imap-capabilities` | Lists IMAP extensions supported — version and feature fingerprinting |

**What I learned:** NSE scripts turn a single Nmap command into the equivalent of manually connecting with Telnet and running protocol commands by hand — which is exactly what I did manually in the Active Reconnaissance and Protocols and Servers rooms. NSE automates that manual process at scale across many ports and hosts simultaneously.

---

## Task 5 — Saving the Output

Professional engagements require documented, reproducible scan results — not just terminal output that disappears.

### Output formats

```bash
nmap -oN scan_normal.txt 10.10.10.1      # Normal — human readable, same as terminal
nmap -oG scan_grepable.txt 10.10.10.1    # Grepable — one line per host, easy for grep/awk/scripts
nmap -oX scan_output.xml 10.10.10.1      # XML — structured, parseable by other tools
nmap -oS scan_script_kiddie.txt 10.10.10.1  # "Script kiddie" — leetspeak output (novelty format)

# Save in all three useful formats simultaneously
nmap -oA scan_results 10.10.10.1
# → creates scan_results.nmap, scan_results.gnmap, scan_results.xml
```

### Why each format matters

| Format | Best for |
|--------|----------|
| **Normal (-oN)** | Reading results yourself, including in a report |
| **Grepable (-oG)** | Quickly searching results with `grep`, `awk`, `cut` — useful for scripting against many hosts |
| **XML (-oX)** | Feeding into other tools — Metasploit can import Nmap XML directly, as can many vulnerability management platforms |
| **-oA (all)** | Standard professional practice — always save all three so the right format is available later regardless of need |

### Practical example

```bash
# Find all hosts with port 80 open from a grepable scan
grep "80/open" scan_grepable.txt

# Import XML results into Metasploit
db_import scan_output.xml
```

**What I learned:** `-oA` should be a habit on every single scan from now on, even in practice rooms. Building this habit now means I never lose scan results, and on a real engagement I always have documentation ready for the final report without having to re-run scans.

---

## Diagnostic Options Reinforced From Previous Rooms

```bash
nmap -sV -sC -O --reason -v 10.10.10.1
```

| Flag | What it adds |
|------|--------------|
| `--reason` | Shows exactly which packet/response led to each conclusion |
| `-v` / `-vv` | Increases detail shown during the scan, including real-time progress |
| `--traceroute` | Adds network path information alongside service/OS data |

**Why combining these matters:** A single comprehensive command like `nmap -sV -sC -O --reason -oA full_scan TARGET` replaces what used to take five separate scans across three different rooms' worth of techniques.

---

## My Complete Post-Discovery Workflow

Combining everything from this room into the standard command I now run after host discovery:

```bash
sudo nmap -sV -sC -O --reason -oA initial_scan 10.10.10.1
```

This single command:
1. Identifies exact service versions (`-sV`)
2. Runs default NSE scripts for deeper enumeration (`-sC`)
3. Attempts OS fingerprinting (`-O`)
4. Explains why each conclusion was reached (`--reason`)
5. Saves results in all three useful formats (`-oA`)

**Follow-up for web services specifically:**
```bash
nmap --script "http-title,http-headers,http-methods,http-enum" -p 80,443 10.10.10.1
```

**Follow-up for deeper vulnerability checking:**
```bash
nmap --script vuln 10.10.10.1
```

---

## Key Takeaways

- **This room is the bridge between scanning and exploitation** — everything before this tells you what's open; this room tells you what to actually research and attack
- **`-sV -sC -O` together is the standard professional opening scan** — I now run this combination by default rather than three separate basic scans
- **NSE automates what I learned to do manually** — the Active Reconnaissance and Protocols and Servers rooms taught me to manually grab banners with Telnet; NSE does the same thing automatically and at scale
- **`-oA` should never be skipped** — saving every scan in all formats costs nothing and means documentation is never an afterthought
- **OS detection is probabilistic, not certain** — confidence percentages matter; treat low-confidence OS guesses as a hint to verify with other techniques, not a confirmed fact

---

## Full Nmap Series — My Complete Picture

| Room | What it taught | Question it answers |
|------|----------------|---------------------|
| Live Host Discovery | ARP, ICMP, TCP/UDP discovery | Is anything alive on this network? |
| Basic Port Scans | TCP Connect, SYN, UDP scans | Which ports are open? |
| Advanced Port Scans | Null/FIN/Xmas, ACK, decoys, zombie | What is the firewall doing, and can I hide? |
| **Post Port Scans** | Service/OS detection, NSE, output formats | What exactly is running, and how do I document it? |

**What I learned across the full series:** Reconnaissance is layered — each room builds on the last. Discovery tells me what's alive, port scanning tells me what's open, advanced scanning tells me about the defenses in the way, and post-scan analysis tells me exactly what I'm looking at and gives me a documented record to act on.

---

## Quick Reference — Post-Scan Commands

```bash
nmap -sV TARGET                              # Service version detection
nmap -O TARGET                                # OS detection
nmap -sC TARGET                               # Default NSE scripts
nmap --script vuln TARGET                     # Vulnerability-focused NSE scripts
nmap --script ssh-hostkey -p 22 TARGET        # Specific script example
nmap -oA results TARGET                       # Save in all formats
nmap -sV -sC -O --reason -oA full TARGET      # My standard opening scan
```

---

## Resources

- [Nmap Book — Service/Version Detection (free)](https://nmap.org/book/vscan.html)
- [Nmap Book — OS Detection (free)](https://nmap.org/book/osdetect.html)
- [NSE Script Library](https://nmap.org/nsedoc/)
- [Nmap Book — Output formats](https://nmap.org/book/output.html)

---

## My Progress

- [x] Active Reconnaissance
- [x] Passive Reconnaissance
- [x] Protocols and Servers
- [x] Protocols and Servers 2
- [x] Nmap Live Host Discovery
- [x] Nmap Basic Port Scans
- [x] Nmap Advanced Port Scans
- [x] Nmap Post Port Scans ← *this writeup — Nmap series complete*
- [ ] Network Services rooms — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
