# TryHackMe — Burp Suite: Repeater

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Burp%20Suite-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Burp Suite: Repeater |
| Path | Jr Penetration Tester → Burp Suite |
| Tasks | Task 1 — Task 9 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | What is Repeater? |
| Task 3 | Basic Usage |
| Task 4 | Message Analysis Toolbar |
| Task 5 | Inspector |
| Task 6 | Practical Example |
| Task 7 | Challenge |
| Task 8 | Extra-mile Challenge |
| Task 9 | Conclusion |

---

## What is Repeater?

Repeater is the most-used Burp module for manual web application testing. It lets you capture a single HTTP request, modify any part of it, resend it, and immediately inspect the response — repeating this cycle as many times as needed.

**Why Repeater is essential:** Browsers make requests automatically and don't let you easily modify individual headers, parameters, or body fields before sending. Repeater gives complete, granular control over every byte of a request.

---

## Core Workflow

```
1. Find an interesting request in Proxy → HTTP History
2. Right-click → Send to Repeater (or Ctrl+R)
3. Repeater tab opens with the full request
4. Modify the request — change a parameter, add a header, alter the body
5. Click Send
6. Response appears on the right panel immediately
7. Repeat — adjust, send, observe, adjust again
```

**The test-modify-observe loop is the core of manual web testing.** Every vulnerability I've found in PortSwigger labs and TryHackMe rooms follows this same pattern in Repeater.

---

## The Interface

```
Left panel:  The HTTP request (fully editable)
Right panel: The server's response
Bottom bar:  Response analysis tabs
```

### Message Analysis Toolbar (response view tabs)

| Tab | What it shows |
|-----|---------------|
| **Pretty** | Formatted, readable version of the response |
| **Raw** | Exact bytes returned by the server |
| **Hex** | Hexadecimal view — useful for binary responses |
| **Render** | Renders the HTML response as a web page |

**Render tab tip:** Use this to visually confirm that a reflected XSS payload or injected content actually appears in the page as rendered HTML — not just as raw text in the source.

### Inspector Panel

Inspector parses the current request/response into structured sections, making specific elements easier to find and edit without manually scanning the raw text:

- **Request Attributes** — method, URL, HTTP version
- **Request Headers** — each header as an editable row
- **Request Body** — decoded body parameters
- **Response Headers** — server headers broken out clearly
- **Cookies** — all cookies shown separately

**When to use Inspector vs raw editing:** Inspector is faster for targeting a specific header or cookie. Raw editing is better when you want to modify the structure itself (add/remove lines, change request method).

---

## Practical Testing Examples

### Testing SQLi in Repeater

```http
GET /product?id=1 HTTP/1.1
Host: target.thm
```

Modified to:
```http
GET /product?id=1' HTTP/1.1
Host: target.thm
```

Observe response — database error? Different content? Then iterate:
```
id=1 OR 1=1--
id=1 UNION SELECT null,null--
id=1 UNION SELECT username,password FROM users--
```

### Testing access control (IDOR)

```http
GET /api/user?id=3 HTTP/1.1
```

Change to `id=1`, `id=2`, `id=4` — does the server return other users' data?

### Testing authentication headers

```http
Cookie: session=eyJ1c2VyIjoidXNlciJ9
```

Decode the cookie (Base64), modify `"user":"user"` → `"user":"admin"`, re-encode, resend — does the server accept it?

---

## Key Takeaways

- **Repeater is where real manual testing happens** — automated scanners find common patterns; Repeater is how you test the application's specific logic
- **The render tab confirms visual impact** — always render injected content to confirm it actually appears as expected in the page, not just in the raw source
- **Inspector saves time on repetitive edits** — when testing many parameter values, Inspector's row-by-row view is faster than hunting through raw text
- **Every PortSwigger lab I've completed uses this exact workflow** — intercept → send to Repeater → modify → observe → iterate. Knowing Repeater fluently is knowing manual web testing fluently

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
