# System Architecture & User Requirements
### Food as a Target: IoT Security Risk Framework for Cold Chain Infrastructure
[← Back to Main README](../README.md)

---

## PASTA Stages 1–3: Objectives, Technical Scope, and Decomposition

---

## Context & Goal

The cold chain is not a single trip — it is a **chain of custody** across multiple stages, operators, and environments. Each stage has different sensors, protocols, threat surfaces, and human decision-makers. A security framework that only addresses one stage (e.g., the truck) misses the majority of the attack surface.

**Long-term goal:** Become a leading expert in OT/IoT security for critical food infrastructure — credible enough that government, defense, and industry call on this work when agricultural systems are under attack.

**Why IoT/Cold Chain specifically:**
- IoT systems directly affect food quality, food safety, and continuity of feeding people
- Food and agriculture is one of 16 critical infrastructure sectors designated by CISA
- Deliberate disruption of food supply during conflict can constitute a war crime (Rome Statute, international humanitarian law)
- 90% of OT infrastructure is currently unmonitored — this framework addresses a documented gap

---

## Industry Verticals

### Primary: Commercial Agriculture / Cold Chain
- Commercial food supply (produce, meat, dairy)
- Farm-to-distribution center pipeline
- Regulatory environment: FDA Food Safety Modernization Act (FSMA), USDA, HACCP
- Threat actors: nation-state, cybercriminal, insider, organized crime

### Secondary: Military Logistics
- Deployed troop rations, forward operating base supply
- Medical/pharmaceutical cold chain (vaccines, blood products)
- Compliance requirements: CMMC, NIST 800-171, DoD supply chain security standards
- Unique challenge: food supply security is an **operational security issue**, not just health and safety

### Transferable Domains
Pharmaceutical cold chain, medical device monitoring, industrial process control, energy infrastructure

---

## Multi-Stage System Architecture

### Stage 1: Production / Harvest
| Element | Detail |
|---|---|
| Location | Farm, processing facility |
| Sensors | Ambient temperature & humidity, soil moisture, CO₂/ethylene |
| Protocols | Modbus RTU, Zigbee/Z-Wave, LoRaWAN |
| Operators | Farm managers, processing supervisors |
| Key threat surface | Sensor tampering, unauthorized access to local control systems |

### Stage 2: Cold Storage / Warehouse
| Element | Detail |
|---|---|
| Location | Refrigerated warehouse, distribution hub |
| Sensors | Industrial temperature controllers, door/access sensors, occupancy sensors, power/refrigeration monitors |
| Protocols | Modbus TCP, BACnet, MQTT over LAN |
| Operators | Warehouse managers, compliance officers |
| Key threat surface | BACnet/Modbus unauthenticated by default, network segmentation failures |

### Stage 3: Transit (Refrigerated Truck)
| Element | Detail |
|---|---|
| Location | Refrigerated transport vehicle |
| Sensors | Temperature/humidity, door sensors, GPS, vibration/shock, reefer unit controller |
| Protocols | Modbus RTU, CANbus (reefer), BLE (sensor-to-gateway), 4G/5G cellular (gateway-to-cloud), MQTT over TLS |
| Operators | Driver (local alerts), logistics coordinator (remote dashboard), QA manager (at receiving) |
| Key threat surface | BLE spoofing, MQTT without TLS, CANbus manipulation, cellular interception |

### Stage 4: Receiving / Distribution Center
| Element | Detail |
|---|---|
| Location | Distribution center, retail receiving dock |
| Sensors | Temperature verification probes, weight/scale sensors, barcode/RFID readers |
| Protocols | USB/serial (probes), RFID EPC Gen2, REST API (WMS/ERP integration) |
| Operators | QA manager (accept/reject decision), compliance officer (audit trail) |
| Key threat surface | Log tampering before sync, RFID cloning, API injection into WMS |

---

## Gateway as Integration Layer

The gateway is the critical interface point — it speaks multiple protocols and translates them into a unified data stream.

```
[Sensor Layer]           [Gateway]                  [Cloud]
Modbus RTU    ──────────►
CANbus        ──────────► Protocol Translation    MQTT/TLS ──► Cloud Backend
BLE sensors   ──────────► + Local Storage     ──►             ──► Dashboard
Wired I/O     ──────────►
```

**Key architecture principle — store-and-forward:**
1. Collects data from sensors via their native protocols
2. Validates and timestamps locally
3. Stores locally during offline periods
4. Transmits to cloud when connected (MQTT over TLS)
5. Generates local alerts independent of cloud connectivity

---

## Key Stakeholders

| Role | Stage | Decision Authority |
|---|---|---|
| Farm manager | Production | Harvest timing, storage conditions |
| Warehouse supervisor | Cold storage | Accept/reject from farm, pre-ship condition |
| Logistics coordinator | Transit | Route monitoring, real-time alerts |
| Driver | Transit | Local alert response (temp spike, door breach) |
| QA manager | Receiving | **Final accept/reject/quarantine decision** |
| Compliance officer | All stages | Audit trail, regulatory reporting |
| Military logistics officer | All stages (military) | Chain of custody under operational conditions |

---

## User Requirements: Commercial QA Manager

**Context:** A QA manager at a distribution center. A refrigerated truck just arrived after being offline for hours during transit. She has **45–60 minutes** to make a confident accept/reject/quarantine decision.

**What she needs from the system:**
- Full time-stamped temperature log for the entire transit duration
- Automatic flagging of threshold breaches AND data gaps
- Simple visual status (green/yellow/red) — not raw data tables
- Product-aware alerts calibrated to the specific product being transported
- Decision recommendation with confidence level
- Prompted organoleptic inspection when data is suspicious or incomplete
- Automatic decision record that logs what data was visible at the time of her decision

**Critical design principle:**
> Most security tools get ignored because they were built for the security team, not for the operator. A QA manager who trusts and uses the system is the best defense against consequence cascades — if she ignores it, even the best threat detection is worthless.

**The human detection layer:**
Organoleptic inspection (sight, smell, touch) is a formally recognized quality assessment method in USDA, FDA, and HACCP protocols. A trained QA manager is a human detection layer that an attacker has to account for. The system doesn't replace her — it makes her more effective and more protected.

---

## User Requirements: Military Logistics Officer

**Context:** A military logistics officer at a forward operating base. A supply truck just arrived after traveling through a denied communications zone for 4+ hours. Decision window: **20–30 minutes**.

| Factor | Commercial QA Manager | Military Logistics Officer |
|---|---|---|
| Environment | Loading dock, controlled | Field, harsh weather, high alert |
| Time to decide | 45–60 minutes | 20–30 minutes |
| Connectivity | Spotty (store-and-forward) | Fully denied for hours |
| Consequences of wrong call | Recalls, liability, USDA violation | Degraded combat readiness, potential court-martial |
| Rejection option | Always available | May not be an option |

**Four decision options (not three):**
- ✅ Accept
- ❌ Reject
- ⚠️ Quarantine
- 🔧 **Mitigate & Use** — consume with precautions (higher cooking temp, limited quantities, health monitoring)

**Offline-first requirement:**
> A system that degrades when offline is a system an adversary can disable by jamming or disrupting cellular. Offline-first is a security requirement, not just a technical one.

**Chain of custody (military evidentiary standard):**
Every record may be used in a military investigation or court-martial proceeding. Required fields: origin location, driver identity, time/date of every handoff, sensor readings at each transfer point, identity of the officer making the final decision, mitigation actions taken, tamper-evident signature.

---

*[← Back to Main README](../README.md) | [Next: Threat Analysis →](02-threat-analysis.md)*
