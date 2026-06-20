# TryHackMe — Protocols and Servers 2

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Password%20Attacks-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Protocols and Servers 2 |
| URL | tryhackme.com/room/protocolsandservers2 |
| Path | Jr Penetration Tester → Network Reconnaissance |
| Tasks Completed | Task 1 — Task 7 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Sniffing Attack | ✅ Done |
| Task 3 | Man-in-the-Middle (MITM) Attack | ✅ Done |
| Task 4 | Transport Layer Security (TLS) | ✅ Done |
| Task 5 | Secure Shell (SSH) | ✅ Done |
| Task 6 | Password Attack | ✅ Done |
| Task 7 | Summary | ✅ Done |

---

## What I Learned (My Own Words)

In this room I learned how real-world network services work and how authentication plays a key role in securing them. I explored protocols like POP3 and IMAP, understanding how users log in and retrieve emails, and how weak passwords can be exploited. I practiced different password attack techniques such as dictionary attacks, brute force, and credential stuffing, and used tools like THC Hydra to automate login attempts against services like IMAP. I also understood the importance of using the attacker machine (like AttackBox) instead of the target system, and learned how modern defenses like MFA, rate limiting, and strong password policies help protect against these attacks.

---

## Task 2 — Sniffing Attack

Sniffing captures network traffic as it passes across a network segment — revealing anything sent in plaintext.

**Why plaintext protocols are vulnerable:**
- POP3, IMAP, FTP, Telnet, and HTTP all send data unencrypted
- Anyone on the same network segment (or with access to a router/switch in the path) can capture credentials with a basic packet sniffer

**Tools used for sniffing:**
```bash
# Wireshark — GUI packet capture and analysis
wireshark

# tcpdump — command-line packet capture
sudo tcpdump -i eth0 -w capture.pcap

# Filter for POP3/IMAP traffic specifically
tcpdump -i eth0 port 110 or port 143 -A
```

**What a sniffed POP3 login looks like:**
```
USER alice
PASS SuperSecret123
```
Both lines are sent and captured in plaintext — no encryption protects them on the wire.

**What I learned:** Sniffing requires no exploitation, no vulnerability, no clever attack — only network position. This is exactly why plaintext protocols are considered insecure by default, regardless of how strong the password is.

---

## Task 3 — Man-in-the-Middle (MITM) Attack

MITM goes further than passive sniffing — the attacker actively positions themselves between the client and server, intercepting and potentially modifying traffic in both directions.

**How MITM differs from simple sniffing:**

| | Sniffing | MITM |
|--|----------|------|
| Position | Passive listener | Active interceptor |
| Can modify traffic? | ❌ No | ✅ Yes |
| Requires network manipulation? | ❌ No (just visibility) | ✅ Yes (ARP spoofing, DNS spoofing) |
| Detectable? | Very hard | Easier (ARP table changes) |

**Common MITM techniques:**

```bash
# ARP spoofing — trick devices into sending traffic through attacker
arpspoof -i eth0 -t <target_ip> <gateway_ip>

# DNS spoofing — redirect domain lookups to attacker-controlled server
# (combined with a tool like ettercap or bettercap)
```

**Real-world impact:**
- Intercept and modify HTTP traffic in real time
- Inject malicious JavaScript into unencrypted web pages
- Capture credentials from any plaintext protocol passing through
- Downgrade HTTPS to HTTP (SSL stripping) if not properly defended against

**What I learned:** MITM is the practical demonstration of why plaintext protocols are dangerous — sniffing shows you can listen, MITM shows what an attacker can actually do with that position: modify, redirect, and inject.

---

## Task 4 — Transport Layer Security (TLS)

TLS is the encryption layer that protects HTTPS, and can be layered on top of other protocols (POP3S, IMAPS, SMTPS) to prevent the sniffing and MITM attacks from Tasks 2 and 3.

**How TLS prevents the previous attacks:**

| Attack | Plaintext protocol | TLS-protected equivalent |
|--------|--------------------|--------------------------|
| Credential sniffing | POP3 (110) — readable | POP3S (995) — encrypted |
| Credential sniffing | IMAP (143) — readable | IMAPS (993) — encrypted |
| MITM content injection | HTTP (80) — modifiable | HTTPS (443) — tamper-evident |

### TLS handshake basics

```
1. Client Hello       → client proposes supported TLS versions/ciphers
2. Server Hello       → server picks cipher, sends its certificate
3. Certificate check  → client verifies cert against trusted CA
4. Key exchange       → both sides derive a shared session key
5. Encrypted session  → all further traffic is encrypted
```

**Checking a server's TLS configuration:**
```bash
openssl s_client -connect target.com:443

# Check certificate details
openssl s_client -connect target.com:443 | openssl x509 -noout -text
```

**What I learned:** TLS does not just encrypt data — it also verifies server identity through certificates, which is what prevents a MITM attacker from successfully impersonating the real server (assuming the client properly validates the certificate chain).

---

## Task 5 — Secure Shell (SSH)

SSH is the encrypted replacement for Telnet — used for secure remote administration.

### SSH authentication methods

| Method | How it works | Security level |
|--------|--------------|-----------------|
| **Password authentication** | Username + password sent over encrypted channel | Medium — still vulnerable to brute force |
| **Public key authentication** | Client proves identity using a private key, server has matching public key | High — no password to steal or brute force |

### Basic SSH usage

```bash
# Connect with password auth
ssh user@10.10.10.1

# Connect with a specific private key
ssh -i private_key.pem user@10.10.10.1

# Generate a new SSH keypair
ssh-keygen -t ed25519 -f my_key

# Copy public key to a server for passwordless login
ssh-copy-id -i my_key.pub user@10.10.10.1
```

### Why SSH key authentication is stronger

- No password transmitted — nothing to capture even if traffic were somehow intercepted
- Private key never leaves the client machine
- Resistant to brute force — there's no password field to guess against

**What I learned:** Even though SSH is encrypted (solving the plaintext problem from Task 2), it can still be brute-forced if password authentication is enabled. This is the bridge into Task 6 — the encryption protects the data in transit, but the authentication mechanism itself can still be attacked.

---

## Task 6 — Password Attack

This task combined everything — using automated tools to attack authentication on the protocols studied earlier in the room.

### Password attack types

| Attack | Method | Best against |
|--------|--------|--------------|
| **Dictionary attack** | Try every password in a wordlist | Weak, common passwords |
| **Brute force** | Try every possible character combination | Short passwords, no complexity rules |
| **Credential stuffing** | Try leaked username/password pairs from other breaches | Password reuse across services |
| **Password spraying** | Try one common password against many usernames | Avoiding lockouts while still finding weak accounts |

### Using THC Hydra

Hydra automates login attempts against network services, including IMAP.

```bash
# Brute force IMAP login
hydra -l alice -P /usr/share/wordlists/rockyou.txt imap://10.10.10.1

# Brute force with username list + password list
hydra -L users.txt -P passwords.txt imap://10.10.10.1

# Target POP3 instead
hydra -l alice -P rockyou.txt pop3://10.10.10.1

# Target SSH
hydra -l root -P rockyou.txt ssh://10.10.10.1

# Specify number of parallel tasks (faster, but noisier)
hydra -l alice -P rockyou.txt -t 4 imap://10.10.10.1
```

### Important operational note

I learned the importance of always running Hydra (and any attack tool) from the **attacker machine (AttackBox)** — never from the target system itself. Running attack tools from the target risks:
- Detection by the target's own monitoring
- Accidentally locking yourself out
- Contaminating the test environment
- Confusing attacker-controlled vs target-controlled activity in logs

### Defenses against password attacks

| Defense | How it stops the attack |
|---------|-------------------------|
| **MFA (Multi-Factor Authentication)** | Password alone is no longer sufficient — attacker also needs the second factor |
| **Account lockout / rate limiting** | Locks account or delays response after N failed attempts — breaks automated brute force |
| **Strong password policy** | Minimum length + complexity defeats dictionary attacks |
| **CAPTCHA** | Prevents fully automated tools like Hydra from completing attempts |
| **IP-based blocking** | Blocks source IP after suspicious volume of failed attempts |
| **Unique passwords per service** | Defeats credential stuffing from other breaches |

**What I learned:** Hydra makes password attacks fast and automatic, but the room reinforced that the real lesson is defensive — MFA alone defeats the entire attack category, regardless of how weak the underlying password might be.

---

## Full Attack Chain — How This Room Connects

```
1. Task 2 — Sniff plaintext POP3/IMAP traffic → capture credentials directly
   (if MFA or TLS not in place)

2. Task 3 — Or position as MITM → intercept and modify traffic actively

3. Task 4 — TLS would have prevented both of the above
   (POP3S/IMAPS instead of POP3/IMAP)

4. Task 5 — SSH replaces Telnet, but password auth is still attackable

5. Task 6 — Hydra automates brute force against IMAP/POP3/SSH
   if no MFA, lockout, or rate limiting exists
```

**The defensive throughline:** every attack in this room is defeated by the same handful of controls — TLS encryption, MFA, and rate limiting. This is a recurring pattern across nearly all authentication-based attacks in cybersecurity.

---

## Key Takeaways

- **Plaintext protocols fail at the first step** — sniffing requires zero skill beyond network positioning. TLS is not optional for any service handling credentials
- **MITM is sniffing's more dangerous sibling** — the ability to modify traffic, not just read it, is what makes MITM a higher-severity finding in a pentest report
- **SSH solves transport encryption, not authentication strength** — a brute-forceable password is still a brute-forceable password, even inside an encrypted SSH session
- **Hydra is fast, but the real skill is choosing the right wordlist and service syntax** — `hydra -l user -P wordlist protocol://target` is a pattern I now know cold
- **Always attack from the AttackBox, never from the target** — this operational discipline matters in real engagements, not just THM rooms
- **MFA defeats this entire category of attack** — the single most effective recommendation in any pentest report involving password attacks is "implement MFA"

---

## Password Attack Quick Reference

```bash
# IMAP brute force
hydra -l USERNAME -P /usr/share/wordlists/rockyou.txt imap://TARGET

# POP3 brute force
hydra -l USERNAME -P rockyou.txt pop3://TARGET

# SSH brute force
hydra -l USERNAME -P rockyou.txt ssh://TARGET

# Username + password list combo
hydra -L users.txt -P passwords.txt PROTOCOL://TARGET

# Check TLS on a service
openssl s_client -connect TARGET:443
```

---

## Resources

- [THC Hydra GitHub](https://github.com/vanhauser-thc/thc-hydra)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [SecLists — wordlists for brute forcing](https://github.com/danielmiessler/SecLists)

---

## My Progress

- [x] Active Reconnaissance
- [x] Protocols and Servers
- [x] Protocols and Servers 2 ← *this writeup*
- [x] Nmap Advanced Port Scans
- [ ] Network Services rooms — coming next

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
