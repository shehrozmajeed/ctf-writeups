# AI Models & Data — TryHackMe

![Room Status](https://img.shields.io/badge/Room%20Completed-100%25-brightgreen)
![Path](https://img.shields.io/badge/Path-AI%20Security%20%E2%80%BA%20AI%20Fundamentals-blue)
![Time](https://img.shields.io/badge/Duration-60%20min-informational)

## Overview

This room explores how data underpins AI security and takes a closer look at the models that power modern AI systems. It builds on foundational AI concepts by examining where models come from, how training data shapes their behavior, and why the internal workings of many models remain difficult to interpret — a challenge with direct security implications.

| Detail | Value |
|---|---|
| **Room** | AI Models & Data |
| **Path** | AI Security → AI Fundamentals |
| **Difficulty** | Easy (Introductory) |
| **Estimated Time** | 60 minutes |
| **Status** | ✅ Completed (100%) |

## Task Breakdown

| Task | Title | Focus |
|---|---|---|
| 1 | Introduction | Why data and model provenance matter for AI security |
| 2 | Training Data | The role of data quality, bias, and integrity in model behavior |
| 3 | Building the Model | How raw data becomes a functioning AI model |
| 4 | The Inheritance Problem | Risks inherited from pre-trained/base models and third-party datasets |
| 5 | The Black Box Problem | Interpretability challenges and why opaque models are a security concern |
| 6 | Practical | Hands-on exercise applying the concepts |
| 7 | Conclusion | Recap and key takeaways |

## Key Concepts Learned

- **Training Data as an Attack Surface** — poor data provenance, bias, or intentional poisoning can compromise a model before it's ever deployed.
- **Model Lifecycle** — the journey from raw data → preprocessing → training → a deployable model, and where security controls should be applied along the way.
- **The Inheritance Problem** — using pre-trained models or third-party datasets means inheriting their vulnerabilities, biases, and any hidden backdoors, making supply-chain-style thinking essential for AI systems.
- **The Black Box Problem** — many AI models (especially deep learning models) are difficult to interpret, which complicates auditing, incident response, and trust verification.
- **Data Integrity & Provenance** — the importance of verifying dataset sources and monitoring for drift or tampering over a model's lifecycle.

## Takeaway

This room highlighted that AI security starts long before deployment — with the data and model choices made upstream. Understanding inherited risk and interpretability gaps is essential context for assessing the trustworthiness of any AI system.

---
*Part of my AI Security learning path on TryHackMe. See also: [AI/ML Security Threats](./01-ai-ml-security-threats.md), [Prompt Engineering](./03-prompt-engineering.md)*
