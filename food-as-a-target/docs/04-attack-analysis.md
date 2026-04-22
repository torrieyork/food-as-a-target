# Attack Analysis — PASTA Stage 6
### Food as a Target: IoT Security Risk Framework for Cold Chain Infrastructure
[← Back to Main README](../README.md)

---

Stage 6 translates the threat landscape (Stage 4) and protocol vulnerabilities (Stage 5) into concrete, feasible attack paths. Each attack tree maps how an attacker reaches their goal — including the human as both a threat vector and a decision point.

**Reading the trees:**
- **OR node** — attacker needs only ONE of these paths to succeed
- **AND node** — attacker needs ALL of these steps in sequence
- **[H]** — human is the active element at this node (threat vector or decision point)
- **[T]** — technical control failure at this node

---

## Scenario 1: Data Integrity Attack
**Attacker goal:** Make a compromised shipment appear acceptable so the QA manager accepts it

```
GOAL: QA manager accepts spoiled/compromised product
│
├── OR ─ [CORRUPT THE DATA]
│   ├── OR ─ Attack the sensor layer [T]
│   │   ├── Signal injection via Modbus (no auth — write false temp values directly)
│   │   ├── BLE spoofing (BLESA/BIAS — impersonate sensor, feed false readings to gateway)
│   │   └── Replay attack (capture clean readings from prior trip, replay during compromised trip)
│   │
│   ├── OR ─ Attack the gateway [T]
│   │   ├── Physical access → tamper with locally stored logs before cloud sync
│   │   └── Remote exploitation of unpatched CVE → full gateway control → alter log values
│   │
│   ├── OR ─ Attack the cloud backend [T]
│   │   ├── Stolen credentials (phishing) → modify stored records directly
│   │   ├── Unsecured API endpoint → inject false readings via crafted API calls
│   │   └── Misconfigured cloud storage → direct database access, no credentials needed
│   │
│   └── OR ─ Insider manipulates logs [H]
│       ├── Driver deletes gateway log segment covering the excursion window
│       ├── Driver alters gateway values to show in-range temps
│       └── Insider with cloud access modifies records after the fact
│
└── AND ─ [DEFEAT THE HUMAN DETECTION LAYER]
    ├── OR ─ Data looks clean and complete → QA manager trusts it [H]
    │   ├── No gaps in the log (replay attack produces this)
    │   ├── No threshold breaches visible (corrupted values stay within safe range)
    │   └── Dashboard shows green status → no reason for additional inspection
    │
    ├── OR ─ Alert fatigue has degraded her vigilance [H]
    │   ├── Prior false positives have conditioned her to dismiss anomalies
    │   └── Time pressure (45-60 min window) forces fast decision on ambiguous data
    │
    └── OR ─ Product passes organoleptic inspection [H]
        ├── Contamination is not visually/olfactory detectable
        └── Attacker specifically targeted contamination that passes sensory inspection
```

**Consequence cascade:** Compromised product accepted → enters distribution → foodborne illness → product recall → regulatory investigation → civil liability → reputational damage → supply chain disruption

> The attack does not succeed by defeating the technology. It succeeds by defeating a human who trusted corrupted technology. Every technical control in this framework exists to make data trustworthy enough that the QA manager's judgment is valid.

---

## Scenario 2: Availability Attack
**Attacker goal:** Eliminate monitoring visibility during transit — or overwhelm decision-making with false alerts

```
GOAL: Monitoring visibility eliminated or decision-making degraded during transit
│
├── OR ─ [BLIND THE SYSTEM — eliminate data flow]
│   ├── OR ─ Attack the cellular connection [T]
│   │   ├── Signal jamming (broadcast interference — deniable, looks like rural dead zone)
│   │   ├── SIM cloning (carrier kicks truck offline — looks like network error)
│   │   └── IMSI catching / fake cell tower → full MitM, then drop connection
│   │
│   ├── OR ─ Attack the gateway [T]
│   │   ├── Physical destruction (obvious, high-risk for attacker)
│   │   ├── SIM card removal (cuts cellular — simple, fast, low-risk with truck access)
│   │   ├── DDoS gateway processor (flood with traffic until it stops responding)
│   │   └── Delete locally stored logs (creates suspicious gap at receiving)
│   │
│   └── OR ─ Insider creates plausible offline window [H]
│       ├── Driver disables cellular intentionally (claims dead zone)
│       └── Driver reports "gateway malfunction" — covers log gap with plausible explanation
│
└── OR ─ [OVERWHELM THE SYSTEM — drown real alerts in noise]
    ├── OR ─ Attack the dashboard [T]
    │   ├── Alert flooding → inject false alerts for multiple trucks simultaneously
    │   └── Alert suppression → silence alerts for target truck once inside dashboard
    │
    └── AND ─ Human decision-making degrades under overload [H]
        ├── Coordinator receives 50+ simultaneous alerts → triage failure
        ├── Real temperature excursion alert dismissed as another false positive
        └── No escalation protocol → issue goes unresponded until truck arrives
```

**Consequence cascade (blind path):** No visibility → real refrigeration failure undetected → no corrective action → full cargo lost → economic loss + chain of custody gap + regulatory violation

**Military cascade extension:** In a contested environment, cellular jamming is an expected adversary move. A jammed supply truck at a forward operating base forces the logistics officer to make a decision with no sensor data — only pre-loaded profiles and organoleptic inspection. If the attacker has also tampered with the cargo, the officer has no reliable technical basis for his decision. Degraded combat readiness follows.

> **Offline-first is not a feature. It is a countermeasure to this attack tree.**

---

## Scenario 3: Insider Threat — Covering Negligence
**Attacker goal:** Driver hides evidence of a temperature excursion to protect his job, comply with pressure, or act on financial incentive

```
GOAL: Temperature excursion hidden — spoiled product accepted at receiving
│
├── AND ─ [EXCURSION OCCURS]
│   ├── Refrigeration unit fails or is disabled (mechanical or intentional)
│   └── Cargo temperature rises above safe threshold during transit
│
└── AND ─ [DRIVER COVERS IT]
    ├── OR ─ Attack the gateway log [H + T]
    │   ├── Delete the log segment covering the excursion window
    │   │   → Gap in log → driver claims gateway malfunction
    │   ├── Alter stored values if gateway interface allows write access
    │   │   → Log shows in-range temperatures — no indication of excursion
    │   └── Power cycle the gateway during excursion (clears volatile buffer)
    │
    ├── OR ─ Prevent cloud sync until cover story is in place [H]
    │   ├── Keep truck offline until excursion window passes
    │   └── Time the arrival — sync data only after gateway log has been manipulated
    │
    ├── OR ─ Create a plausible alternative explanation [H]
    │   ├── Report "sensor malfunction" or "gateway error" to logistics coordinator
    │   ├── Blame route conditions (extreme external heat, long stop in sun)
    │   └── Claim refrigeration unit had a "brief fault that self-corrected"
    │
    └── AND ─ [QA MANAGER MUST ACCEPT THE COVER STORY] [H]
        ├── OR ─ Log gap accepted as equipment fault, not tampering [H]
        │   ├── No anomaly detection flags the gap as suspicious
        │   └── QA manager has no baseline to compare against
        │
        ├── OR ─ Product passes inspection [H]
        │   ├── No obvious signs of spoilage
        │   └── Time pressure forces accept decision despite incomplete data
        │
        └── OR ─ Institutional pressure overrides QA judgment [H]
            ├── Shortage situation — receiving location needs the product now
            ├── Supplier relationship pressure — reject means losing the account
            └── QA manager's report is overridden by operations manager
                → System flagged the problem. Human hierarchy overrode the flag.
```

**The institutional pressure branch is the most dangerous path.** The system worked — it flagged the problem. A trained QA manager recognized a suspicious log. But a human authority figure overrode the flag for operational reasons. This is not a technology failure — it is a governance failure. No technical control can prevent a human manager from telling a QA manager to accept a shipment anyway. This is why tamper-evident logging and regulatory documentation exist — they create accountability that survives the pressure in the moment.

---

## Cross-Scenario Comparison

| Factor | Scenario 1: Data Integrity | Scenario 2: Availability | Scenario 3: Insider |
|---|---|---|---|
| Primary target | Data accuracy | Data availability | Data existence |
| Attacker skill floor | Medium | Low (jamming hardware is cheap) | Very low (physical access) |
| Deniability | Very high | Very high (looks like dead zone) | Medium |
| Detection method | Anomaly in log values | Gap in log / alert silence | Gap in log / driver behavior |
| Last human defense | QA organoleptic inspection | Coordinator pattern recognition | QA suspicion + governance |
| Most likely threat actor | Organized crime, nation-state | Nation-state, hacktivist | Insider |
| Military relevance | High | Very high — jamming is standard TTPs | Medium — soldier coercion is documented |

---

## Control Gaps Identified

**1. Log integrity cannot be verified without cryptographic timestamping**
All three scenarios exploit the fact that logs can be altered, deleted, or fabricated without leaving evidence. Cryptographic signing at the point of creation is the single control that closes the replay, log tampering, and insider cover-up vectors simultaneously.

**2. The QA manager is the last line of defense in every scenario**
Her effectiveness depends on three things: trustworthy data, calibrated alerting, and organizational authority to act on her assessment. All three are currently gaps in most food logistics deployments.

**3. Gap detection is more valuable than value monitoring**
Every scenario produces a gap — in log continuity, alert coverage, or chain of custody. A system that treats gaps with the same severity as threshold breaches catches all three attack patterns.

> **The absence of data is as informative as the presence of bad data. Both must trigger investigation.**

---

*[← Vulnerability Analysis](03-vulnerability-analysis.md) | [Next: Risk & Impact Analysis →](05-risk-and-impact.md)*
