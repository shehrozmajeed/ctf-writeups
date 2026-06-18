# TryHackMe — Protocols and Servers

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Network%20Protocols-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Platform | TryHackMe |
| Room | Protocols and Servers |
| URL | tryhackme.com/room/protocolsandservers |
| Path | Jr Penetration Tester → Network Reconnaissance |
| Tasks Completed | Task 1 — Task 8 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Telnet | ✅ Done |
| Task 3 | Hypertext Transfer Protocol (HTTP) | ✅ Done |
| Task 4 | File Transfer Protocol (FTP) | ✅ Done |
| Task 5 | Simple Mail Transfer Protocol (SMTP) | ✅ Done |
| Task 6 | Post Office Protocol 3 (POP3) | ✅ Done |
| Task 7 | Internet Message Access Protocol (IMAP) | ✅ Done |
| Task 8 | Summary | ✅ Done |

---

## What I Learned (My Own Words)

In this room I learned how different network services work by communicating with them manually using Telnet and Netcat. Instead of using high-level tools that do everything automatically, I typed raw protocol commands directly to the servers. This gave me a real understanding of how these protocols work at a low level — and more importantly, where they are insecure.

**The biggest lesson:** Most of these protocols transmit data in **plaintext** — meaning anyone who can intercept network traffic can read usernames, passwords, emails, and file contents. This is why these services are such common pentest findings on internal networks.

---

## Protocol Quick Reference

| Protocol | Port | Purpose | Plaintext? | Secure version |
|----------|------|---------|-----------|----------------|
| Telnet | 23 | Remote terminal access | ✅ Yes — everything | SSH (port 22) |
| HTTP | 80 | Web browsing | ✅ Yes — all data | HTTPS (port 443) |
| FTP | 21 | File transfer | ✅ Yes — including passwords | SFTP / FTPS |
| SMTP | 25 | Sending email | ✅ Yes | SMTPS (port 465/587) |
| POP3 | 110 | Receiving email (download) | ✅ Yes | POP3S (port 995) |
| IMAP | 143 | Receiving email (sync) | ✅ Yes | IMAPS (port 993) |

---

## Task 2 — Telnet

Telnet is a remote terminal protocol that predates SSH. Everything sent over Telnet — including passwords — is transmitted in plaintext.

### What I did

I connected to a Telnet service and authenticated manually:

```bash
telnet 10.10.10.1 23

# Server prompts:
login: admin
Password: password123

# Everything above is visible to anyone sniffing the network
```

### Why Telnet is a security risk

- All traffic is unencrypted — Wireshark capture shows everything in plain text
- Credentials visible in packet captures
- Still found running on embedded devices, printers, and legacy systems in real networks

### Finding Telnet in a pentest

```bash
nmap -p 23 --open 10.10.10.0/24     # Find Telnet services on a subnet
telnet TARGET 23                      # Connect manually
```

**What I learned:** Telnet should never be running on a modern system. When found during a pentest, it is always a High finding because credential capture requires only a basic network sniff. Recommend replacing with SSH immediately.

---

## Task 3 — HTTP (Hypertext Transfer Protocol)

HTTP is the foundation of all web communication. Every web page request and response goes over HTTP (or HTTPS for encrypted).

### What I did

I sent raw HTTP requests manually using Telnet to understand exactly what a browser sends:

```bash
telnet 10.10.10.1 80

# Type this exactly:
GET / HTTP/1.1
Host: 10.10.10.1
[press Enter twice]
```

### HTTP response I received

```
HTTP/1.1 200 OK
Date: Thu, 19 Jun 2026 10:30:00 GMT
Server: Apache/2.4.41 (Ubuntu)          ← Server software + version
X-Powered-By: PHP/7.4.3                 ← Backend language + version
Content-Type: text/html; charset=UTF-8
Content-Length: 1234

<!DOCTYPE html>
<html>...page content...
```

### Key HTTP methods

| Method | Purpose | Security concern |
|--------|---------|-----------------|
| GET | Retrieve resource | Parameters in URL — logged everywhere |
| POST | Submit data | Body data — but still plaintext on HTTP |
| PUT | Upload/replace resource | Often misconfigured — allows file upload |
| DELETE | Remove resource | Often unprotected on misconfigured servers |
| OPTIONS | List allowed methods | Reveals attack surface |

### What server headers reveal

```bash
Server: Apache/2.4.41        → Search for Apache 2.4.41 CVEs
X-Powered-By: PHP/7.4.3      → Search for PHP 7.4.3 CVEs
X-AspNet-Version: 4.0.30319  → .NET version
```

**What I learned:** HTTP response headers leak exact software versions before I even look at the application. This is the first thing I check manually after finding port 80 open — the headers often give me my exploitation path.

---

## Task 4 — FTP (File Transfer Protocol)

FTP transfers files between client and server. Uses two connections — port 21 for commands, a separate port for data transfer.

### What I did

I connected to an FTP server manually and navigated it:

```bash
telnet 10.10.10.1 21

# Server response (banner):
220 ProFTPD 1.3.5 Server ready
# → Version immediately visible — searchable for CVEs

# Authenticate
USER anonymous
331 Password required
PASS anonymous@email.com
230 User logged in       ← Anonymous login allowed!

# List files
LIST
# → Shows directory contents

# Download a file
RETR filename.txt
```

### Anonymous FTP — common finding

Anonymous FTP login (username: `anonymous`, any password) is frequently found on internal networks and is always a finding in a pentest report.

```bash
# Test for anonymous FTP with Nmap script
nmap --script ftp-anon -p 21 10.10.10.1
```

### FTP security issues

| Issue | Risk |
|-------|------|
| Anonymous login enabled | Anyone can access files without credentials |
| Plaintext credentials | Username and password visible in packet capture |
| Writable directories | Attacker can upload files (web shells, malware) |
| Banner reveals version | CVE lookup immediately possible |
| Active mode FTP | Firewall bypass — server initiates data connection back to client |

**What I learned:** FTP is one of the most commonly misconfigured services on internal networks. Anonymous login + writable directory = file upload to web shell in a single session. Always test for this.

---

## Task 5 — SMTP (Simple Mail Transfer Protocol)

SMTP sends email between mail servers. Port 25 (server-to-server), 587 (client submission).

### What I did

I manually composed and sent an email using raw SMTP commands:

```bash
telnet 10.10.10.1 25

# Server banner:
220 mail.target.com ESMTP Postfix (Ubuntu)

# Introduce myself
EHLO attacker.com
250-mail.target.com

# Specify sender
MAIL FROM:<attacker@evil.com>

# Specify recipient
RCPT TO:<victim@target.com>

# Write the email
DATA
Subject: Test email
From: attacker@evil.com
To: victim@target.com

This is the email body.
.           ← Single dot on new line = end of email
250 Message queued

QUIT
```

### Email spoofing

By controlling the `MAIL FROM` and `From:` headers manually, I demonstrated how email spoofing works. The sending address in the email body can be set to anything — this is the technical basis for phishing emails.

### SMTP user enumeration

```bash
# VRFY command checks if a user exists
VRFY admin
252 admin@target.com       ← User exists

VRFY nonexistent
550 User unknown           ← User does not exist

# EXPN expands mailing list membership
EXPN marketing
250 alice@target.com
250 bob@target.com
```

**What I learned:** SMTP exposes two critical security issues — email spoofing (trivial to fake the sender address without SPF/DKIM) and user enumeration (VRFY command confirms valid email addresses, which feeds into phishing campaigns).

---

## Task 6 — POP3 (Post Office Protocol 3)

POP3 downloads emails from a server to a local device and (by default) deletes them from the server. Port 110.

### What I did

I accessed a mailbox manually using raw POP3 commands:

```bash
telnet 10.10.10.1 110

# Server greeting:
+OK POP3 server ready

# Authenticate
USER alice
+OK
PASS password123
+OK Logged in

# Check mailbox
STAT
+OK 3 1024    ← 3 messages, 1024 bytes total

# List messages
LIST
+OK
1 512
2 256
3 256

# Read message 1
RETR 1
+OK
From: boss@company.com
Subject: Confidential
[full email content in plaintext]

# Delete message 1
DELE 1

# Quit
QUIT
```

### Security issues with POP3

- Credentials sent in plaintext — captured in any network sniff
- Emails downloaded in plaintext — content visible to anyone monitoring
- No server-side copy after download — harder to audit if compromised

**What I learned:** If I capture POP3 traffic on a network, I get the username, password, and every email the user downloads — all in one capture. This is why email services must use POP3S (port 995) with TLS encryption.

---

## Task 7 — IMAP (Internet Message Access Protocol)

IMAP synchronises email across multiple devices — emails stay on the server. Port 143.

### What I did

I accessed a mailbox using raw IMAP commands:

```bash
telnet 10.10.10.1 143

# Server greeting:
* OK IMAP4 server ready

# Login (IMAP uses tags before each command)
A1 LOGIN alice password123
A1 OK Logged in

# List mailboxes
A2 LIST "" "*"
* LIST (\HasNoChildren) "." INBOX
* LIST (\HasNoChildren) "." Sent

# Select inbox
A3 SELECT INBOX
* 5 EXISTS      ← 5 emails in inbox

# Fetch email headers
A4 FETCH 1 BODY[HEADER]
* 1 FETCH (BODY[HEADER] {header content})

# Fetch full email
A5 FETCH 1 BODY[]
[full email in plaintext]

# Logout
A6 LOGOUT
```

### IMAP vs POP3

| | POP3 | IMAP |
|--|------|------|
| Emails stored | Downloaded to device, deleted from server | Stay on server, synced to all devices |
| Multiple devices | ❌ Poor — emails on one device | ✅ Good — same inbox everywhere |
| Offline access | ✅ Yes | Partial |
| Server storage required | Minimal | Yes |
| Security issues | Same plaintext risks | Same plaintext risks |

**What I learned:** IMAP lets an attacker browse the entire mailbox without downloading anything — perfect for quiet surveillance. Capturing IMAP credentials gives persistent access to someone's full email history as long as the password is unchanged.

---

## Plaintext Protocol Attack — Full Scenario

Here is how these protocols chain together in a real attack:

```
1. Nmap finds port 25 (SMTP) open on target mail server
2. VRFY command enumerates valid usernames: alice, bob, admin
3. Nmap finds port 110 (POP3) open on same server
4. Wireshark captures POP3 session → alice's password in plaintext
5. Log in to POP3 as alice → read all her emails
6. Find email with server credentials → pivot to next target
7. Use SMTP to send spoofed phishing email from alice@target.com
```

**Defender fix:** Replace all plaintext protocols with encrypted equivalents. POP3 → POP3S, IMAP → IMAPS, SMTP → SMTPS, HTTP → HTTPS, FTP → SFTP, Telnet → SSH.

---

## Key Takeaways

- **Plaintext protocols are an attacker's best friend** — one network capture session reveals credentials, email content, and file contents simultaneously
- **Banner grabbing is automatic** — every protocol announces its version on connection. That version number is the starting point for CVE research
- **SMTP VRFY is user enumeration** — always test this on exposed mail servers. Valid usernames feed directly into phishing and brute force attacks
- **Anonymous FTP is always a finding** — it should never be enabled on a production system. When found, document it as at least Medium severity
- **Knowing raw protocol commands separates real pentesters** — automated tools do this work, but understanding what they do manually means you can work around filters, test edge cases, and explain findings clearly in reports

---

## Protocol Commands Cheat Sheet

```bash
# HTTP
telnet TARGET 80
GET / HTTP/1.1
Host: TARGET
[blank line]

# FTP
telnet TARGET 21
USER anonymous
PASS test@test.com
LIST
RETR filename

# SMTP
telnet TARGET 25
EHLO attacker.com
VRFY username
MAIL FROM:<test@test.com>
RCPT TO:<victim@target.com>
DATA
[message]
.

# POP3
telnet TARGET 110
USER username
PASS password
STAT
LIST
RETR 1

# IMAP
telnet TARGET 143
A1 LOGIN username password
A2 LIST "" "*"
A3 SELECT INBOX
A4 FETCH 1 BODY[]
```

---

## Resources

- [RFC 2821 — SMTP](https://tools.ietf.org/html/rfc2821)
- [RFC 1939 — POP3](https://tools.ietf.org/html/rfc1939)
- [RFC 3501 — IMAP](https://tools.ietf.org/html/rfc3501)
- [HackTricks — Network Services](https://book.hacktricks.xyz/network-services-pentesting)

---

## My Progress

- [x] Active Reconnaissance
- [x] Protocols and Servers ← *this writeup*
- [ ] Nmap Advanced rooms (nmap03, nmap04)
- [ ] Network Services rooms

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
