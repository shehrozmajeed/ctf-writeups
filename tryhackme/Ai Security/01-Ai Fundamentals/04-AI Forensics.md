# AI Forensics — TryHackMe

![Room Status](https://img.shields.io/badge/Room%20Completed-100%25-brightgreen)
![Path](https://img.shields.io/badge/Path-AI%20Security-blue)
![Category](https://img.shields.io/badge/Category-DFIR%20%2F%20AI-orange)

## Overview

This room explores how AI/ML techniques are applied in Digital Forensics and Incident Response (DFIR) — from anomaly detection and log classification to the legal and ethical challenges of using AI-driven evidence in court. It closes with a hands-on investigation simulating a real breach, using AI-assisted tooling to trace an attacker's actions from initial access through to source code exfiltration.

| Detail | Value |
|---|---|
| **Room** | AI Forensics |
| **Path** | AI Security |
| **Status** | ✅ Completed (100%) |

## Task 2 — AI Fundamentals for DFIR

| Question | Answer |
|---|---|
| AI ability that helps a DFIR investigator recognize patterns they might not comprehend | **Anomaly Detection** |
| Metric describing the proportion of positively flagged results that were actually correct | **Precision** |
| Term for the AI characteristic where the same input can yield different outputs across runs | **Non-determinism** |

## Task 3 — AI Techniques in Forensic Analysis

| Question | Answer |
|---|---|
| Neural network type commonly used in image/video forensics for learning spatial patterns | **Convolutional Neural Network (CNN)** |
| Analysis performed on chat logs/social media to assess emotional tone | **Sentiment Analysis** |
| Data type AI correlates to automatically reconstruct an incident timeline | **Time-Sequenced Data** |
| Analysis that observes program behavior (e.g., API call sequences) to determine maliciousness | **Dynamic Analysis** |

## Task 4 — Legal, Ethical & Privacy Considerations

| Question | Answer |
|---|---|
| U.S. legal test for admissibility of expert/scientific testimony | **Daubert Standard** |
| Term for AI models whose decision-making is difficult to interpret | **Black Box** |
| Real-world tech shown to produce racially biased identification results | **Facial Recognition** |
| Technique enabling ML training without centralizing sensitive data | **Federated Learning** |

## Task 5 — Practical: AI-Assisted DFIR Investigation

A simulated breach investigation using AI-assisted tooling to classify logs and flag anomalous files, then manually validating each finding.

### Phase I: Initial Access

Activated the DFIR environment and ran an AI-based log classifier against the authentication log to surface suspicious entries:

```bash
source /opt/dfir-env/bin/activate
python3 /opt/dfir-lab/classify_logs.py /var/log/auth.log
```

The classifier flagged anomalous authentication activity, indicating the attacker had escalated privileges.

Next, an anomaly-detection script trained to identify malicious files was run across the filesystem:

```bash
python3 /opt/dfir-lab/file_anomalies.py
```

Since AI-flagged results aren't automatically ground truth, each flagged file was manually reviewed, starting with the most suspicious.

**`invoice_dump.txt`** — Reviewing `bash_history` revealed an invoice file had been accessed. The associated user could be identified either via `find` or by cross-referencing the AI-flagged suspicious logs (`j.morgan`, `r.house`):

```bash
find . -name "invoice_Q1_2075.ods"
cat ./j.morgan/Documents/Invoices/invoice_Q1_2075.ods
```

This confirmed the invoice was used for data harvesting, explaining how the attacker initially compromised `j.morgan`'s account (phishing).

### Phase II: Tooling and Infrastructure

Reviewed additional flagged files in `/tmp`:

```bash
cat /tmp/.syncd
cat /tmp/.x
```

These files showed the attacker downloading a reverse shell payload and saving it locally as `.x`. Using `curl` to pull an already-weaponized shell (rather than crafting one in place) is a common evasion technique — it reduces the on-disk footprint of suspicious activity during delivery.

### Phase III: Privilege Escalation

Reviewing `j.morgan`'s `bash_history` revealed the attacker modified `r.house`'s `authorized_keys` file to add their own SSH key, escalating access from `j.morgan` to `r.house`:

```bash
sudo nano /home/r.house/.ssh/authorized_keys
```

### Phase IV: Disguise and Persistence

A second reverse shell was found bound to a different port, alongside a fabricated log designed to impersonate Sysmon — an attempt to blend malicious activity into legitimate monitoring output and evade analyst detection.

### Phase V: Source Code Theft

Investigated two source files flagged by the AI tooling:

- `/opt/robbco/engineering/MFBootAgent/mfboot_main.c`
- `/opt/robbco/firmware/RETROS_BIOS/core.asm`

Both turned out to be benign — a useful reminder that **AI-based detection is not 100% accurate** and requires human validation.

However, `/dev/shm/.core_dump_2025.tgz.enc` was found to be a Base64-encoded archive. Decoding it revealed a compressed archive containing stolen source code, staged for exfiltration in a location (`/dev/shm`, tmpfs) designed to avoid leaving persistent disk artifacts.

### Investigation Findings

| Question | Answer |
|---|---|
| Time the attacker successfully logged in as `j.morgan` | **03:01:02** |
| Initial access attack method | **Phishing** |
| Attacker's email address | **akeane@poseidonenergy.net** |
| Command used to gain access to `r.house`'s account | `sudo nano /home/r.house/.ssh/authorized_keys` |
| Full path of the archive used to steal RobbCo's source code | `/dev/shm/.core_dump_2025.tgz.enc` |

## Key Takeaways

- **AI accelerates triage** — anomaly detection and log classification surfaced suspicious activity far faster than manual review alone, but every AI-flagged result still required human validation.
- **False positives happen** — two flagged source files turned out to be benign, reinforcing that AI output is a lead, not a verdict.
- **Attacker tradecraft mirrors traditional DFIR patterns** — phishing for initial access, SSH key manipulation for privilege escalation, log spoofing for persistence, and encoded archives in volatile storage (`/dev/shm`) for exfiltration.
- **Legal and ethical context matters** — understanding standards like *Daubert* and the limitations of black-box models is essential when AI-derived evidence may factor into legal proceedings.
- **Privacy-preserving techniques** like Federated Learning show how AI/ML can be applied to sensitive forensic data without centralizing it.

---
*Part of my AI Security learning path on TryHackMe.*
