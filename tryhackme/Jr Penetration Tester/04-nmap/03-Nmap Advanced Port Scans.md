# TryHackMe — Nmap Advanced Port Scans

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Nmap%20%7C%20Evasion-informational)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Nmap Advanced Port Scans |
| URL | tryhackme.com/room/nmap03 |
| Path | Jr Penetration Tester → Nmap |
| Tasks Completed | Task 1 — Task 7 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | TCP Null Scan, FIN Scan, and Xmas Scan | ✅ Done |
| Task 3 | TCP Maimon Scan | ✅ Done |
| Task 4 | TCP ACK, Window, and Custom Scan | ✅ Done |
| Task 5 | Spoofing and Decoys | ✅ Done |
| Task 6 | Fragmented Packets | ✅ Done |
| Task 7 | Idle/Zombie Scan | ✅ Done |

---

## What I Learned (My Own Words)

In this room I learned how different TCP scan techniques help me understand a target system and its firewall behavior instead of just detecting open ports. I practiced scans like ACK, Window, Null, FIN, and Xmas, and saw how they are mainly used to map firewall rules by analyzing responses like RST or no response rather than directly identifying services. I also learned advanced techniques such as spoofing source IPs, decoy scans, packet fragmentation, idle (zombie) scanning, and how tools like `--reason` and verbosity levels give deeper insight into why Nmap makes certain conclusions. Overall, I understood that advanced Nmap scanning is less about simply finding open ports and more about analyzing network defenses, evasion techniques, and how systems react differently to crafted packets.

---

## The Core Shift in Thinking

Basic scans (`-sS`, `-sT` from the earlier Nmap rooms) answer: **"Is this port open?"**

Advanced scans answer a different question: **"What does the firewall do, and can I get information without it noticing?"**

This is the key mental shift this room teaches — these scans are not primarily about service discovery, they are about firewall mapping and evasion.

---

## Task 2 — TCP Null, FIN, and Xmas Scans

These three scans work on the same principle — sending packets with unusual TCP flag combinations to see how the target responds.

### How they work

| Scan | Flags set | Command |
|------|-----------|---------|
| **Null Scan** | No flags at all | `nmap -sN TARGET` |
| **FIN Scan** | Only the FIN flag | `nmap -sF TARGET` |
| **Xmas Scan** | FIN, PSH, and URG flags ("lit up like a Christmas tree") | `nmap -sX TARGET` |

### Interpreting responses (per RFC 793)

| Port state | Response received |
|-----------|-------------------|
| **Closed** | RST packet sent back |
| **Open or Filtered** | No response at all |

**Why this is confusing at first:** Unlike a SYN scan where open = clear SYN/ACK response, these scans give you **no response** for open ports. You only get a definitive answer for closed ports (RST). This is why the result shows as `open|filtered` — Nmap genuinely cannot distinguish the two without more information.

### Why use these scans at all

```bash
nmap -sN 10.10.10.1    # Null scan
nmap -sF 10.10.10.1    # FIN scan
nmap -sX 10.10.10.1    # Xmas scan
```

**Evasion value:** Many older or simply-configured firewalls only filter packets with the SYN flag set (because that's what "normal" connection attempts look like). A Null, FIN, or Xmas scan can sometimes slip past these firewalls because the unusual flag combination isn't what the filter is looking for.

**Limitation:** Modern stateful firewalls and most current operating systems (including Windows) don't follow RFC 793 behavior precisely, making these scans unreliable against modern targets. They work best against older Unix-based systems and basic packet-filtering firewalls.

**What I learned:** These scans are not about which is "best" — they are about variety. If a SYN scan gets blocked, switching scan types changes the packet signature and might bypass a poorly-configured filter that's only watching for SYN packets specifically.

---

## Task 3 — TCP Maimon Scan

The Maimon scan is a lesser-known variant, named after its discoverer Uriel Maimon.

```bash
nmap -sM 10.10.10.1
```

### How it differs

The Maimon scan sends a packet with both **FIN and ACK** flags set together. On certain BSD-derived systems, this combination causes the kernel to drop the packet for open ports (no response) while closed ports still respond with RST — similar pattern to the Null/FIN/Xmas family, but targeting a different class of systems.

**Best used against:** Older BSD-derived network stacks specifically. Largely a historical/niche technique today, but useful to know it exists and why it was designed.

**What I learned:** Each of these unusual scan types was designed around a specific quirk in how certain operating system network stacks implement the TCP specification. Modern systems have mostly patched these quirks, but understanding why each scan was invented teaches you to think about TCP/IP implementation differences as an attack surface in itself.

---

## Task 4 — TCP ACK, Window, and Custom Scans

These scans serve a fundamentally different purpose from everything above — they are not for finding open ports, they are specifically for **mapping firewall rules**.

### ACK Scan

```bash
nmap -sA 10.10.10.1
```

Sends only an ACK packet — never tries to establish a real connection.

| Response | Meaning |
|----------|---------|
| RST received | Port is **unfiltered** (not blocked by a stateful firewall — but says nothing about open/closed) |
| No response | Port is **filtered** (a stateful firewall is dropping the packet) |

**Why this matters:** ACK scans cannot tell you if a port is open — only whether a firewall is actively filtering it. This is purely a firewall-mapping tool, not a port-discovery tool.

### Window Scan

```bash
nmap -sW 10.10.10.1
```

Same packet structure as the ACK scan, but examines the **TCP window size** in the RST response. On some systems, open ports return a non-zero window size while closed ports return zero — giving Window scan slightly more information than a plain ACK scan, on systems where this quirk exists.

### Custom Scan (manual flag control)

```bash
nmap --scanflags SYNFIN 10.10.10.1
nmap --scanflags URGACKPSHRSTSYNFIN 10.10.10.1
```

Lets you manually specify exactly which TCP flags to set — useful for crafting scans that don't match any "standard" signature a firewall or IDS might be looking for.

**What I learned:** ACK and Window scans completely reframed how I think about scanning — they exist specifically to answer "is there a firewall here, and what is it doing?" rather than "what's running here?" This is a distinct reconnaissance goal from service discovery.

---

## Task 5 — Spoofing and Decoys

This task covers techniques to obscure which IP address is actually performing the scan.

### Decoy scanning

```bash
nmap -D decoy1,decoy2,decoy3,ME 10.10.10.1

# Use random decoys
nmap -D RND:5 10.10.10.1
```

Sends scan packets that appear to come from multiple source IPs simultaneously — the real attacker IP is hidden among the decoys, making it harder for the target to know which IP is the actual scanner.

### Source IP spoofing

```bash
nmap -S SPOOFED_IP -e eth0 -Pn 10.10.10.1
```

Forces Nmap to use a specified source IP rather than your real one. **Important limitation:** since the response goes to the spoofed IP, not back to you, this technique only works if you control the spoofed address or are simply trying to cause confusion/noise rather than receive scan results yourself.

### MAC address spoofing

```bash
nmap --spoof-mac 0 10.10.10.1          # Random MAC
nmap --spoof-mac Apple 10.10.10.1      # Vendor-specific MAC
```

**What I learned:** Decoy scanning is more practically useful than IP spoofing in most cases — it doesn't break the response path back to you, but still adds noise that makes attribution harder for a defender reviewing logs.

---

## Task 6 — Fragmented Packets

Packet fragmentation splits a single scan packet into multiple smaller pieces, which are reassembled by the target's network stack.

```bash
nmap -f 10.10.10.1              # Basic fragmentation
nmap -f -f 10.10.10.1           # Smaller fragments (double -f)
nmap --mtu 24 10.10.10.1        # Custom fragment size (must be multiple of 8)
```

**Why this can evade detection:** Some older or simpler IDS/firewall systems inspect packets individually and may fail to properly reassemble and analyze fragmented packets — letting malicious payloads or scan signatures slip through in pieces that look harmless individually.

**Limitation:** Modern firewalls and IDS systems generally perform packet reassembly before inspection specifically to counter this technique, making fragmentation less effective against current security infrastructure — but still a relevant historical and conceptual technique to understand.

**What I learned:** Fragmentation is conceptually similar to the unusual-flag scans — it works by exploiting a gap between how the network stack processes traffic and how the security control processes traffic. The pattern across all evasion techniques in this room is finding inconsistencies between the target's network stack and any inspection point in front of it.

---

## Task 7 — Idle/Zombie Scan

This is the most advanced technique in the room — scanning a target **without sending a single packet from your own IP address.**

### How a Zombie Scan works

```bash
nmap -sI ZOMBIE_IP TARGET_IP
```

**The mechanism, step by step:**

```
1. Attacker probes the "zombie" host to check its current IP ID sequence number
2. Attacker sends a SYN packet to the TARGET, spoofed to appear from the ZOMBIE's IP
3. If the target port is OPEN:
     → Target sends SYN/ACK to the zombie (not the attacker)
     → Zombie didn't expect this, responds with RST
     → Zombie's IP ID counter increments by 1
4. If the target port is CLOSED:
     → Target sends RST to the zombie
     → Zombie ignores it (no response, no increment)
5. Attacker probes the zombie's IP ID again
     → If it increased by 2 (1 from attacker's own probe, 1 from the RST) → port is OPEN
     → If it only increased by 1 (just from attacker's own probe) → port is CLOSED
```

**Requirements for a usable zombie host:**
- Must be idle (low/no other network traffic — otherwise IP ID changes become noise)
- Must use predictable, incremental IP ID sequence numbers (many modern OSes randomize this, which breaks the technique)

### Finding a suitable zombie

```bash
nmap -O 10.10.10.5    # Check OS and IP ID sequence generation behavior
```

**Why this is significant:** The scan appears to come entirely from the zombie's IP address. The target's logs show the zombie as the source — the real attacker's IP never appears anywhere in the target's records.

**What I learned:** This is the clearest demonstration in the entire Nmap series of "thinking in side-channels" — the attacker never directly asks the target anything. Instead, they infer the answer by watching how a third, uninvolved system's behavior changes. This pattern of inferring information through an indirect side-channel shows up again in much more advanced attacks later in a security career.

---

## Diagnostic Flags Used Throughout

| Flag | Purpose |
|------|---------|
| `--reason` | Shows exactly why Nmap concluded a port is open/closed/filtered (which packet triggered the verdict) |
| `-v` / `-vv` | Increases verbosity — more detail on what Nmap is doing in real time |
| `-d` / `-dd` | Debug output — even more detail, useful for understanding scan mechanics |
| `--packet-trace` | Shows every packet sent and received — the most granular view available |

```bash
nmap -sA --reason -v 10.10.10.1
```

**What I learned:** `--reason` turned this room from memorizing scan types into actually understanding them. Seeing exactly which response (RST, no-response, SYN/ACK) Nmap used to reach its conclusion made the theory concrete instead of abstract.

---

## Comparison Table — All Advanced Scans

| Scan | Flags | Primary purpose | Open port response | Closed port response |
|------|-------|-----------------|--------------------|-----------------------|
| Null | None | Firewall/IDS evasion | No response | RST |
| FIN | FIN | Firewall/IDS evasion | No response | RST |
| Xmas | FIN+PSH+URG | Firewall/IDS evasion | No response | RST |
| Maimon | FIN+ACK | Evasion (BSD-specific) | No response | RST |
| ACK | ACK | Firewall rule mapping | Unfiltered (RST) | N/A — can't tell open/closed |
| Window | ACK | Firewall mapping + open/closed inference | Non-zero window | Zero window |
| Idle/Zombie | SYN (spoofed) | Total source anonymity | Zombie IP ID +2 | Zombie IP ID +1 |

---

## Key Takeaways

- **Advanced scans answer "what is the firewall doing" not "what is open"** — this reframing is the single most important lesson in the room
- **Every evasion technique exploits a gap between RFC specification and real-world implementation** — Null/FIN/Xmas exploit flag-handling quirks, fragmentation exploits reassembly gaps, zombie scans exploit predictable IP ID sequences
- **Modern systems have closed most of these gaps** — these techniques are far less reliable against current Windows/Linux systems and modern stateful firewalls, but remain essential to understand conceptually and still occasionally useful against legacy infrastructure
- **`--reason` makes the invisible visible** — always add this flag when learning or troubleshooting unexpected scan results
- **Zombie scanning is the purest form of reconnaissance stealth** — no packet ever leaves with the attacker's real IP as the source, demonstrating a side-channel inference technique that appears throughout more advanced security topics

---

## Advanced Nmap Quick Reference

```bash
nmap -sN TARGET                          # Null scan
nmap -sF TARGET                          # FIN scan
nmap -sX TARGET                          # Xmas scan
nmap -sM TARGET                          # Maimon scan
nmap -sA TARGET                          # ACK scan (firewall mapping)
nmap -sW TARGET                          # Window scan
nmap -D RND:5 TARGET                     # Decoy scan, 5 random decoys
nmap -f TARGET                           # Fragment packets
nmap -sI ZOMBIE_IP TARGET                # Idle/Zombie scan
nmap --reason -v TARGET                  # Show reasoning behind results
```

---

## Resources

- [Nmap Book — Firewall/IDS Evasion (free)](https://nmap.org/book/man-bypass-firewalls-ids.html)
- [RFC 793 — TCP Specification](https://datatracker.ietf.org/doc/html/rfc793)
- [Nmap Idle Scan paper](https://nmap.org/book/idlescan.html)

---

## My Progress

- [x] Active Reconnaissance
- [x] Protocols and Servers
- [x] Protocols and Servers 2
- [x] Nmap Live Host Discovery
- [x] Nmap Basic Port Scans
- [x] Nmap Advanced Port Scans ← *this writeup*
- [ ] Network Services rooms — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
