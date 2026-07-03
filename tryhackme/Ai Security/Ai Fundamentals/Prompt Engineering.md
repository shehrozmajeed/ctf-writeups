# Prompt Engineering — TryHackMe

![Room Status](https://img.shields.io/badge/Room%20Completed-100%25-brightgreen)
![Path](https://img.shields.io/badge/Path-AI%20Security%20%E2%80%BA%20AI%20Fundamentals-blue)
![Time](https://img.shields.io/badge/Duration-60%20min-informational)

## Overview

This room covers how Large Language Models (LLMs) process text and how to craft effective prompts — both for legitimate use and for security/adversarial testing purposes. It bridges general prompt engineering skills with the security mindset needed to probe LLM behavior.

| Detail | Value |
|---|---|
| **Room** | Prompt Engineering |
| **Path** | AI Security → AI Fundamentals |
| **Difficulty** | Easy (Introductory) |
| **Estimated Time** | 60 minutes |
| **Status** | ✅ Completed (100%) |

## Task Breakdown

| Task | Title | Focus |
|---|---|---|
| 1 | Introduction | Why prompt engineering matters for both usability and security |
| 2 | LLM Fundamentals | How LLMs tokenize, process, and generate text |
| 3 | The Anatomy of a Prompt | Structure and components of an effective prompt |
| 4 | System vs User Prompts | The distinction between system-level instructions and user input, and why it matters for security |
| 5 | Advanced Prompting Techniques | Techniques such as few-shot prompting, chain-of-thought, and role framing |
| 6 | Challenge | Applied, hands-on prompt engineering challenge |
| 7 | Conclusion | Recap and key takeaways |

## Key Concepts Learned

- **LLM Text Processing** — how tokenization and context windows shape the way models interpret input, which underlies many prompt-based attack techniques.
- **Prompt Anatomy** — the building blocks of a well-formed prompt (context, instructions, constraints, examples) and how each part influences model output.
- **System vs. User Prompt Boundaries** — understanding the (often fragile) separation between system instructions and user-supplied input is foundational to reasoning about prompt injection risks.
- **Advanced Prompting Techniques** — structured approaches (few-shot examples, chain-of-thought reasoning, explicit role assignment) that improve output quality and are also relevant when crafting adversarial test cases.
- **Security Relevance** — how the same prompting skills used to get better outputs from an LLM can be repurposed to test for prompt injection, jailbreaking, and other LLM-specific weaknesses in a controlled, authorized setting.

## Takeaway

This room tied together LLM fundamentals with practical prompt-crafting skills, framing prompt engineering not just as a productivity tool but as a core technique for adversarial testing of AI systems.

---
*Part of my AI Security learning path on TryHackMe. See also: [AI/ML Security Threats](./01-ai-ml-security-threats.md), [AI Models & Data](./02-ai-models-and-data.md)*
