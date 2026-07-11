# Room 04 — Systems as Attack Vectors

**Path:** SOC Level 1 → Blue Team Introduction
**Platform:** TryHackMe

> Personal study notes and reflection. Not a reproduction of the room's proprietary content — flags and identifiers are masked/generalized.

---

## Overview

The counterpart to the human-focused room — how attackers get in even when people do everything right, by targeting weak or unpatched systems instead. Even a well-trained employee can't compensate for a weak lock on the gate.

## What Counts as a "System"

Any platform where data is stored or processed: physical servers, personal laptops, cloud platforms (e.g. Microsoft 365), web applications. Impact scales with what the system controls:

- Compromised personal laptop → minor data theft
- Compromised admin machine → full network access
- Compromised mail server → thousands of accounts exposed
- Compromised industrial system → entire operations disrupted

## How Systems Get Attacked

**Human-led entry points** — even system attacks often start with human mistakes: weak/reused passwords, pirated or malicious software, unknown USB devices. Roughly 81% of breaches involve compromised passwords.

**Vulnerabilities** — flaws in software that attackers can exploit. Thousands are discovered yearly and tracked via **CVE (Common Vulnerabilities and Exposures)** IDs. **Zero-days** are the worst case: found and exploited by attackers before defenders even know they exist.

**Supply chain attacks** — compromising trusted software or libraries to hit many organizations at once through a single point of entry (e.g. SolarWinds-style incidents). Dangerous because of blast radius, not sophistication.

**Misconfigurations** — human errors in setup rather than software flaws: weak default passwords, exposed databases, open access to critical systems. One of the most common real-world attack vectors.

## Remediation Model

| Problem | Fix |
|---------|-----|
| Vulnerability | **Patch** (update the software; in the meantime, restrict access or apply vendor mitigations) |
| Misconfiguration | **Reconfigure** (fix the setup — a patch won't help) |

## Proactive Defense Strategies

- **Penetration testing** — an authorized, simulated attack to find weaknesses
- **Vulnerability scanning** — automated detection of flaws
- **Configuration audits** — checking systems follow best practices

## System Protection Measures

Patch management, IT security training (to reduce configuration mistakes), network access restrictions, antivirus/EDR solutions.

## SOC Analyst's Role

Monitor system-level alerts, detect exploitation attempts, investigate suspicious activity, recommend remediation, work with IT — bridging detection and response.

## Lab Reflection

Analyzed a set of vulnerable systems and matched each one to the correct remediation strategy — patch vs. reconfigure.

**Flags:** `THM{**********}` and `THM{**********}` *(masked)*

## Takeaways

- Not every attack needs a human mistake — weak systems are enough on their own
- Vulnerabilities and misconfigurations need different fixes, and mixing them up wastes response time
- Supply chain attacks are dangerous because of blast radius, not sophistication
- Strong defense = humans **and** systems, not either/or

---

*Part of the [SOC Level 1](../../) learning series.*
