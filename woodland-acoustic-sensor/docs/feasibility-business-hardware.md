# Feasibility: business and hardware

**Question:** Can this work, what are the financials, and how much can be done in-house?
**Basis:** Phase 0 is unanswered, so this uses the Phase 0 defaults (parts ≤ £150 per node, kit of 8 nodes plus 1 gateway, UK pilot). Prices are from my own knowledge as of 2026, not live quotes. Treat them as ±25 % and confirm them in Phase 2.

---

## 1. Verdict

**Yes, it can work as a product, with three conditions.** Each technical building block is already proven in the field. The risk is in combining them on a solar budget and in proving that customers will pay.

| Area | Verdict | Condition |
|---|---|---|
| Detecting gunshots and chainsaws | Proven (AudioMoth gunshot work in Belize, RFCx chainsaw alerts, commercial urban gunshot systems) | Must handle **legal shooting** (shoot days, stalking, foxing). An alert alone means "someone fired a gun", not "poacher". |
| Detecting vehicles | Feasible | Range of about 200–500 m, so it only works on tracks and rides near nodes |
| Detecting human voices | **Weak** | Speech only carries 50–100 m. Useful at gates and car parks, not across a whole wood. GDPR burden as well. |
| Wildlife ID | Feasible **in the cloud or offline**, not on-node | Bird classifiers are too big for a µW-class MCU, and BirdNET is non-commercial (see the [risk register](risk-register.md)) |
| TDOA localisation | Feasible for impulsive sounds | 10–25 m inside the node array, poor outside it (Section 3) |
| Alert within 60 s | Easy | LoRa airtime is under 1 s, gateway to cloud to push notification is a few seconds |
| Solar through UK winter | **Hardest constraint** | Needs a ~0.5 Wh/day node design. It does **not** work with continuous recording on 4 mics under conifer canopy (Section 2). |
| Raw audio "stored locally" | Needs redefining | Continuous mono audio fills a 128 GB card in about 15–30 days. Recommendation: rolling buffer + event clips + scheduled survey windows. |

**Conditions to proceed:**
1. Customer discovery shows people will pay (Section 5 kill criteria), **before** any custom PCB.
2. Field data from one wood shows gunshot and chainsaw range ≥ 500 m and localisation error ≤ 25 m.
3. The measured node power stays ≤ 0.6 Wh/day.

---

## 2. Hardware feasibility

### 2.1 Power: the deciding calculation

**Node consumption, two design styles** (3.3 V system, averaged over 24 h):

| Block | "Brute force" (always recording, 4 mics, GPS on) | "Low-power" (recommended) | Notes |
|---|---|---|---|
| Microphones | 4 × ~1 mA × 1.8 V ≈ 4 mW | 1 mic always on ≈ 1 mW; array woken only on a trigger | PDM low-power mode |
| MCU + detector | ~15 mW (always computing) | ~5 mW (DMA plus cheap energy/onset trigger, CNN only on trigger) | Cortex-M4F/M33 class |
| microSD writes | 10–20 mW (96 kB/s continuous) | ~1–2 mW (event clips + 2 h/day survey) | SD write bursts are 30–100 mA |
| GPS | ~25 mW (continuous tracking) | ~2–3 mW (periodic fix + PPS discipline of a TCXO, kept on after events) | 1–2 ppm TCXO drifts only ~10–20 µs over 10 s |
| Radio | 15 mW if acting as a **mesh relay** (always listening) | ~0.3 mW (LoRaWAN Class A, star) | **Mesh costs a lot of power.** Prefer a star topology to a mast gateway. |
| Quiescent / regulators | ~2 mW | ~1 mW | |
| **Total** | **~60–70 mW → 1.5–1.7 Wh/day** | **~12–20 mW → 0.3–0.5 Wh/day** | |

**Winter energy available** from a 6 W panel at 70 % system efficiency (MPPT, temperature, dirt, cable). Peak-sun-hour (PSH) figures are my estimates for December on a steeply tilted panel. Check them for the real site with PVGIS (EU JRC, free).

| Location (Dec) | PSH/day | Open sky | Under leafless broadleaf (≈ 30 % transmission) | Under conifer (≈ 10 %) |
|---|---|---|---|---|
| Southern England | ~1.0 | 6 × 1.0 × 0.7 = **4.2 Wh** | **1.3 Wh** | **0.4 Wh** |
| Northern Scotland | ~0.5 | **2.1 Wh** | **0.6 Wh** | **0.2 Wh** |

**Conclusion:**
- The **low-power design (≤ 0.5 Wh/day) works** in the open and under broadleaf, everywhere in the UK.
- Under conifer it needs the panel placed in a ride or gap, or on a 3–5 m cable up the trunk.
- The brute-force design fails under almost any canopy, so I won't pursue it.

**Battery:** for dull spells, plan 14 days with no harvest: 0.5 Wh × 14 = 7 Wh. One LiFePO4 32700 cell (6 Ah × 3.2 V ≈ 19 Wh, ~£8–12) gives about 38 days. LiFePO4 must **not be charged below 0 °C**, so the charger needs NTC-based cut-off. It's still preferred over Li-ion for cycle life and safety in a box strapped to a tree.

### 2.2 Indicative node BOM at 100 units

| Block | Example part (Phase 2 confirms) | £ @100 (approx.) |
|---|---|---|
| MCU / SoC | nRF52840 / nRF5340 or STM32U5 module | 6–10 |
| Microphones (4) | Infineon IM69D130 / IM73A135, or Knowles SPH0641 | 8–12 |
| GPS + antenna | u-blox MAX-M10S + patch | 13–19 |
| LoRa radio | Semtech SX1262 module (868 MHz), pre-certified | 5–8 |
| Storage | 128 GB industrial microSD | 12–25 |
| Battery | LiFePO4 32700 6 Ah | 8–12 |
| Solar panel | 6 W ETFE | 15–25 |
| Charger / power | MPPT LiFePO4 charger (e.g. TI BQ25798 class) + regulators | 4–8 |
| PCB + assembly | 4-layer, JLCPCB/PCBWay or a UK EMS | 15–25 |
| Enclosure | ASA print (~250 g), ePTFE acoustic vents × 4, gaskets, stainless fixings, cable gland | 15–25 |
| Mounting | Strap / bracket / anti-tamper fixings | 5 |
| **Total** | | **≈ £110–175** |

**Gateway** (solar, 4G, mast-mounted): an off-the-shelf certified LoRaWAN gateway plus solar kit ≈ **£350–600**. Buy it rather than build it; that avoids a second RED certification.

### 2.3 Fastest prototype route

| Option | Pros | Cons | Use |
|---|---|---|---|
| **RAK WisBlock modular** (nRF52840 + SX1262 core, GNSS, audio/PDM mic and solar/battery modules) | Low power, LoRaWAN ready, solderless | Mic array geometry is fixed by the modules; limited RAM for ML | **Recommended** for the low-power proof |
| Raspberry Pi Zero 2 W + USB/I²S mics + GPS HAT | Fastest way to record field data and test TDOA; Python | ~1 W, so no solar realism | **Recommended in parallel** for range and accuracy data |
| AudioMoth + separate GPS logger | Cheap, proven acoustics | No real-time radio; GPS sync isn't native | Cheap baseline only |

---

## 3. Expected detection range and localisation

| Source | Source level @1 m (approx.) | Practical detection range in woodland | Node spacing for ≥ 3–4 nodes hearing it | Expected location error inside the array |
|---|---|---|---|---|
| Shotgun / rifle | 150–170 dB peak | 1–2 km | 500–700 m | **10–25 m** |
| Chainsaw | ~110 dB | 0.5–1 km | 350–500 m | 20–50 m (continuous signal, cross-correlation) |
| Vehicle | 70–85 dB | 200–500 m | — | 50 m+ (track-level) |
| Voices | 60–70 dB | 50–100 m | not practical | presence only |

The error comes mostly from sound-speed uncertainty (wind up to ~1.5 %, temperature ~0.2 %/°C) and diffracted first arrivals, not from GPS timing: PPS is ~50 ns, and one sample at 48 kHz is 7 mm. **Coverage:** at 600 m triangular spacing each node covers ~31 ha, so **an 8-node kit covers roughly 150–250 ha** for gunshots. Phase 1 has the full maths.

---

## 4. Financials

### 4.1 Unit economics (default kit: 8 nodes + 1 gateway)

| Item | £ |
|---|---|
| Node parts | 8 × 150 = 1,200 |
| Gateway | 400 |
| Kitting, test, packaging, shipping (~£20/node + £40) | 200 |
| **Kit cost of goods** | **1,800** |
| **Kit retail (suggested)** | **4,800** (2.7× cost → 62 % gross margin) |
| Warranty / returns reserve (5 % of retail) | 240 |

| Subscription per site | £/year |
|---|---|
| Price (alerts, app, 4G data, cloud storage, species report) | **720** (£60/month) |
| 4G SIM data | −60 |
| Cloud hosting + notifications | −60 |
| Support allowance | −100 |
| **Subscription gross profit** | **500 (≈ 70 %)** |

Alternative model: **hardware-as-a-service** at £180–220 per site per month on a 3-year term. The upfront price is easier to sell but needs ~£1,800 of working capital per site. Phase 7 picks one.

**Benchmark:** 8 × 4G trail cameras cost ~£1.5–3k plus ~£80–120/month in SIMs and only cover line of sight. A kit at £4.8k + £60/month covering ~200 ha is competitive. Customer interviews must still prove that.

### 4.2 Three-year model (illustrative; driven by sites sold)

| | Year 1 (pilot) | Year 2 | Year 3 |
|---|---|---|---|
| New sites | 5 (at 50 % pilot price) | 30 | 80 |
| Installed base at year end | 5 | 35 | 115 |
| Hardware revenue | £12k | £144k | £384k |
| Subscription revenue (new sites billed half a year) | £2k | £14k | £54k |
| **Revenue** | **£14k** | **£158k** | **£438k** |
| Cost of goods (hardware + subscription) | £10k | £58k | £161k |
| **Gross profit** | **£4k** | **£100k (63 %)** | **£277k (63 %)** |
| Development cash (prototypes, PCB spins, certification, tooling, legal) | £60k | £20k | £20k |
| Operating costs (people, marketing, insurance) — lean | founder time | £90–120k | £200–250k |
| **Operating result** | **≈ −£56k** | **≈ −£10k to −£40k** | **≈ +£7k to +£57k** |

**Cash needed:** ~**£80–120k** to reach Year-3 break-even. That excludes founder salaries and includes ~£15–25k of stock and working capital for the first 100 nodes. If engineering is contracted out instead of done in-house, add £100–150k (Section 6).

**What the model says:**
- It's a **viable niche hardware business**: break-even at about 100–120 installed sites.
- To get bigger it needs EU expansion, a second revenue line (biodiversity/BNG survey reports, the fieldsports SaaS integration) or the hardware-as-a-service model.
- The figure everything depends on is **30 sites in Year 2**. That's ambitious for a new brand and needs evidence from pilots.

### 4.3 Development budget to a certified product (cash, in-house engineering)

| Item | £ (range) |
|---|---|
| Customer discovery (travel, 20 interviews, Game Fair visit) | 1–3k |
| Prototype hardware (12–15 dev nodes + 1 gateway) | 3–6k |
| Tools: power profiler (Nordic PPK2 ~£100 / Joulescope ~£1k), oscilloscope, hot-air/reflow, enclosed printer if needed | 1.5–3k |
| PCB spins (3 × prototype runs of 10–20 boards, assembled) | 3–6k |
| Field pilot (8 nodes through one winter, travel, test shoot) | 3–8k |
| **Certification (outsourced test lab):** EMC, radio, GNSS receiver, safety, RF exposure, **RED cybersecurity (EN 18031)**, IP65 test | **12–25k** |
| Legal: terms, DPIA template, trademark; product liability insurance first year | 4–8k |
| Cloud / app MVP (in-house build, hosting) | 1–2k |
| Contingency (20 %) | 6–12k |
| **Total** | **≈ £35–75k** |

Grants worth checking (I'm not sure what's currently open): Innovate UK competitions, Defra Farming Innovation Programme, Forestry Commission woodland innovation funds, and Horizon Europe / EIC through UK association.

---

## 5. Commercial reality check

**Who pays, and why:**

| Segment | Pain | Budget source | Fit |
|---|---|---|---|
| Shooting estates, deer managers | Deer poaching, hare coursing, trespass, theft | Estate security budget | **Strong pain; wedge candidate** |
| Conservation trusts, BNG / nature-recovery sites | Biodiversity monitoring evidence | Grants, BNG monitoring obligations (30 years) | **Recurring money**, less urgency |
| Forestry companies | Illegal felling (rare in the UK), fly-tipping, fire | Operations | Weaker in the UK, stronger in parts of the EU |
| Private woodland owners | Trespass, peace of mind | Personal | Price-sensitive |

**Recommended positioning:** one kit, two budgets. Sell it as a **"security tripwire + biodiversity report"**: real-time alerts justify the purchase, and the monthly species report justifies the subscription. Acoustic alerts are most useful when they **cue a camera or a keeper**. Plan for integration with 4G trail cameras rather than competing with them.

**Kill criteria (Stage A, before custom hardware):**
1. Fewer than 6 of 20 target interviewees say they'd pay ≥ £3k up front + ≥ £40/month (or the hardware-as-a-service equivalent).
2. Field test: gunshot detection range < 500 m or localisation error > 50 m inside a 4-node array.
3. A prototype measured with a power profiler can't reach ≤ 0.6 Wh/day.

---

## 6. Can it be done in-house?

**Mostly yes.** It needs a core of 2 people with these skills, plus outsourced certification and PCB assembly.

| Workstream | In-house? | Skill needed | Outsource when |
|---|---|---|---|
| Customer discovery, product management | **Yes** | You | — |
| Enclosure CAD + FDM printing | **Yes** | OpenSCAD/Fusion, printing ASA/PETG | Injection moulding at ~1,000+ units/yr (tooling ~£8–20k) |
| Prototype on dev boards | **Yes** | Arduino/Zephyr C, soldering | — |
| Custom PCB design | **Yes, if** someone knows KiCad | Mixed-signal layout; use pre-certified radio/GNSS modules to avoid RF design | No KiCad skill → contractor, ~£3–8k per board revision |
| PCB fabrication + assembly | **No** | — | Always: JLCPCB/PCBWay (cheap) or a UK/EU EMS (e.g. Eurocircuits, Newbury Electronics) |
| Firmware | **Yes** | Embedded C, low-power, DMA, LoRaWAN | Contractor £400–600/day if needed |
| On-device ML | **Yes** | Python, TensorFlow/PyTorch → TFLite Micro; Edge Impulse speeds this up (**check its commercial licence terms**) | — |
| Wildlife ID (cloud) | **Yes** | Own model or a licensed one | Licence BirdNET commercially, or train on Xeno-canto data **after a licence audit** |
| Backend + app | **Yes** | ChirpStack (open-source LoRaWAN server), Postgres/PostGIS, a web map, push notifications; a progressive web app before native apps | Native iOS/Android apps later |
| TDOA solver | **Yes** | Numerical least squares (Python/numpy) | — |
| Certification testing | **No** | — | Accredited test lab. UKCA/CE is **self-declared** using its reports if harmonised standards are applied. |
| Legal / GDPR / insurance | Partly | — | Solicitor for terms + DPIA review; broker for product liability |
| Volume manufacturing (> 500/yr) | No | — | UK/EU contract manufacturer |

**Cost of contracting it all out instead:** about £100–150k extra (≈ 200–300 contractor days across hardware, firmware, ML and backend). In-house is the main reason the plan in Section 4 needs only ~£80–120k.

---

## 7. Recommended next steps (staged, each with a go/no-go)

| Stage | Duration | Cash | Deliverable | Go/no-go |
|---|---|---|---|---|
| **A. Validate** | 0–3 months | £3–6k | 20 customer interviews; 4 Pi-based recorders + 2 WisBlock low-power nodes in one wood; range, TDOA and power data | Kill criteria in Section 5 |
| **B. Pilot** | 3–9 months | £15–30k | Custom PCB v1, printed enclosure v1, 8-node + gateway pilot through winter on a partner estate, MVP app | ≥ 90 % uptime through winter, false positives < 1/day/site, ≥ 3 pilots convert to paid |
| **C. Productise** | 9–18 months | £20–40k | PCB v2, certification, first batch of 100, launch (e.g. at the Game Fair) | Pre-orders ≥ 15 sites |

## 8. Risks and open questions

**Top risks** (also in the [risk register](risk-register.md)):
- Willingness to pay is unproven
- Winter energy under conifer
- False alerts from legal shooting
- BirdNET licence
- RED cybersecurity requirements (EN 18031, mandatory for radio products since 1 Aug 2025)
- GDPR around voices
- Single-founder bandwidth

**Open questions for you:**
1. Who is in the team today, and which rows in Section 6 can you or they cover?
2. Can you fund ~£80–120k (savings, angel, grant), or should the plan aim to break even sooner with a hardware-as-a-service or survey-first model?
3. Do you have an estate partner for Stage A interviews and a pilot site?
4. Do you agree with dropping voice detection to "presence only, gates/car parks" and moving wildlife ID to the cloud?
