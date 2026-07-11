# Room 01 — Junior Security Analyst Intro

**Path:** SOC Level 1 → Blue Team Introduction
**Platform:** TryHackMe

> Personal study notes and reflection. Not a reproduction of the room's proprietary content — flags and identifiers are masked/generalized.

---

## Overview

An introduction to the Junior Security Analyst (SOC L1) role — what the job actually looks like day to day inside a 24/7 Security Operations Center.

## The Threat Landscape

The starting point for the role: organizations face a constant stream of threats — DDoS attacks, nation-state campaigns against critical infrastructure, supply chain compromises, SaaS data breaches. Every organization is a target; the SOC L1 analyst's job is making sure their org isn't the next headline.

## Responsibilities Covered

- Monitoring security alerts
- Investigating suspicious activity
- Escalating incidents to senior team members
- Collaborating across teams
- Continuously learning new threats and defenses

## SOC Team Structure (as modeled in the room)

| Role | Responsibility |
|------|-----------------|
| Senior Analyst | Handles complex investigations, supports L1 analysts |
| SOC Engineer | Maintains and configures security tools & alerting |
| SOC Manager | Oversees SOC operations, reports to leadership |
| Incident Responder | Steps in for major, escalated incidents |

## Lab Reflection

Triaged a simulated SOC alert, identified a malicious external address (`target_ip`), and worked through the correct escalation path before taking firewall action.

**Flag:** `THM{**********}` *(masked)*

## Takeaways

- How L1 analysts triage and prioritize alerts
- Why escalation procedures exist and matter
- What a real incident response workflow looks like
- Security is a team effort, not a solo one

---

*Part of the [SOC Level 1](../../) learning series.*
