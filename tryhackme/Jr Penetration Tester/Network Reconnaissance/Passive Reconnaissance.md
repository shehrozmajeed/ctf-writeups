# TryHackMe — Passive Reconnaissance

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
| Room | Passive Reconnaissance |
| URL | tryhackme.com/room/passiverecon |
| Path | Jr Penetration Tester → Network Reconnaissance |
| Tasks Completed | Task 1 — Task 7 (all, 100%) |
| Date Completed | June 2026 |

---

## Tasks Completed

| Task | Topic | Status |
|------|-------|--------|
| Task 1 | Introduction | ✅ Done |
| Task 2 | Passive Versus Active Recon | ✅ Done |
| Task 3 | Whois | ✅ Done |
| Task 4 | nslookup and dig | ✅ Done |
| Task 5 | DNSDumpster | ✅ Done |
| Task 6 | Shodan.io | ✅ Done |
| Task 7 | Summary | ✅ Done |

---

## Task 2 — Passive vs Active Recon

This is the most important distinction in the recon phase of any engagement.

| | Passive Recon | Active Recon |
|--|---------------|-------------|
| **Direct contact with target?** | ❌ No | ✅ Yes |
| **Detectable by target?** | ❌ No | ✅ Yes |
| **Legal without permission?** | ✅ Generally yes | ❌ No |
| **Examples** | WHOIS, Google, Shodan, DNS | Nmap, Ping, Telnet, Netcat |
| **When to use** | Before engagement begins | After written permission obtained |

**Key rule:** Passive recon uses only publicly available information — you never send a single packet to the target. Active recon directly interacts with target systems and requires written authorisation.

**Why passive recon matters:**
- Reveals attack surface before touching anything
- Completely undetectable — no logs generated on the target
- Often reveals more than active scanning (employee names, email formats, leaked credentials, tech stack)

---

## Task 3 — Whois

WHOIS is a public database that stores registration information about domain names and IP addresses.

### What WHOIS reveals

```bash
whois tryhackme.com
```

**Sample output fields:**

| Field | What it tells you |
|-------|------------------|
| `Registrar` | Where the domain was purchased (GoDaddy, Namecheap, etc.) |
| `Creation Date` | When the domain was registered — old domains are more trusted |
| `Updated Date` | Last modification |
| `Expiry Date` | When it expires — expiring domains can be hijacked |
| `Registrant Name` | Owner name (often hidden by privacy protection) |
| `Registrant Email` | Contact email — phishing target, or reveals real owner |
| `Name Servers` | DNS servers used — reveals hosting provider |
| `Registrant Country` | Jurisdiction information |

### Running WHOIS

```bash
# Command line
whois target.com
whois 10.10.10.1        # Reverse WHOIS on IP address

# Online tools (no installation needed)
# whois.domaintools.com
# lookup.icann.org
```

### What I found during the room

- Domain registration details revealed the hosting provider
- Name servers identified the DNS infrastructure
- Registrant email (when not privacy-protected) is a direct phishing target
- IP block ownership via reverse WHOIS showed the hosting company

**What I learned:** WHOIS is always the first tool I run on a new target domain. It takes 10 seconds and often reveals the hosting provider, DNS infrastructure, and sometimes real contact details — all before touching the target.

---

## Task 4 — nslookup and dig

DNS (Domain Name System) translates domain names to IP addresses. DNS queries are passive — you are querying public DNS servers, not the target.

### nslookup

```bash
# Basic A record lookup (domain → IP)
nslookup tryhackme.com

# Lookup specific record type
nslookup -type=A tryhackme.com        # IPv4 address
nslookup -type=AAAA tryhackme.com     # IPv6 address
nslookup -type=MX tryhackme.com       # Mail servers
nslookup -type=TXT tryhackme.com      # Text records (SPF, DKIM, verification)
nslookup -type=NS tryhackme.com       # Name servers
nslookup -type=CNAME tryhackme.com    # Canonical name (alias)

# Query a specific DNS server
nslookup tryhackme.com 8.8.8.8        # Use Google's DNS
```

### dig (more powerful than nslookup)

```bash
# Basic lookup
dig tryhackme.com

# Specific record types
dig tryhackme.com A
dig tryhackme.com MX
dig tryhackme.com TXT
dig tryhackme.com NS
dig tryhackme.com ANY      # All records

# Short output (just the answer)
dig tryhackme.com A +short

# Query specific DNS server
dig @8.8.8.8 tryhackme.com

# Reverse DNS (IP → domain)
dig -x 10.10.10.1
```

### DNS Record Types — What Each Reveals

| Record | Purpose | Why pentesters care |
|--------|---------|---------------------|
| **A** | Domain → IPv4 address | Direct IP of server |
| **AAAA** | Domain → IPv6 address | IPv6 attack surface |
| **MX** | Mail server for domain | Email infrastructure, phishing routes |
| **TXT** | Arbitrary text | SPF policy, DKIM keys, sometimes internal info |
| **NS** | Name servers | DNS provider, potential zone transfer target |
| **CNAME** | Alias to another domain | Reveals hidden subdomains or cloud services |
| **SOA** | Start of Authority | Primary DNS server, admin email |
| **PTR** | IP → domain (reverse) | Reveals hostnames from IP addresses |

### What I found during the room

```bash
dig tryhackme.com MX
# → Revealed mail provider (Google Workspace / Microsoft 365)

dig tryhackme.com TXT
# → SPF record showed all authorised mail senders
# → Verification tokens revealed services in use (Google, Atlassian, etc.)

dig tryhackme.com NS
# → Name servers revealed DNS hosting provider
```

**What I learned:** MX records reveal the email provider, which tells me whether phishing attempts will pass SPF checks. TXT records often leak what cloud services a company uses — Atlassian, Salesforce, and similar services add TXT verification records that are publicly visible.

---

## Task 5 — DNSDumpster

DNSDumpster is a free online tool that maps a domain's full DNS infrastructure — finding subdomains, IP ranges, and connected services automatically.

**URL:** dnsdumpster.com

### What DNSDumpster does

- Queries multiple DNS sources simultaneously
- Finds subdomains not visible in a simple dig query
- Maps relationships between domains and IP addresses visually
- Identifies hosting providers for each discovered host
- Shows geolocation of discovered servers

### What I found during the room

Running a target domain through DNSDumpster revealed:

```
Main domain:    target.com → 10.10.10.1 (CloudFlare CDN)
Subdomains:
  mail.target.com          → 10.10.20.5 (Microsoft Exchange)
  vpn.target.com           → 10.10.30.2 (Cisco ASA)
  dev.target.com           → 10.10.40.8 (AWS EC2)
  staging.target.com       → 10.10.40.9 (AWS EC2)
  admin.target.com         → 10.10.50.1 (direct IP — no CDN)
```

**Why this matters:**
- `dev.target.com` and `staging.target.com` are often less hardened than production
- `admin.target.com` without CDN means the real server IP is exposed
- `vpn.target.com` confirms VPN software — searchable for CVEs
- `mail.target.com` with Exchange is a common exploitation target

**What I learned:** DNSDumpster in 60 seconds gives me a complete picture of a company's external attack surface — subdomains, hosting providers, and IP addresses — all from public DNS data without touching the target once.

---

## Task 6 — Shodan.io

Shodan is a search engine for internet-connected devices. While Google indexes web pages, Shodan indexes open ports, services, and banners from every IP address on the internet.

**URL:** shodan.io

### What Shodan finds

- Open ports and services on any IP address
- Software versions and banners (same as manual banner grabbing)
- SSL certificate details and expiry
- Geographic location of servers
- Whether a host has known CVEs
- Default credentials on devices

### How I used Shodan during the room

```
Search: hostname:target.com
→ Shows all IPs Shodan has scanned for that domain

Search: org:"Company Name"
→ Shows all IPs registered to that company

Search: ip:10.10.10.1
→ Shows open ports, services, banners for that specific IP
```

### Shodan search filters

| Filter | Example | What it finds |
|--------|---------|---------------|
| `hostname:` | `hostname:target.com` | All IPs for a domain |
| `org:` | `org:"Company Ltd"` | All IPs owned by an organisation |
| `country:` | `country:PK` | Devices in Pakistan |
| `port:` | `port:22` | Devices with SSH open |
| `product:` | `product:Apache` | Apache web servers |
| `vuln:` | `vuln:CVE-2021-44228` | Devices vulnerable to Log4Shell |
| `default password` | — | Devices with default credentials |

### What I found during the room

- Shodan showed the exact Apache version running — matched what Telnet banner grab found
- SSL certificate revealed additional subdomains (SANs — Subject Alternative Names)
- Shodan flagged the host as potentially vulnerable to a known CVE based on its version

**What I learned:** Shodan does banner grabbing at internet scale — passively. Before I even connect to a target, Shodan may already have the exact service versions, open ports, and CVE flags. It is the fastest way to answer "what is running on this IP?" without touching the target at all.

---

## Complete Passive Recon Workflow

Here is the exact order I now follow at the start of every engagement:

```
Step 1 → whois target.com
         What: Registration info, hosting provider, name servers
         Time: 30 seconds

Step 2 → dig target.com ANY +short
         What: All DNS records — IPs, mail servers, TXT records
         Time: 30 seconds

Step 3 → dnsdumpster.com → search target.com
         What: All subdomains, visual infrastructure map
         Time: 2 minutes

Step 4 → shodan.io → search hostname:target.com
         What: Open ports, service versions, CVE flags, SSL certs
         Time: 5 minutes

Step 5 → Review everything collected
         Build a target list: IPs, subdomains, services, versions
         Identify highest-value targets before active recon begins
```

**Total time:** Under 10 minutes. Zero packets sent to target. Complete external attack surface mapped.

---

## Key Takeaways

- **Passive recon is always first** — gather maximum intelligence before sending a single packet. The more you know before active recon, the more targeted and quiet your active scanning can be
- **DNS reveals the full infrastructure** — MX records expose mail servers, TXT records expose cloud services, NS records expose DNS providers. All from one dig command
- **Shodan has already scanned your target** — before you run Nmap, Shodan may already have the port list and banner information. Check it first
- **Subdomains are the real attack surface** — the main domain is often hardened. `dev.`, `staging.`, `admin.`, and `vpn.` subdomains are frequently less protected and directly accessible
- **WHOIS privacy protection is common but not universal** — when registrant email is visible, it is a direct phishing target. When hidden, name servers still reveal the hosting provider

---

## Tools Summary

| Tool | Type | What it gives you | Free? |
|------|------|------------------|-------|
| `whois` | CLI | Domain registration, hosting, name servers | ✅ Yes |
| `nslookup` | CLI | DNS records by type | ✅ Yes |
| `dig` | CLI | DNS records — more detail than nslookup | ✅ Yes |
| DNSDumpster | Web | Subdomain map, visual infrastructure | ✅ Yes |
| Shodan.io | Web | Open ports, versions, CVEs, SSL certs | ✅ Free tier |

---

## Resources

- [DNSDumpster](https://dnsdumpster.com)
- [Shodan](https://shodan.io)
- [MXToolbox — DNS lookup](https://mxtoolbox.com)
- [SecurityTrails — DNS history](https://securitytrails.com)
- [crt.sh — SSL certificate subdomain finder](https://crt.sh)

---

## My Progress

- [x] Passive Reconnaissance ← *this writeup*
- [x] Active Reconnaissance
- [x] Protocols and Servers
- [ ] Nmap Advanced rooms (nmap03, nmap04)
- [ ] Network Services rooms

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
