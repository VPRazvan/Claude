# Option: home 3D printers + Raspberry Pi with add-ons

**Question:** Can we build this with home 3D printers and Raspberry Pi boards plus off-the-shelf add-on boards (HATs) instead of custom hardware?

**Short answer:**

| | Prototype / pilot | Product sold to customers |
|---|---|---|
| **Home 3D printer** | **Yes** | **Yes, up to ~300–500 units/yr**, with an enclosed printer, ASA filament and a per-unit leak test |
| **Raspberry Pi as the gateway** | **Yes** | **Yes** for the pilot. For sale, buy a certified gateway or put the Pi inside one (certification, Section 4). |
| **Raspberry Pi as the field-test recorder** | **Yes, the fastest way to get real data** | No: needs a large battery, not solar-viable |
| **Raspberry Pi as the always-on solar node** | Not under canopy | **No**: it uses 40–60× the node's winter energy budget (Section 2) |

Prices are approximate (from my own knowledge, 2026, not live quotes). Check UK stockists: The Pi Hut, Pimoroni, Farnell, Mouser, RS.

---

## 1. Home 3D printers

### 1.1 What works

| Printer class | Examples | Approx. price | Materials | Verdict |
|---|---|---|---|---|
| **Enclosed, fast** | Bambu Lab P1S / X1C class, Prusa Core One | £550–1,200 | ASA, ABS, PETG, TPU | **Recommended.** You need an enclosure for ASA, which is the UV-stable outdoor material. |
| Open frame | Prusa MK4/A1, Creality Ender 3 class | £200–800 | PETG, PLA, TPU | Prototypes only. PETG yellows and gets brittle after ~1–2 UK summers in sun unless painted. |

**Material choice:**
- **ASA** for the shell (UV stable; usable from about −20 to +90 °C).
- **TPU 95A** for printed gaskets and bumpers.
- **PETG** acceptable for internal brackets.
- ASA gives off styrene fumes while printing. Print in a ventilated room or a garage, or use a printer with a carbon filter.

### 1.2 Can a home print reach IP65?

Yes, but not straight off the printer. FDM layers leak along the layer lines. The fixes are standard:
- ≥ 4 perimeters (walls at least 2 mm), 100 % infill at the gasket faces.
- Seal on a **TPU or EPDM-cord gasket in a printed groove**, never directly on the layer lines.
- Brass heat-set inserts and stainless screws for the lid.
- **ePTFE acoustic vents** (e.g. Gore or Donaldson adhesive vents) over the mic ports. They pass sound and block water.
- A breather vent to stop condensation pumping, plus a desiccant pack.
- Conformal coating on the PCB as a second line of defence.
- **Leak-test every unit:** a 10-minute vacuum or immersion test at ~£0 cost with a jar and a hand vacuum pump. Phase 3 details it.

### 1.3 Throughput and cost

| | Value |
|---|---|
| Material per node (shell, lid, rain hood, gasket) | ~250–300 g ASA + 20 g TPU ≈ **£6–9** |
| Print time per node | ~8–12 h on a fast enclosed printer |
| Nodes per printer per week | ~12–18 |
| 100 nodes | **1 printer ≈ 6–8 weeks; 3 printers ≈ 2–3 weeks** |
| Switch to injection moulding | ~500–1,000 units/yr (tooling ~£8–20k) — Phase 7 |

**Verdict:** a 2–3 printer home "farm" (~£2–3k) is a sound way to make the pilot and the first year's 100–300 units.

---

## 2. Raspberry Pi: where it fits and where it doesn't

### 2.1 Power: the deciding number

| Configuration | Average power | Per day | Panel needed (Dec, southern England, open sky) | Panel needed under leafless broadleaf | Battery for 14 days |
|---|---|---|---|---|---|
| Pi Zero 2 W always on + mics + GPS | ~0.8 W | ~19 Wh | 19 ÷ (1.0 PSH × 0.7) ≈ **27 W** | ≈ **90 W** | ≈ 270 Wh (~£120–200) |
| Pi 4 always on | ~2.5–3 W | ~65 Wh | ~95 W | ~300 W | ~900 Wh |
| Pi 5 always on | ~3–4 W | ~80 Wh | ~115 W | ~380 W | ~1.1 kWh |
| **Low-power microcontroller node** (feasibility study) | ~15–20 mW | **~0.4 Wh** | **~0.6 W** | **~2 W** | ~6 Wh |

**Conclusion:** an always-on Pi node needs a 30–90 W panel and a car-battery-sized pack per tree. That's not discreet, not cheap and not tree-mountable. **Use a microcontroller for the solar node.** In Scotland the Pi numbers double.

### 2.2 The hybrid that *does* work: microcontroller "sentinel" + Pi on demand

- A low-power microcontroller (e.g. Adafruit Feather nRF52840 Sense, which has an on-board PDM mic) listens continuously.
- It keeps a few seconds of audio in RAM and timestamps sound onsets against GPS PPS.
- It only powers up the Pi through a load switch when something loud happens.

| | Value |
|---|---|
| Sentinel | ~15 mW → 0.36 Wh/day |
| Pi Zero 2 W per event (boot + classify + shut down) | ~1 W × 45 s ≈ 12.5 mWh |
| 20 events/day | 0.25 Wh |
| **Total** | **≈ 0.6 Wh/day: solar-viable** |

**Catch:**
- The Pi takes ~15–25 s to boot, so the sentinel has to do the time-critical work: onset timestamp, TDOA and the first alert. The Pi only adds a better classifier and audio handling.
- Repeated power cycling risks SD-card corruption, so it needs a read-only root filesystem.
- Pi Zero 2 W cold-weather behaviour should be tested at −10 °C.

**Verdict:** this is a useful stepping stone if your team is stronger in Python/Linux than embedded C. A single larger microcontroller (Phase 2) ends up simpler for the product.

### 2.3 Pi as field recorder (Stage A): recommended

The goal is real data from a real wood on detection range, TDOA accuracy and false-trigger sources **before** designing hardware. For this, power doesn't matter: a 12 V battery lasts about a week.

**Per recorder node:**

| Item | Example | Approx. £ |
|---|---|---|
| Computer | Raspberry Pi Zero 2 W (or Pi 4 2 GB if you want 4+ USB/I²S channels) | 15 (45) |
| Microphones | 2 × I²S MEMS breakouts (Adafruit ICS-43434 or SPH0645) in stereo, **or** a ReSpeaker 4-mic HAT (check availability) | 12–40 |
| GPS with **PPS output** | Adafruit Ultimate GPS HAT, or a u-blox NEO-M9N breakout with PPS | 20–40 |
| Storage | 128 GB high-endurance microSD | 15 |
| Power | 12 V 20 Ah LiFePO4 + 12→5 V buck converter (≈ 256 Wh → ~8 days at 1.3 W) | 70–100 |
| Housing | Cheap IP65 junction box + printed mic hood with ePTFE vent | 10–15 |
| **Total** | | **≈ £140–215** |

**Time sync, the important trick:**
- Running `gpsd` + `chrony` with PPS disciplines the Pi **clock** to ~1 µs.
- But the **audio** clock isn't locked to it, and ALSA buffering adds milliseconds of jitter. That alone makes TDOA useless.
- **Fix:** feed the GPS PPS pulse (through a resistor divider) into a spare audio input channel. Every recording then carries a sample-accurate second marker. Alignment and sample-rate drift correction become simple post-processing.
- This is how many research TDOA arrays are built.

**Software (all Python):**

| Tool | Licence | Note |
|---|---|---|
| `arecord` / `sox` | Open source | Recording |
| `gpsd` + `chrony` | Open source | Timing |
| OpenSoundscape | MIT | Detection experiments |
| BirdNET-Pi | Non-commercial | **For research use only.** Fine for Stage A data, not for the product (licence). |

**Stage A kit:** 4 recorders (~£600–850) placed 300–500 m apart, plus test shots on a licensed shoot. That answers the three kill criteria from the [feasibility study](feasibility-business-hardware.md) for under £1k.

### 2.4 Pi as the gateway: recommended for the pilot

| Item | Example | Approx. £ |
|---|---|---|
| Computer | Raspberry Pi 4 (2–4 GB) or Compute Module 4/5 | 45–70 |
| LoRaWAN concentrator | RAK2287 or RAK5146 (SX1302/SX1303, 868 MHz) + Pi HAT | 110–160 |
| 4G | Teltonika RUT241 router (robust, UK/EU certified), or a SIM7600E HAT | 60–130 |
| Antenna | 868 MHz fibreglass mast antenna + LMR-400 cable | 40–80 |
| Power | 50 W panel + 40–50 Ah LiFePO4 + MPPT controller (gateway ≈ 3–4 W → ~85 Wh/day) | 150–250 |
| Enclosure | IP66 ABS/polycarbonate box + glands (not printed: it lives on a mast) | 25–40 |
| Software | ChirpStack (open-source LoRaWAN network server, MIT) | 0 |
| **Total** | | **≈ £430–730** |

**Same job off the shelf:** a RAKwireless WisGate Edge outdoor gateway with LTE, ~£350–550 plus solar. It's already certified. Phase 2 makes the final call.

---

## 3. Recommended path

| Stage | Hardware | Printer use |
|---|---|---|
| **A: Validate** (now) | 4 × Pi Zero 2 W recorders with GPS PPS injected into the audio, on 12 V batteries; 1 × Feather nRF52840 Sense to measure the real low-power listening budget | Mic hoods and brackets in PETG; junction boxes bought |
| **B: Pilot** | Low-power microcontroller nodes (WisBlock or custom PCB v1) + **Pi-based LoRaWAN/4G gateway** | Full ASA enclosures from 1–2 enclosed printers, leak-tested |
| **C: Product** | Custom PCB nodes + certified gateway (bought, or a Pi CM-based one taken through certification) | 3-printer farm up to ~300–500/yr, then injection moulding |

## 4. Risks and open questions

**Risks:**
- **Certification:** a Pi + HAT combination is a new radio product and needs its own RED/EMC testing before sale (the Pi's own compliance doesn't carry over). Fine for pilots on your own or partner land; budget it for sale.
- **SD-card wear and corruption** in Pi recorders. Use high-endurance cards and a read-only root filesystem where possible.
- **Mic availability:** Seeed ReSpeaker Pi HATs have had patchy availability. Confirm stock before planning on them; I²S breakouts are the safe fallback.
- **Print consistency at volume:** every printed unit must pass a leak test. Print settings and filament batch go in the build record.

**Open questions:**
1. Which printer(s) do you have now? Is it enclosed?
2. Is the team more comfortable in Python/Linux (Pi) or embedded C (microcontroller)? That decides whether the sentinel + Pi hybrid is worth it as a stepping stone.
3. Shall I write the Stage A recorder build guide next: wiring, the PPS-into-audio circuit, chrony config, recording script and a printed mic hood?
