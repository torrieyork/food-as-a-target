# Threat Analysis — PASTA Stage 4
### Food as a Target: IoT Security Risk Framework for Cold Chain Infrastructure
[← Back to Main README](../README.md)

---

## Threat Actor Profile

| Threat Actor | Primary Motivation |
|---|---|
| Insider | Cover negligence, sabotage, revenge |
| Nation-state | Critical infrastructure disruption, war strategy |
| Competitor | Industrial espionage, supply chain intelligence |
| Hacktivist | Ideological disruption (animal rights, environmental, anti-corporate) |
| Cybercriminal | Ransomware, extortion, financial gain |
| Organized crime | Food fraud, product substitution, cover-up at scale |

**Note on organized crime:** Already a multi-billion dollar problem in the food industry without cyber involvement. IoT manipulation makes food fraud significantly harder to detect — goal is to look *normal*, not cause disruption. Patient, low-and-slow attack pattern.

---

## Component 1: Sensors

**OT-native attacks (most realistic for this environment):**

| Attack | Method | Why It's Dangerous |
|---|---|---|
| Signal injection | Attacker speaks the sensor's native protocol (Modbus, BLE) and injects false readings directly | Looks like legitimate sensor traffic, very hard to detect |
| Replay attack | Attacker records normal readings from a previous trip and plays them back during a compromised trip | Logs look perfectly normal — real data from another time; undetectable without cryptographic timestamping |
| Firmware manipulation | Attacker modifies sensor firmware so it permanently reports attacker-controlled values | Survives reboots, appears completely normal, no external signs of compromise |

**IT-adjacent attacks:**

| Attack | Method | STRIDE Category |
|---|---|---|
| Man-in-the-Middle | Intercept data between sensor and gateway, replace with false readings | Tampering |
| Physical destruction | Sensor disabled or removed | Denial of Service |
| DDoS | Flood sensor or gateway with traffic | Denial of Service |

**Business impact — human dimension:**
> A replay attack on temperature sensors causes a QA manager to review perfect logs and accept a compromised shipment. The attacker didn't just beat the technology — they beat the human decision maker by making the data look trustworthy.

---

## Component 2: Gateway Device

The gateway is the highest value target on the truck — it controls what data is stored, what gets transmitted, and what the QA manager ultimately sees.

| Attack | Method | STRIDE Category |
|---|---|---|
| SIM swapping | Physical removal/replacement of SIM — cuts cellular, logistics coordinator loses real-time visibility | Denial of Service |
| Log deletion | Delete stored sensor data — no logs available at receiving | Denial of Service |
| Log tampering | Alter stored temperature values — logs appear complete and clean | Tampering |
| Remote CVE exploitation | Exploit known vulnerability over cellular — attacker gains full gateway control without physical access | Elevation of Privilege |

**OT reality:** Patching a truck gateway means taking the vehicle out of service, testing against proprietary sensor protocols, and getting vendor approval — most operators skip it. Known CVEs sit unpatched indefinitely.

---

## Component 3: Cellular Connection

| Attack | Method | STRIDE Category |
|---|---|---|
| Signal jamming | Broadcast interference on cellular frequency — deniable, looks like a rural dead zone | Denial of Service |
| SIM cloning | Duplicate truck's SIM credentials — carrier kicks truck offline, looks like a network error | Denial of Service |
| IMSI catching / fake cell tower | Attacker deploys fake cell tower — truck connects to it, full MitM at cellular level | Tampering / Information Disclosure |

**Key characteristic:** All three are highly deniable. A 2-hour offline window during a rural route raises no immediate suspicion.

---

## Component 4: Cloud Backend

Unlike truck components, the cloud backend is internet-facing and always on. A cloud attack affects every shipment simultaneously.

| Attack | Method | STRIDE Category |
|---|---|---|
| Stolen credentials | Phishing targeting logistics staff — attacker accesses all shipment data as legitimate user | Spoofing |
| API exploitation | Unsecured API endpoints — read or modify logs without credentials | Tampering / Information Disclosure |
| Misconfigured cloud storage | Publicly accessible database — no active attacker needed | Information Disclosure |
| Mass log deletion/alteration | Once inside — alter records across all trucks simultaneously | Tampering / Denial of Service |

> **Only the cloud backend enables a simultaneous, multi-truck, nationwide attack. Every other component is limited to one shipment.**

---

## Component 5: Dashboard

| Attack | Method | STRIDE Category |
|---|---|---|
| Default credentials | Vendor default login never changed — attacker logs straight in | Spoofing |
| Alert suppression | Silence alerts for target truck — coordinator has no reason to investigate | Tampering |
| Alert flooding | Inject false alerts across all trucks — real alert buried in the noise | Denial of Service |
| Session hijacking | Steal active session token — take over coordinator's view without credentials | Spoofing / Elevation of Privilege |

**Industry reality:** Dashboard software in food logistics was built for operational visibility, not security. Most small to mid-size companies have no dedicated security team. Alert administration controls are standard in enterprise IT but nearly absent in OT food logistics environments.

---

## Component 6: Human Operators

Humans are both a **threat vector** and a **decision point**.

**Intentional threats (insider):**

| Motivation | Example |
|---|---|
| Self-preservation | Driver hides refrigeration failure to protect job/reputation |
| Financial incentive | Paid by competitor or organized crime to manipulate logs |
| Coercion / blackmail | Pressured by external attacker into cooperating |

**Unintentional threats:**

| Mistake | Result |
|---|---|
| Phishing | Fake login notification tricks coordinator into entering real credentials |
| Unattended session | Dashboard left open and logged in — anyone nearby has full access |
| Password reuse | One breached account unlocks multiple systems |

> **The human layer is simultaneously the most exploitable vulnerability and the most resilient defense in the system. An attacker who can manipulate a trusted insider bypasses every technical control. A trained operator who trusts her instincts can catch what the technology misses.**

---

*[← Architecture & Requirements](01-architecture-and-requirements.md) | [Next: Vulnerability Analysis →](03-vulnerability-analysis.md)*
