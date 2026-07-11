# LLM Security — TryHackMe

![Room Status](https://img.shields.io/badge/Room%20Completed-100%25-brightgreen)
![Path](https://img.shields.io/badge/Path-AI%20Security%20%E2%80%BA%20Secure%20AI%20Systems-blue)
![Time](https://img.shields.io/badge/Duration-60%20min-informational)

## Overview

This room examines **Large Language Models (LLMs) as an attack surface**, moving beyond general AI security concepts into threats specific to how LLMs ingest data, are trained, are deployed as systems, and are interacted with by end users. The threat model is broken down into four layers — **data, model, system, and user** — mirroring how real-world LLM deployments are actually attacked in practice, rather than treating "LLM security" as a single monolithic risk.

| Detail | Value |
|---|---|
| **Room** | LLM Security |
| **Path** | AI Security → Secure AI Systems |
| **Estimated Time** | 60 minutes |
| **Status** | ✅ Completed (100%) |

## Task Breakdown

| Task | Title |
|---|---|
| 1 | Introduction |
| 2 | Data-Based Threats |
| 3 | Model-Based Threats |
| 4 | System-Based Threats |
| 5 | User-Based Threats |
| 6 | Conclusion |

---

## Task 2 — Data-Based Threats

Threats that originate from the **data used to train, fine-tune, or augment** an LLM before it ever reaches deployment.

| Threat | Description |
|---|---|
| **Data Poisoning** | Malicious or manipulated samples inserted into a training/fine-tuning dataset to bias model behavior, plant hidden triggers, or degrade accuracy on specific inputs. |
| **Backdoor Attacks** | A poisoning variant where a specific "trigger" phrase or pattern causes the model to produce attacker-chosen output while behaving normally otherwise — making the compromise hard to detect through standard testing. |
| **Sensitive Data Exposure / Memorization** | LLMs can memorize verbatim snippets of training data (PII, credentials, proprietary text) and later regurgitate them in response to carefully crafted prompts. |
| **Data Provenance & Supply Chain Risk** | Datasets scraped from the open web or sourced from third parties may carry hidden poisoning, licensing issues, or embedded malicious content without the model owner's knowledge. |

**Key mitigations:** dataset vetting and provenance tracking, anomaly detection during data collection, differential privacy during training, and de-duplication/memorization audits before deployment.

## Task 3 — Model-Based Threats

Threats that target the **model itself** — its weights, architecture, and learned behavior.

| Threat | Description |
|---|---|
| **Model Extraction / Theft** | Repeatedly querying a model's API to reconstruct a functionally equivalent copy, stealing intellectual property without accessing the underlying weights directly. |
| **Model Inversion** | Reconstructing sensitive training data (e.g., personal information) by analyzing model outputs or gradients. |
| **Adversarial Examples** | Carefully perturbed inputs designed to cause misclassification or unexpected output, exploiting the model's decision boundaries. |
| **Jailbreaking** | Crafting inputs (role-play framing, encoding tricks, multi-step reasoning chains) that bypass a model's built-in safety alignment to elicit disallowed content. |
| **Supply Chain Risk in Pre-Trained Models** | Using a third-party base model that may contain an embedded backdoor or vulnerability, inherited silently into any downstream fine-tune or deployment. |

**Key mitigations:** rate-limiting and query monitoring to detect extraction attempts, adversarial robustness testing, red-teaming for jailbreak resistance, and verifying the provenance of any pre-trained model before use.

## Task 4 — System-Based Threats

Threats that arise not from the model in isolation, but from **how the LLM is integrated into a larger application or system** — APIs, plugins, retrieval pipelines, and tool access.

| Threat | Description |
|---|---|
| **Prompt Injection (Indirect)** | Malicious instructions embedded in external content (a webpage, document, or email) that the LLM ingests via retrieval or tool use, causing it to execute unintended actions. |
| **Excessive Agency** | Granting an LLM-based agent overly broad permissions (file system access, API calls, financial actions) without sufficient guardrails, turning a successful prompt injection into a high-impact compromise. |
| **Insecure Output Handling** | Passing LLM-generated output directly into a downstream system (e.g., rendering it in a browser, executing it as code, or using it in a SQL query) without sanitization — enabling XSS, RCE, or injection attacks. |
| **Insecure Plugin / Tool Design** | Third-party plugins or tools connected to an LLM that lack proper authentication, input validation, or scoped permissions, expanding the overall attack surface. |
| **Denial of Service** | Resource-exhaustion attacks exploiting expensive inference operations (e.g., extremely long context inputs) to degrade availability or drive up cost. |

**Key mitigations:** treating all LLM output as untrusted before it touches downstream systems, sandboxing tool/plugin execution, enforcing least-privilege on any agentic capabilities, and applying strict input/output validation at every system boundary.

## Task 5 — User-Based Threats

Threats that stem from **how end users (or malicious actors posing as users) interact directly with an LLM**.

| Threat | Description |
|---|---|
| **Direct Prompt Injection** | A user directly instructs the model to ignore its system prompt or safety instructions, attempting to override intended behavior. |
| **Social Engineering via Conversation** | Multi-turn manipulation — building rapport, incremental escalation, or false context — to gradually steer a model toward producing disallowed or sensitive output. |
| **Privacy Leakage Through Interaction** | Users unintentionally (or an attacker deliberately) causing a model to expose information from earlier in a conversation, a connected knowledge base, or another user's session, if session isolation is weak. |
| **Over-Reliance / Misplaced Trust** | Users treating LLM output as authoritative without verification, which attackers can exploit by manipulating a model into confidently producing false or harmful guidance (misinformation, unsafe instructions). |

**Key mitigations:** robust system-prompt enforcement that can't be trivially overridden, conversation-level monitoring for manipulation patterns, strict session/data isolation between users, and clear UX signaling that LLM output should be verified rather than blindly trusted.

## Key Takeaways

- **LLM security isn't one problem — it's four.** Threats need to be reasoned about separately at the data, model, system, and user layers, since each has distinct attack vectors and mitigations.
- **The most severe real-world incidents are usually system-layer failures**, not model-layer ones — an LLM with excessive agency and insecure output handling turns a simple prompt injection into a full compromise.
- **Trust boundaries must be explicit.** Any point where untrusted content (user input, retrieved documents, plugin output) crosses into the model — or where model output crosses back into a downstream system — needs to be treated as a security boundary, not a convenience.
- **Defense-in-depth applies here just as it does in traditional security**: no single control (input filtering, output sanitization, least privilege, monitoring) is sufficient on its own.

---
*Part of my AI Security learning path on TryHackMe. See also: [Securing AI Systems](./01-securing-ai-systems.md), [Prompt Engineering](../01-Ai Fundamentals/03-prompt-engineering.md), [AI/ML Security Threats](../01-Ai Fundamentals/01-ai-ml-security-threats.md)*
