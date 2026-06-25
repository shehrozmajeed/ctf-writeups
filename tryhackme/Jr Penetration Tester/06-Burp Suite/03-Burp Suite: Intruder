# TryHackMe — Burp Suite: Intruder

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-Burp%20Suite-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-June%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Burp Suite: Intruder |
| Path | Jr Penetration Tester → Burp Suite |
| Tasks | Task 1 — Task 13 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | What is Intruder |
| Task 3 | Positions |
| Task 4 | Payloads |
| Task 5 | Introduction to Attack Types |
| Task 6 | Sniper |
| Task 7 | Battering Ram |
| Task 8 | Pitchfork |
| Task 9 | Cluster Bomb |
| Task 10 | Practical Example |
| Task 11 | Practical Challenge |
| Task 12 | Extra Mile Challenge |
| Task 13 | Conclusion |

---

## What is Intruder?

Intruder automates customised attacks against web applications — it takes a request, marks specific positions within it as injection points, and iterates through payload lists automatically, sending a modified request for each payload value.

**Community edition limitation:** Intruder is rate-throttled in the free version — requests are sent slowly. For fast attacks, use `ffuf`, `hydra`, or `wfuzz` from the command line instead. Intruder Community is still useful for smaller, targeted attacks where speed is not critical.

---

## Positions — Marking Injection Points

Positions define exactly where in the request payloads will be inserted. Burp marks them with `§` symbols.

```http
POST /login HTTP/1.1
Host: target.thm
Content-Type: application/x-www-form-urlencoded

username=§admin§&password=§password§
```

**The § symbols mark what gets replaced.** Everything outside the markers stays identical across every request. Only the marked values change.

**Setting positions:**
- Auto-assign — Burp guesses injection points (often marks too much)
- Clear all → manually highlight what you want → Add § — more precise
- Always review auto-assigned positions and clear anything irrelevant

---

## Payloads

Payloads are the values Intruder substitutes into the marked positions.

**Payload types:**

| Type | What it does | Best used for |
|------|-------------|---------------|
| **Simple list** | Tries each item in a list one by one | Password wordlists, username lists |
| **Runtime file** | Reads from a file on disk (useful for huge lists) | Large wordlists that would slow Burp to load |
| **Numbers** | Generates sequential or random numbers | ID parameter fuzzing (IDOR testing) |
| **Dates** | Date sequences | Date-based parameter testing |
| **Brute forcer** | Every combination of specified characters up to a length | Short password brute force |
| **Null payloads** | Sends the request with no change | Testing rate limiting (repeat identical request) |

---

## The Four Attack Types

This is the most important concept in the room — each attack type serves a different testing purpose.

### Sniper — one position, one list

```
Position: username=§VALUE§&password=admin

Payload list: [alice, bob, carol, admin]

Requests sent:
  username=alice&password=admin
  username=bob&password=admin
  username=carol&password=admin
  username=admin&password=admin
```

**Best for:** Username enumeration, testing a single parameter with a wordlist, finding injection points.

---

### Battering Ram — multiple positions, same value

```
Positions: username=§VALUE§&password=§VALUE§

Payload list: [admin, root, test]

Requests sent:
  username=admin&password=admin
  username=root&password=root
  username=test&password=test
```

**Best for:** When the same value needs to appear in multiple positions simultaneously — testing if username == password is accepted, or if a value appears in both a parameter and a cookie.

---

### Pitchfork — multiple positions, parallel lists

```
Positions: username=§POS1§&password=§POS2§

List 1 (usernames): [alice, bob, carol]
List 2 (passwords): [Password1!, Summer2024!, Welcome1!]

Requests sent:
  username=alice&password=Password1!     (row 1 of each list)
  username=bob&password=Summer2024!      (row 2 of each list)
  username=carol&password=Welcome1!      (row 3 of each list)
```

**Best for:** Testing known username:password pairs from a credential list (credential stuffing). Lists move in parallel — list 1 row 1 always pairs with list 2 row 1.

---

### Cluster Bomb — multiple positions, every combination

```
Positions: username=§POS1§&password=§POS2§

List 1 (usernames): [alice, bob]
List 2 (passwords): [password1, password2, password3]

Requests sent:
  username=alice&password=password1
  username=alice&password=password2
  username=alice&password=password3
  username=bob&password=password1
  username=bob&password=password2
  username=bob&password=password3
```

Total requests = List1 × List2 = 2 × 3 = 6

**Best for:** Full brute force where you don't know which username/password combination is valid. Tries every possible combination. Generates the most requests — use carefully.

---

## Attack Type Comparison

| Attack | Positions | Lists | Request count | Best use case |
|--------|-----------|-------|---------------|---------------|
| **Sniper** | Any (one at a time) | 1 | Length of list | Single parameter fuzzing |
| **Battering Ram** | Multiple | 1 | Length of list | Same value in multiple positions |
| **Pitchfork** | Multiple | One per position | Length of shortest list | Known username:password pairs |
| **Cluster Bomb** | Multiple | One per position | Product of all list lengths | Full brute force |

---

## Analysing Results

After an attack runs, the results table shows every request and its response. Columns to sort by:

| Column | What to look for |
|--------|-----------------|
| **Status code** | A different status code (200 vs 302 redirect) often signals a successful login |
| **Response length** | A different length response suggests different content — success vs failure message |
| **Response time** | A much slower response can indicate a time-based blind injection |

**Workflow:** Sort by status code first. If all responses are identical, sort by length. Any outlier is worth investigating — right-click → Show response in browser.

---

## Practical Use Cases

**Username enumeration:**
```
Attack: Sniper
Position: username=§VALUE§
Payload: common-usernames.txt
Watch for: Different response length or message ("Invalid password" vs "User not found")
```

**IDOR testing:**
```
Attack: Sniper
Position: id=§VALUE§
Payload: Numbers 1-1000
Watch for: Different status codes or response lengths on IDs that aren't yours
```

**Credential stuffing:**
```
Attack: Pitchfork
Positions: username=§P1§&password=§P2§
List 1: usernames from breach data
List 2: matching passwords from breach data
Watch for: 302 redirect (successful login)
```

---

## Key Takeaways

- **Choose attack type based on the question you're answering** — Sniper for "what values work here?", Pitchfork for "do these pairs work?", Cluster Bomb for "find any working combination"
- **Community throttling makes Intruder slow** — for brute forcing in real engagements or CTFs where speed matters, use `hydra` or `ffuf` instead; use Intruder for smaller, targeted attacks
- **Response length is often more reliable than status code** — many applications return 200 for failed logins with an error message; the success case has a different length
- **Cluster Bomb request count grows exponentially** — two lists of 100 = 10,000 requests. Always calculate before running to avoid flooding a target unintentionally

---

## Burp Suite Series — Complete

| Room | What it taught |
|------|----------------|
| Burp Suite: The Basics | Proxy setup, FoxyProxy, HTTP history, scoping |
| Burp Suite: Repeater | Manual request modification and testing loop |
| **Burp Suite: Intruder** | Automated fuzzing, four attack types, payload management |

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
