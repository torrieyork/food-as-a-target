# Food as a Target: IoT Security Risk Framework for Cold Chain Infrastructure

A PASTA-based threat modeling framework for multi-stage IoT cold chain infrastructure — addressing protocol-level vulnerabilities, consequence cascades, and decision-making under data uncertainty — with applications in commercial agriculture and military logistics.

---

## About the Author

I came to cybersecurity through agriculture. My background in ag systems gave me a foundation in infrastructure thinking, availability, and the real-world consequences when physical systems fail — not as abstractions, but as outcomes that affect whether people eat. When I pivoted into cybersecurity, cold chain IoT wasn't a niche I stumbled into — it was a deliberate choice to bring domain knowledge to a problem most security professionals haven't lived close to.

This framework was built over several months, validated against practitioner feedback from OTSecCon 2026, and shaped by mentorship from working OT security professionals. My long-term focus is the military cold chain specifically — a domain where food supply security is an operational readiness issue, not just a health and safety compliance requirement. The goal is to be the analyst practitioners and defense organizations call when agricultural and food supply systems are under coordinated attack.

---

## What This Is

A strategic risk framework — not a penetration test. Every threat in this document is tied to a business consequence, a regulatory exposure, or an operational impact. The intended reader is a security executive (CISO or equivalent) making investment decisions about cold chain security, or a defense organization evaluating IoT supply chain risk.

**Two verticals covered:**
- Commercial food logistics (produce, meat, dairy — FDA/USDA/HACCP regulatory environment)
- Military logistics (deployed troop rations, FOB supply, medical/pharmaceutical cold chain — CMMC/NIST 800-171)

**Framework:** PASTA (Process for Attack Simulation and Threat Analysis) — developed by VerSprite. STRIDE used as a categorization tool within Stage 4.

**Risk formula:** Threat × Consequences × Vulnerability = Financial Risk *(McMickle/Lindseth)*

---

## Framework Overview

| PASTA Stage | Section | Status |
|---|---|---|
| Stages 1–3: Objectives, Scope, Decomposition | System Architecture + User Requirements | ✅ |
| Stage 4: Threat Analysis | Threat Actors + Component-by-Component Analysis | ✅ |
| Stage 5: Vulnerability Analysis | Protocol-Level Weaknesses (CVE/CWE) | ✅ |
| Stage 6: Attack Analysis | Attack Trees — 3 Core Scenarios | ✅ |
| Stage 7: Risk & Impact Analysis | Risk Matrix + Control Recommendations | ✅ |
| Synthesis | Consequence Cascade Walkthrough | ✅ |

---

## Table of Contents

1. [System Architecture & User Requirements](docs/01-architecture-and-requirements.md)
2. [Threat Analysis — PASTA Stage 4](docs/02-threat-analysis.md)
3. [Vulnerability Analysis — PASTA Stage 5](docs/03-vulnerability-analysis.md)
4. [Attack Analysis — PASTA Stage 6](docs/04-attack-analysis.md)
5. [Risk & Impact Analysis — PASTA Stage 7](docs/05-risk-and-impact.md)
6. [Consequence Cascade Walkthrough](docs/06-consequence-cascades.md)

---

## Key Principles

**The system must earn operator trust.**
A QA manager who receives too many alerts will ignore all of them. A dashboard that requires too many steps will be abandoned. Security tools that make the operator's job harder get worked around — and a tool that gets ignored is a failed security control.

**Offline-first is a security requirement, not a technical convenience.**
Cellular jamming is a standard adversary tactic in contested environments. A system that degrades when offline is a system an adversary can disable on demand. This is especially critical in military logistics where denied communications environments are the expected condition, not the exception.

**The absence of data is as informative as the presence of bad data.**
Every attack scenario in this framework produces a gap — in log continuity, alert coverage, or chain of custody. A system that treats gaps with the same severity as threshold breaches catches all three attack patterns. A system that only monitors temperature values catches none of them reliably.

**The human detection layer is both the most exploitable vulnerability and the most resilient defense.**
An attacker who manipulates a trusted insider bypasses every technical control. A trained operator who trusts her instincts can catch what the technology misses. The goal of this framework is not to replace human judgment — it is to make human judgment as effective as possible.

---

*For questions, collaboration, or speaking inquiries — connect on LinkedIn.*
