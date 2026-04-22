# Risk & Impact Analysis — PASTA Stage 7
### Food as a Target: IoT Security Risk Framework for Cold Chain Infrastructure
[← Back to Main README](../README.md)

---

Stage 7 translates the threat landscape and vulnerability analysis into a prioritized risk picture and actionable control recommendations. The intended reader is a security executive (CISO or equivalent) making investment decisions — commercial food logistics and military logistics addressed separately where stakes and compliance requirements diverge.

**Risk formula:** Threat × Consequences × Vulnerability = Financial Risk *(McMickle/Lindseth)*

---

## Risk Matrix

| # | Risk | Threat Likelihood (1–5) | Consequence Severity (1–5) | Financial Risk Score | Primary Threat Actor |
|---|---|---|---|---|---|
| R1 | Sensor data manipulation — spoiled product accepted | 4 | 5 | 20 | Organized crime, insider |
| R2 | Gateway log tampering — chain of custody falsified | 4 | 5 | 20 | Insider, cybercriminal |
| R3 | Cloud backend compromise — mass multi-shipment log alteration | 2 | 5 | 10 | Nation-state, cybercriminal |
| R4 | Cellular jamming / availability attack — monitoring blinded during transit | 3 | 4 | 12 | Nation-state, hacktivist |
| R5 | BLE sensor spoofing — false readings injected at gateway | 3 | 4 | 12 | Organized crime, insider |
| R6 | BACnet warehouse manipulation — cold zone temperature altered silently | 3 | 4 | 12 | Insider, nation-state |
| R7 | Dashboard alert suppression — real alerts silenced | 3 | 4 | 12 | Cybercriminal, insider |
| R8 | Insider negligence cover-up — excursion hidden by driver | 5 | 4 | 20 | Insider |
| R9 | Phishing / credential theft — logistics staff account compromised | 5 | 3 | 15 | Cybercriminal |
| R10 | LoRaWAN replay attack — false farm sensor readings forwarded | 2 | 3 | 6 | Competitor, nation-state |

**Score interpretation:**
- **16–25: Critical** — unacceptable residual risk, immediate control investment required
- **9–15: High** — significant exposure, controls should be prioritized in next budget cycle
- **1–8: Medium/Low** — managed risk, compensating controls or monitoring sufficient

---

## Priority Risk Narratives

### R1 + R2 + R8 — Data Integrity / Insider (Score: 20) — CRITICAL

These three risks share the same consequence: a QA manager accepts a compromised shipment because the data she depends on has been falsified. Whether the falsification comes from a remote attacker injecting sensor readings, a gateway with tampered logs, or a driver covering a negligence event, the end state is identical — bad product enters the food supply.

R1 and R2 require technical controls (cryptographic integrity, anomaly detection). R8 requires governance controls (audit trail authority, tamper-evident logging, QA manager protected from institutional override). Both are necessary. Neither alone is sufficient.

**Commercial consequence:** Foodborne illness → product recall → FDA/USDA investigation → civil liability → reputational damage. Average cost of a major U.S. food recall exceeds $10 million before litigation.

**Military consequence:** Compromised rations reach a forward operating base → degraded combat readiness → mission failure risk. In a contested environment, this is an adversary capability, not an accident. CMMC non-compliance compounds liability.

---

### R3 — Cloud Backend Compromise (Score: 10) — HIGH

The cloud backend is the only component where a single attack affects every shipment simultaneously. All other risks are scoped to one truck or one facility. A nation-state actor with cloud access can falsify audit trails for hundreds of QA decisions at once — making a coordinated, nationwide food safety failure indistinguishable from routine operations until illness clusters surface.

**Commercial consequence:** Complete loss of audit trail integrity → inability to trace contamination → inability to prove regulatory compliance → potential criminal exposure for executives → supply chain shutdown during investigation.

**National security implication:** This is the scenario where food supply disruption becomes a national security event, not a food safety event.

---

### R4 + R7 — Availability / Alert Suppression (Score: 12) — HIGH

Availability attacks don't require sophisticated access — jamming hardware is commercially available, and default credentials on dashboard software are the norm in food logistics. Alert suppression is particularly dangerous because it produces no anomaly — the system appears to be working normally while real alerts are silenced.

**Commercial consequence:** Refrigeration failure during transit goes undetected → full cargo lost → $50,000–$500,000 per truckload → no documentation to support insurance claim.

**Military consequence:** Jamming is a standard adversary tactic in contested environments. The offline-first architecture requirement exists as a direct countermeasure.

---

### R9 — Phishing / Credential Theft (Score: 15) — HIGH

Phishing is the most common initial access vector across all industries and requires no OT-specific knowledge. A logistics coordinator's credentials unlock the cloud dashboard, shipment records, and potentially the MQTT broker. This risk is the entry point for R3 — cloud compromise almost always begins with stolen credentials.

**Commercial consequence:** Single phished employee → full cloud access → all downstream data integrity risks become exploitable without any technical attack on OT infrastructure.

---

## Control Recommendations

### Section A — Technical Controls

**A1. Network segmentation — isolate OT protocols from corporate IT**
Modbus, BACnet, and CANbus networks must be physically or logically isolated from internet-connected systems. An attacker who compromises a corporate workstation should have no path to the warehouse BACnet controller or the truck gateway.
*Example: VLAN separation between office network and cold storage BACnet segment, with a unidirectional data diode for monitoring traffic.*

**A2. Application-layer authentication for Modbus and BACnet**
Since Modbus and BACnet have no native authentication, compensating controls must be applied at the network layer. Allowlist-based firewall rules restrict which devices can send commands to which controllers.
*Example: Industrial DMZ with deep packet inspection appliance (Claroty, Dragos, or Nozomi Networks) enforcing communication allowlists.*

**A3. Cryptographic timestamping for all sensor log entries**
Log entries must be signed at the point of creation so any alteration — at the gateway, in transit, or in the cloud — is detectable. This is the single control that closes the replay attack, log tampering, and insider cover-up vectors simultaneously.
*Example: Gateway firmware generates HMAC-signed log entries using a hardware security module (HSM); signature chain is verified at cloud ingestion.*

**A4. MQTT hardening — TLS + authentication mandatory**
MQTT brokers must require TLS encryption and client authentication. Anonymous connections must be disabled. Topic-level access controls must prevent any client from subscribing to all topics.
*Example: Eclipse Mosquitto with TLS 1.3, client certificate authentication, and ACL file restricting topic access by device identity.*

**A5. BLE pairing enforcement and firmware patch management**
BLE sensors must use Secure Simple Pairing (BLE 4.2+). Sensors affected by SweynTooth-class vulnerabilities must be prioritized for firmware updates or hardware replacement.
*Example: Maintain a sensor firmware inventory; establish a vendor notification process for CVE disclosures affecting deployed chipsets.*

**A6. OT asset discovery and continuous monitoring**
You cannot protect assets you don't know exist. An OT asset inventory tool provides continuous visibility into what devices are on the network, what they're communicating with, and when behavior changes. A live demonstration of OT asset topology visualization observed at OTSecCon 2026 showed real-time mapping of manufacturing environments — the same capability applied to cold chain infrastructure would surface unauthorized devices, unexpected communication paths, and configuration changes before they become incidents.
*This directly addresses the practitioner finding cited at OTSecCon 2026: approximately 90% of OT infrastructure is currently unmonitored.*

---

### Section B — Governance Controls

**B1. Tamper-evident audit trail with legal standing**
Every decision record must be logged with the data visible at the time of decision, the decision-maker's identity, a timestamp, and a cryptographic signature. The log must be append-only — no record can be altered or deleted after creation.
*This protects the QA manager legally and creates accountability that survives institutional pressure to accept a shipment.*

**B2. QA manager decision authority — protected from operational override**
The QA manager's accept/reject decision must be documented as final and independent. Any override by an operations manager must be logged separately with identity and documented justification.
*Without this control, Scenario 3 (insider cover-up) can be executed by management, not just the driver.*

**B3. Gap detection policy — absence of data treated as a breach event**
Any gap in sensor log continuity must trigger the same investigation protocol as a threshold breach. A two-hour offline window is not automatically a dead zone — it is an unverified period that must be explained and documented before a shipment can be accepted.
*This closes the deniability window exploited in all three attack scenarios.*

**B4. MFA and least-privilege access for all cloud systems**
Every account with access to the cloud dashboard, MQTT broker, or audit trail database must use multi-factor authentication. Access must be scoped to the minimum required.
*Primary control against R9 (credential theft) and the entry point for R3 (cloud compromise).*

**B5. Alert administration out-of-band notification**
Any change to alert thresholds, notification settings, or user permissions must trigger an out-of-band notification (email or SMS) to IT and QA leadership through a channel completely separate from the dashboard.
*Standard in enterprise IT but nearly absent in OT/food logistics — a consistent finding in security assessments of this sector.*

---

### Section C — Military-Specific Controls

**C1. Offline-first architecture — full function without connectivity**
The system must be designed to operate completely offline, not merely tolerate connectivity loss. All product profiles, thresholds, and decision frameworks must be pre-loaded before departure. The gateway must store a complete, tamper-evident log throughout transit regardless of connectivity.

> *Cellular jamming is a standard adversary tactic in contested environments. A system that degrades when offline is a system an adversary can disable on demand.*

**C2. Hardened decision interface for field conditions**
The decision interface must function under conditions a commercial QA manager will never face: gloves, bright sun, low light, high stress, 20–30 minute decision window. Interface complexity acceptable in a loading dock is a safety risk in the field.
*Controls: minimal screen elements, high-contrast display, large touch targets, no dependency on cloud connectivity for decision support.*

**C3. Mitigate & Use decision with documented mitigation logging**
A fourth decision option is required for the military vertical: Mitigate & Use — consume the shipment with documented precautions (higher cooking temperatures, limited quantities, health monitoring). Every mitigation action must be logged with the officer's identity and timestamp for potential court-martial proceedings.

**C4. Tamper-evident logging to military evidentiary standard**
Required fields: origin facility, driver identity (name + ID), time and GPS of every handoff, sensor readings at each transfer point, identity of the officer making the final decision, and all mitigation actions taken. Log entries must be cryptographically signed and append-only.

**C5. CMMC and NIST 800-171 compliance baseline**
Any system handling military supply chain data must meet CMMC requirements and NIST 800-171 controls for protecting Controlled Unclassified Information (CUI). These are contractual requirements for DoD suppliers — not optional enhancements.

---

## Residual Risk Summary

Even with all recommended controls implemented, residual risk remains in three areas:

**1. Physical insider access**
No technical control prevents a driver with physical access to the truck from damaging hardware. Governance controls create accountability but cannot prevent the act.

**2. Organoleptic inspection limitations**
The human detection layer cannot catch contamination with no visible, olfactory, or textural indicators. Some bacterial contamination and chemical adulteration passes sensory inspection. This is the residual risk that makes cryptographic log integrity the most critical single control — when senses fail, the log must be trustworthy.

**3. Nation-state cellular attack**
IMSI catching and coordinated cellular jamming at scale are nation-state capabilities that individual organizations cannot defend against unilaterally. Offline-first architecture mitigates the impact but cannot prevent the attack. This risk requires sector-wide coordination (CISA, USDA, DoD).

> *The goal of this framework is not zero risk — it is informed, prioritized risk management. Every control recommendation above is designed to interrupt the consequence cascade before it reaches the QA manager's decision point. When that fails, the human detection layer is the last defense. The system's job is to make that human as effective as possible.*

---

*[← Attack Analysis](04-attack-analysis.md) | [Next: Consequence Cascades →](06-consequence-cascades.md)*
