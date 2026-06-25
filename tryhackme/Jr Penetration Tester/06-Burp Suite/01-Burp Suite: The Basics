# TryHackMe — Burp Suite: The Basics

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Burp%20Suite-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Burp Suite: The Basics |
| Path | Jr Penetration Tester → Burp Suite |
| Tasks | Task 1 — Task 15 (all, 100%) |

---

## Tasks Completed

| Task | Topic | Task | Topic |
|------|-------|------|-------|
| Task 1 | Introduction | Task 9 | Connecting through Proxy (FoxyProxy) |
| Task 2 | What is Burp Suite | Task 10 | Site Map and Issue Definitions |
| Task 3 | Features of Burp Community | Task 11 | The Burp Suite Browser |
| Task 4 | Installation | Task 12 | Scoping and Targeting |
| Task 5 | The Dashboard | Task 13 | Proxying HTTPS |
| Task 6 | Navigation | Task 14 | Example Attack |
| Task 7 | Options | Task 15 | Conclusion |
| Task 8 | Introduction to Burp Proxy | | |

---

## What is Burp Suite?

Burp Suite is the industry-standard web application security testing platform. It sits as a proxy between the browser and the target server, intercepting every HTTP/HTTPS request and response — giving complete visibility and control over all web traffic.

**Two editions:**

| Edition | Cost | Key limitations |
|---------|------|-----------------|
| Community | Free | No active scanner, throttled Intruder |
| Professional | ~$449/year | Full active scanner, unlimited Intruder, Burp Collaborator |

**What Burp Community covers (what I use):** Proxy, Repeater, Intruder (rate-limited), Decoder, Comparer, Sequencer — sufficient for manual testing and the vast majority of real web pentesting work.

---

## Core Burp Proxy Setup

The proxy intercepts all browser traffic — this is the foundation of everything else Burp does.

### Setup workflow

```
1. Open Burp Suite → Proxy tab → ensure listener on 127.0.0.1:8080

2. Configure browser to route traffic through 127.0.0.1:8080
   → Manual: Browser → Settings → Network → Manual Proxy → 127.0.0.1:8080
   → Easier: Install FoxyProxy extension → add Burp profile (127.0.0.1:8080)
      → toggle on/off with one click

3. For HTTPS: download Burp's CA certificate
   → Browse to http://burpsuite while proxy is on
   → Download certificate → install in browser trust store
   → Without this step, HTTPS sites show certificate errors
```

### Intercept mode

```
Intercept ON  → Burp holds every request, waiting for you to forward/drop/modify
Intercept OFF → Traffic flows through Burp transparently (still logged in HTTP history)
```

**Best practice:** Keep intercept OFF by default. Browse normally. Use HTTP History tab to find interesting requests, then send them to Repeater for manual testing. Only turn intercept ON when you want to modify a specific request before it reaches the server.

---

## Key Features Learned

### The Dashboard
- Shows active scans, events, and Burp's advisory output
- Issue definitions tab — a reference library of vulnerability descriptions Burp knows about
- Useful for understanding what each vulnerability type means even without the Pro scanner

### Site Map
- Automatically builds a tree of every URL and endpoint Burp has seen
- Reveals the application's full structure just by browsing around it
- Shows which endpoints returned interesting status codes (403, 500, redirects)

### Scoping and Targeting
```
Target → Scope → Include in scope → add target domain

Why this matters:
- Burp stops logging/processing out-of-scope traffic (third-party CDNs, analytics)
- Prevents accidental testing of systems outside the engagement boundary
- Keeps HTTP History clean and focused on the actual target
```

### Proxying HTTPS
The browser rejects Burp's intercepted HTTPS responses unless its CA certificate is trusted. Installing the cert is a one-time setup step per browser profile — after that, all HTTPS traffic is transparent.

---

## Key Takeaways

- **FoxyProxy makes Burp practical** — toggling manually in browser settings every session would be unworkable; one-click toggle is essential for real workflow
- **Intercept OFF + HTTP History is the correct default workflow** — not having every single request held waiting for approval
- **Scope is not optional** — testing out-of-scope assets (third-party logins, CDNs, analytics) is a legal and ethical boundary issue, not just a noise issue
- **Burp is the foundation** — Repeater, Intruder, and every other tool builds on the proxy. Without understanding the proxy, the other modules don't make sense

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
