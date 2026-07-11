# Room 03 — Humans as Attack Vectors

**Path:** SOC Level 1 → Blue Team Introduction
**Platform:** TryHackMe

> Personal study notes and reflection. Not a reproduction of the room's proprietary content — flags and identifiers are masked/generalized.

---

## Overview

Why people — not just systems — are the primary target for attackers, and how SOC analysts factor into defending the "human layer."

## Why Humans Get Targeted

People are gatekeepers to valuable access — email, corporate systems, financial platforms, sensitive data. Compromising a person is often easier than compromising a system directly. Real-world goals include things like using a compromised HR account to steal employee data, or a compromised admin credential to gain VPN/network access.

## Social Engineering

A manipulation tactic built on psychology, not technical exploits. Effective because it's:

- **Trustworthy-looking** — appears legitimate
- **Emotionally charged** — urgency, fear, curiosity

## Common Attack Types Covered

- **Phishing** — the most common vector by volume; billions of phishing emails sent daily
- **Malware downloads** — fake software, malicious sites, fake CAPTCHAs, often paired with SEO poisoning
- **Deepfakes** — AI-generated audio/video impersonation used in real fraud cases (including multi-million dollar incidents)
- **Impersonation** — fake IT staff, executives, or partners; many ransomware attacks start this way
- **Other vectors** — USB drops, insider threats, fake job offers, physical access attacks

## Defense Model: Mitigation vs Detection

- **Mitigation** — prevent or reduce the likelihood of an attack succeeding
- **Detection** — catch what slips past mitigation

Key mitigation measures: anti-phishing solutions, antivirus/EDR tools, a "trust but verify" culture, and security awareness training — arguably the strongest long-term defense of the group.

## SOC Analyst's Role

Monitor phishing/malware alerts, investigate suspicious user activity, partner with IT/HR, help shape awareness programs. In some orgs, SOC analysts are also the point of contact when employees report something suspicious.

## Lab Reflection

Worked through identifying at-risk employees and shaping an appropriate security policy response.

**Flags:** `THM{**********}` and `THM{**********}` *(masked)*

## Takeaways

- Humans are the most common entry point, not just "the weakest link"
- Attackers want access — social engineering is just the means
- Security awareness training is one of the strongest long-term defenses
- AI-driven threats (deepfakes) are raising the stakes

---

*Part of the [SOC Level 1](../../) learning series.*
