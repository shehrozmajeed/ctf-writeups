# Securing AI Systems — TryHackMe

![Room Status](https://img.shields.io/badge/Room%20Completed-100%25-brightgreen)
![Path](https://img.shields.io/badge/Path-AI%20Security%20%E2%80%BA%20Secure%20AI%20Systems-blue)
![Time](https://img.shields.io/badge/Duration-60%20min-informational)

## Overview

This room focuses on mapping AI system architecture, identifying attack surfaces through the lens of OWASP and MITRE ATLAS frameworks, and applying secure design principles across AI trust boundaries. It shifts from AI fundamentals into practical secure-design thinking for real-world AI deployments.

| Detail | Value |
|---|---|
| **Room** | Securing AI Systems |
| **Path** | AI Security → Secure AI Systems |
| **Estimated Time** | 60 minutes |
| **Status** | ✅ Completed (100%) |

## Task Breakdown

| Task | Title | Focus |
|---|---|---|
| 1 | Introduction | Framing the goals of secure AI system design |
| 2 | Anatomy of an AI System | Components of an AI system (data pipeline, model, inference layer, APIs, integrations) and their trust boundaries |
| 3 | The AI Attack Surface | Mapping attack surfaces using **OWASP** (e.g., OWASP Top 10 for LLMs) and **MITRE ATLAS** |
| 4 | System-Level Threats | Threats that emerge from how AI components interact within a broader system, not just the model in isolation |
| 5 | Secure Design Patterns | Architectural and design patterns for reducing AI-specific risk (input validation, output filtering, least privilege, sandboxing) |
| 6 | Auditing TryAssist: A Conversation with the System | Applied exercise — auditing a live AI assistant ("TryAssist") for security weaknesses via direct interaction |
| 7 | Conclusion | Recap and key takeaways |

## Key Concepts Learned

- **Anatomy of an AI System** — breaking a system down into its core components (data ingestion, training pipeline, model, inference/serving layer, application logic, and external integrations) to identify where trust boundaries exist and where they can be crossed.
- **Attack Surface Mapping** — using established frameworks like the **OWASP Top 10 for LLM Applications** and **MITRE ATLAS** to systematically catalog AI-specific threats (e.g., prompt injection, training data poisoning, model theft, supply-chain risk) rather than relying on ad-hoc threat identification.
- **System-Level vs. Model-Level Threats** — recognizing that many real-world AI risks arise not from the model itself but from how it's wired into surrounding systems — plugins, tool access, over-permissioned APIs, and unchecked data flows.
- **Secure Design Patterns** — practical mitigations such as strict input/output validation, sandboxing model actions, enforcing least privilege on any tools or APIs a model can access, and maintaining human-in-the-loop checkpoints for high-impact actions.
- **Practical Auditing** — applying the above concepts hands-on by probing "TryAssist," a simulated AI assistant, to identify security gaps through direct conversation and interaction rather than static analysis alone.

## Takeaway

This room tied together architectural thinking with structured threat frameworks (OWASP, MITRE ATLAS), reinforcing that securing AI systems requires the same rigor as securing traditional software — mapping trust boundaries, cataloging threats systematically, and applying defense-in-depth design patterns — while also accounting for AI-specific risks like prompt injection and model-level exploitation.

---
*Part of my AI Security learning path on TryHackMe. See also: [AI/ML Security Threats](./01-ai-ml-security-threats.md), [AI Models & Data](./02-ai-models-and-data.md), [Prompt Engineering](./03-prompt-engineering.md), [AI Forensics](./04-ai-forensics.md)*
