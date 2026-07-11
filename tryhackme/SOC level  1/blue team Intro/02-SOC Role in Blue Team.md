# Room 02 — SOC Role in Blue Team

**Path:** SOC Level 1 → Blue Team Introduction
**Platform:** TryHackMe

> Personal study notes and reflection. Not a reproduction of the room's proprietary content — flags and identifiers are masked/generalized.

---

## Overview

Zooms out from the SOC itself to see how it fits into a company's wider security structure — leadership, team divisions, and career paths.

## Security Priorities Vary by Domain

Different organizations weight security differently: law firms prioritize confidentiality, factories prioritize availability, hospitals prioritize safety and integrity. Context shapes what "secure" means.

## Security Leadership Hierarchy

- **CEO** — business strategy
- **CISO** — owns cybersecurity strategy org-wide
- **Security teams** — execute day-to-day operations

## Security Departments

| Team | Focus |
|------|-------|
| 🔴 Red Team | Offensive security, simulated attacks |
| 🟢 GRC Team | Governance, risk, regulatory compliance |
| 🔵 Blue Team | Defensive security — monitor, detect, respond |

## SOC Roles

L1 Analysts (triage) → L2 Analysts (deeper investigation) → SOC Engineers (SIEM/EDR, detection rules) → SOC Manager (oversight)

## CIRT — Cyber Incident Response Team

The escalation point for major incidents — forensics, malware analysis, threat intel, high-pressure response. Effectively cybersecurity's "firefighting unit."

## Specialized Blue Team Roles

As organizations grow, they add specialized roles that often evolve out of SOC experience: Digital Forensics Analyst, Threat Intelligence Analyst, AppSec Engineer, DevSecOps Engineer, AI Security Researcher.

## Internal SOC vs MSSP

| Feature | Internal SOC | MSSP |
|---------|--------------|------|
| Environment | Single organization | Multiple clients |
| Workload | Moderate | High-pressure |
| Tooling | Deep, limited | Broad, varied |
| Learning curve | Slower | Faster exposure |

## Career Path Mapped

`SOC L1 → SOC L2 → Specialized Roles (CIRT / Threat Intel / Engineering) → Leadership (Manager → CISO)`

## Lab Reflection

Final challenge involved acting as CISO and assigning the right teams to handle multiple simultaneous incidents.

**Flag:** `THM{**********}` *(masked)*

## Takeaways

- Blue Team = defensive security; SOC = first line of defense
- CIRT handles the critical, high-severity stuff
- CISO is the top security decision-maker
- The natural next step after L1 is L2 — not straight to specialization

---

*Part of the [SOC Level 1](../../) learning series.*
