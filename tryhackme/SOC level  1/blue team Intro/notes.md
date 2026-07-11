# Blue Team Introduction — Notes

**Path:** SOC Level 1 → Blue Team Introduction
**Purpose:** Quick-reference talking points for technical interviews covering SOC/Blue Team fundamentals.

---

## 1. What is Blue Team?

The defensive side of cybersecurity — responsible for monitoring, detecting, and responding to threats to protect an organization's systems, networks, and data.

- Centers on **prevention + detection + response**
- Defends against real attacks, as opposed to Red Team's simulated ones

## 2. What is a SOC?

A Security Operations Center (SOC) is a centralized team that monitors security alerts 24/7 and responds to potential threats.

- Built around **SIEM and EDR** tooling
- Works alerts as tickets through a defined workflow
- The organization's first line of defense

## 3. SOC Roles

| Role | Function |
|------|----------|
| L1 Analyst | Alert monitoring & triage |
| L2 Analyst | Deeper investigation |
| SOC Engineer | Tool configuration & detection rules |
| SOC Manager | Team oversight & reporting |

## 4. What Does a SOC Analyst Do Day to Day?

- Monitor alerts
- Investigate suspicious activity
- Escalate incidents
- Document findings
- Collaborate with IT/security teams

> "We follow a triage → investigation → escalation workflow."

## 5. What is CIRT?

The Cyber Incident Response Team handles major or critical incidents that exceed the SOC's scope — forensics, malware analysis, and containment.

## 6. Red Team vs. Blue Team

- **Red Team** — attacks; simulates real-world adversaries
- **Blue Team** — defends; protects systems and data
- **Purple Team** — the two working together

## 7. Why Are Humans a Common Target?

People can be manipulated through social engineering, which is often an easier path in than breaking a hardened system directly. Examples: phishing, impersonation, malware downloads.

## 8. What is Social Engineering?

A technique where attackers manipulate human psychology — rather than exploiting a technical flaw — to gain access or information.

Key levers: **trust, urgency, fear.**

## 9. Common Human-Targeted Attacks

- Phishing
- Malicious links/downloads
- Deepfakes
- Impersonation

*Interview tip: name the category, then give one concrete example.*

## 10. What Do Attackers Actually Want?

Access. Once an attacker has access, they can steal data, deploy ransomware, or move laterally through the network.

## 11. Can an Attack Happen Without a User Involved?

Yes — attackers can exploit system vulnerabilities or misconfigurations directly, with no user interaction required.

## 12. Vulnerability vs. Misconfiguration

The single most important distinction in this module:

| Type | Definition | Fix |
|------|------------|-----|
| Vulnerability | A flaw in the software itself | **Patch** |
| Misconfiguration | A human error in setup | **Reconfigure** |

## 13. What is a CVE?

Common Vulnerabilities and Exposures — a unique identifier assigned to a publicly known security vulnerability, used to track and manage it across the industry.

## 14. What is a Supply Chain Attack?

An attack where malicious code enters through trusted software, updates, or third-party libraries — meaning a single compromised vendor can affect every downstream user.

## 15. Mitigation vs. Detection

- **Mitigation** — reduces the chance an attack succeeds
- **Detection** — identifies an attack that's already happening

> "Even strong mitigation isn't complete on its own — detection matters because some attacks always get through."

## 16. Baseline Security Controls Worth Mentioning

- Patch management
- Antivirus / EDR
- Network access restrictions
- Security awareness training

## 17. Internal SOC vs. MSSP

- **Internal SOC** — one organization, deeper context over time
- **MSSP** — multiple clients, faster-paced, broader exposure

> "MSSP work builds breadth faster; an internal SOC builds depth in one environment."

---

## Fallback Answer

If a question runs long or you get stuck mid-answer, land on this:

> "As a SOC analyst, my role is to monitor alerts, investigate suspicious activity, and escalate incidents when necessary to protect the organization."

---

*Part of the [SOC Level 1](../../) learning series.*
