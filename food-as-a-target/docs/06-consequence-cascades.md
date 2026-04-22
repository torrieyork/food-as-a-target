# Consequence Cascade Walkthrough
### Food as a Target: IoT Security Risk Framework for Cold Chain Infrastructure
[← Back to Main README](../README.md)

---

The consequence cascade is the core analytical contribution of this framework. It demonstrates that a small, targeted technical failure — one that may never trigger an alert — can produce outcomes that are economic, public health, legal, and in the military context, national security events.

Each cascade is mapped to the PASTA stage that identified the relevant element, showing how the framework builds from objectives through to real-world impact.

> *A small technical failure → operational issues → physical/economic harm → societal impact → systemic instability*

---

## Cascade 1: Data Integrity Attack

**Trigger:** An organized crime actor or nation-state targets a high-value cold chain shipment and manipulates sensor data to make a compromised shipment appear acceptable.

| Stage | Event | PASTA Reference |
|---|---|---|
| 1. Technical exploitation | Attacker injects false temperature readings via Modbus signal injection or BLE spoofing (BLESA). No authentication required — the protocol allows any device on the network to write sensor values. | Stage 5: Modbus CWE-306, BLE CVE-2020-9770 |
| 2. Data corruption | Gateway records attacker-controlled values throughout transit. Logs appear complete, continuous, and within safe temperature range. No anomalies, no gaps, no alerts. | Stage 6: Scenario 1 — data looks clean and complete |
| 3. Cloud sync | At receiving, gateway syncs stored data to cloud. QA dashboard shows green status. MQTT channel may itself be unencrypted, allowing a second opportunity to intercept or alter data. | Stage 5: MQTT CWE-306; Stage 7: R2 |
| 4. Human decision point | QA manager reviews logs within her 45–60 minute window. Data is clean, no threshold breaches, no gaps. She has no technical means to verify log integrity without cryptographic timestamping. She accepts the shipment. | Stage 3: QA manager user requirements |
| 5. Product enters distribution | Compromised product moves into retail or food service supply chain. No recall mechanism triggered because no breach was recorded. | Stage 7: R1 — cascade begins |
| 6. Foodborne illness | Consumers become ill. Illness clusters reported days to weeks after the shipment — the chain of evidence is already cold by the time investigators begin tracing the source. | Stage 7: R1 — $10M+ average recall cost |
| 7. Regulatory response | FDA or USDA opens an investigation. The company cannot produce a reliable audit trail — because the logs were falsified, the chain of custody documentation is worthless. Regulatory violation confirmed. | Stage 7: B1 control gap |
| 8. Legal and financial exposure | Civil liability for illness. Potential criminal charges if negligence is proven. Recall costs. Supply chain partners terminate contracts. | Stage 7: R1 financial risk score 20 |
| 9. Systemic impact | Reputational damage reduces consumer trust in the broader supply chain. If the attacker is nation-state, the incident is a proof-of-concept for food supply disruption at scale. | Stage 1: food as critical infrastructure |

**Where controls interrupt this cascade:**
- **A3 (cryptographic timestamping)** — attacker cannot produce a falsified log that passes integrity verification → cascade stops at Stage 2
- **A2 (network segmentation + allowlisting)** — signal injection requires physical access or network compromise first
- **B3 (gap detection policy)** — any log discontinuity triggers investigation → cascade stops at Stage 3
- **QA manager organoleptic inspection** — last line of defense if all technical controls fail

---

## Cascade 2: Availability Attack

**Trigger:** An adversary targets a distribution company during a high-value period. Goal is not contamination but economic disruption and loss of chain of custody documentation.

| Stage | Event | PASTA Reference |
|---|---|---|
| 1. Cellular jamming or gateway attack | Attacker broadcasts interference on cellular frequency covering the truck's route, or removes the gateway SIM. The truck goes offline. In a rural corridor, this is indistinguishable from a dead zone. | Stage 4: Cellular threats; Stage 5: deniable by design |
| 2. Monitoring blackout | Logistics coordinator loses real-time visibility. No alerts, no location updates, no temperature readings. The silence looks like a coverage gap — no immediate investigation triggered. | Stage 6: Scenario 2 — blind the system |
| 3. Refrigeration failure undetected | During the blackout window, the reefer unit malfunctions. No alert reaches the coordinator. Driver may not notice if cab temperature display is separate from cargo monitoring. | Stage 4: CANbus; Stage 5: CWE-306 |
| 4. No corrective action | Because no alert was received, no rerouting, no emergency maintenance dispatch, no early arrival. The cargo spoils during transit. | Stage 7: R4 |
| 5. Truck arrives at receiving | Gateway comes back online and syncs. Logs show a gap — the offline window — followed by out-of-range temperatures. QA manager sees the damage. | Stage 3: QA manager — logs have gaps |
| 6. Cargo rejected or quarantined | Full shipment rejected. Economic loss: $50,000–$500,000 per truckload depending on product. | Stage 7: R4 narrative |
| 7. Chain of custody gap | The offline window creates an undocumented period. Insurance claim may be denied — the company cannot prove when the failure began. | Stage 7: B3 control gap |
| 8. Regulatory exposure | USDA and FDA require continuous chain of custody documentation. A documented gap is a compliance violation regardless of cause. | Stage 1: FSMA, USDA, HACCP |
| 9. Pattern of attacks | Attacker repeats across multiple routes. Each incident looks like a coverage or equipment issue. The attacker operates with complete deniability. | Stage 4: cellular attacks are highly deniable |

**Military cascade extension:**
In a military context, this attack becomes an operational security event. A jammed supply truck at a forward operating base means the logistics officer makes a decision with no sensor data — only pre-loaded profiles and organoleptic inspection. If the attacker has also physically tampered with the cargo, the officer has no reliable technical basis for his decision. Degraded combat readiness follows.

**Where controls interrupt this cascade:**
- **C1 (offline-first architecture)** — gateway continues logging locally during blackout; gap is documented even if cellular is jammed
- **B3 (gap detection policy)** — offline window triggers investigation at receiving, not automatic acceptance
- **A1 (network segmentation)** — CANbus isolation prevents cellular compromise from pivoting to reefer controller

---

## Cascade 3: Insider Threat — Covering Negligence

**Trigger:** A driver experiences a refrigeration failure during transit. Facing job loss or contract termination, he manipulates the gateway log to hide the excursion before arrival at receiving.

| Stage | Event | PASTA Reference |
|---|---|---|
| 1. Temperature excursion occurs | Reefer unit fails or is compromised during transit. Cargo temperature rises above safe threshold. The gateway logs the excursion accurately — the record exists. | Stage 4: Component 1 — sensors record event correctly |
| 2. Driver intervention | Driver identifies the excursion values and acts before cloud sync. Options: delete the log segment, power cycle the gateway to clear volatile buffer, keep truck offline until manipulation is complete. | Stage 6: Scenario 3 — gateway log attack |
| 3. Cover story constructed | Driver contacts logistics coordinator and reports "sensor malfunction" or "brief gateway fault." This explanation is credible — equipment failures are routine. No investigation triggered at this stage. | Stage 6: Scenario 3 — plausible alternative explanation |
| 4. Gateway syncs manipulated data | When the truck arrives, the gateway syncs a log showing either no excursion (altered values) or a gap with a documented equipment fault explanation. Dashboard shows no breach. | Stage 5: No cryptographic integrity — alteration undetectable |
| 5. Human decision point | QA manager reviews the logs. Data appears clean or the gap is explained. She may be suspicious, but she has no technical means to prove the log was altered. Time pressure forces a decision. | Stage 3: QA manager — decision under time pressure |
| 6. Institutional pressure | QA manager flags the shipment. She faces pressure from an operations manager to accept — the product is needed, the supplier relationship is at stake. The system flagged the problem. A human authority overrode the flag. | Stage 6: Scenario 3 — institutional pressure branch; Stage 7: B2 |
| 7. Product enters distribution | Same end state as Cascade 1. Compromised product accepted, enters supply chain. | Stage 7: R8 — critical |
| 8. Illness and investigation | Foodborne illness traced back to the shipment. Investigators pull the chain of custody record. Without cryptographic integrity verification, the falsification may never be proven. | Stage 7: B1 control gap |
| 9. Legal exposure | Unlike an external attack, a documented insider action creates criminal liability: fraud, falsification of records, potential manslaughter charges. The company faces additional exposure for failure to maintain tamper-evident records. | Stage 7: R8 — criminal liability |
| 10. Systemic organizational impact | If the insider cover-up pattern is tolerated due to operational pressure, it becomes a cultural norm. The human detection layer — the QA manager — is progressively undermined. The most resilient part of the system erodes. | Stage 4: Component 6 — human as threat vector and decision point |

**Where controls interrupt this cascade:**
- **A3 (cryptographic timestamping)** — log entries signed at creation; deletion or alteration produces a detectable integrity failure → cascade stops at Stage 2
- **B1 (tamper-evident audit trail)** — append-only log means driver cannot delete or alter entries → cascade stops at Stage 2
- **B2 (QA manager decision authority)** — operations manager override must be separately logged with identity and justification → institutional pressure documented, accountability preserved
- **B3 (gap detection policy)** — log gap treated as a breach event, not an equipment fault

---

## Cross-Cascade Summary — Where the Framework Pays Off

| Control | Cascade 1 | Cascade 2 | Cascade 3 | PASTA Stage |
|---|---|---|---|---|
| Cryptographic timestamping (A3) | Stops at Stage 2 | — | Stops at Stage 2 | Stage 7 |
| Network segmentation (A1) | Raises attacker skill floor | Prevents CANbus pivot | — | Stage 7 |
| Gap detection policy (B3) | Stops at Stage 3 | Eliminates deniability | Triggers investigation | Stage 7 |
| Tamper-evident audit trail (B1) | — | Preserves compliance record | Stops at Stage 2 | Stage 7 |
| QA manager authority protection (B2) | — | — | Stops at Stage 6 | Stage 7 |
| Offline-first architecture (C1) | — | Stops availability impact | — | Stage 7 |
| Organoleptic inspection | Last defense if all else fails | — | Last defense if all else fails | Stage 3 |

---

> **The most important insight across all three cascades:** The consequence only materializes because a human trusted corrupted, absent, or pressured data. Every technical control in this framework exists to ensure that when a human makes a decision, the data behind it is trustworthy. When the technical layer fails, the human detection layer — a trained QA manager, a military logistics officer, a compliance officer with decision authority — is the last line of defense. The system's job is to make that human as effective as possible, not to replace her.

---

*[← Risk & Impact Analysis](05-risk-and-impact.md) | [↑ Back to Main README](../README.md)*
