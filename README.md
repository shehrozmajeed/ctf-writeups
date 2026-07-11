# 🏴 CTF Writeups

Walkthroughs of machines and challenges from TryHackMe and Hack The Box, documented to reinforce methodology and demonstrate hands-on penetration testing skills.

## About

Each writeup follows a structured approach — reconnaissance, enumeration, exploitation, and privilege escalation — and is written independently, in my own words, after completing each machine.

## Machines Completed

### TryHackMe

| Machine | Difficulty | Category |
|---|---|---|
| Pickle Rick | Easy | Web / Linux |
| Vulnversity | Easy | Web / Privilege Escalation |
| Basic Pentesting | Easy | Enumeration / Credential Cracking |
| Game Zone | Medium | SQL Injection / SSH |
| Skynet | Medium | SMB / Remote File Inclusion |

### Hack The Box

*Writeups in progress — will be added as machines are completed.*

## Writeup Format

Each machine writeup includes:

- **Target Overview** — machine name and difficulty
- **Reconnaissance** — Nmap scan output and initial findings
- **Enumeration** — services explored and reasoning
- **Exploitation** — vulnerability identified, tool used, and payload
- **Privilege Escalation** — misconfiguration found and method used
- **Key Takeaway** — one lesson learned from the machine

## Tools Used

| Category | Tools |
|---|---|
| Scanning & Enumeration | Nmap, Gobuster |
| Exploitation | Burp Suite, Metasploit, Netcat |
| Credential Attacks | Hydra, John the Ripper |
| Privilege Escalation | LinPEAS |
| Traffic Analysis | Wireshark, tcpdump |

## Connect

- **LinkedIn:** [linkedin.com/in/shehroz-majeed-a46a012b8](https://www.linkedin.com/in/shehroz-majeed-a46a012b8/)
- **TryHackMe:** [tryhackme.com/p/shehrozmajeed](https://tryhackme.com/p/shehrozmajeed)

---

## 🌐 Documentation Website (MkDocs)

This repository is powered by **MkDocs Material** to automatically compile these Markdown notes into a fully responsive, professional documentation website hosted on GitHub Pages.

### Features
- 🚀 **Automatic Navigation**: No need to manually add files in `mkdocs.yml`; directories are mapped automatically.
- 🌓 **Dark/Light Mode**: User preference matching with a manually toggleable high-contrast cybersecurity theme.
- 🔍 **Instant Search**: Find commands, notes, and topics across all paths instantly.
- 📋 **Copy to Clipboard**: Quick copy buttons on all code snippets.
- 📅 **Last Updated Timestamps**: Uses Git history to show when each note page was last modified.

### Local Setup & Testing

To run and preview the documentation website locally, follow these steps:

1. **Create and activate a virtual environment:**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

2. **Install the dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Recreate build symlinks (if on Linux/macOS):**
   ```bash
   mkdir -p docs
   ln -sf ../tryhackme docs/tryhackme
   ln -sf ../index.md docs/index.md
   ln -sf ../assets docs/assets
   ```

4. **Start the local development server:**
   ```bash
   mkdocs serve
   ```
   Open your browser and navigate to `http://127.0.0.1:8000/`.

### Automatic Deployment

Whenever you push changes to the `main` or `master` branch:
1. **GitHub Actions** (`.github/workflows/deploy.yml`) is triggered.
2. It fetches your full git history, installs dependencies, compiles the documentation, and deploys it directly to your **GitHub Pages** branch (`gh-pages`).
3. Your site will automatically go live at `https://shehrozmajeed.github.io/ctf-writeups/`.

> [!NOTE]
> Ensure that GitHub Pages is enabled in your repository settings (**Settings** -> **Pages** -> **Build and deployment** -> **Source: Deploy from a branch** -> Select **`gh-pages`** and `/ (root)`).
