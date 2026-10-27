# Decision log

| ID | Date | Phase | Decision | Recommendation | Runner-up | Why | Status |
|---|---|---|---|---|---|---|---|
| D-000 | 2026-10-26 | 0 | Project structure | Single repo, one folder per discipline | Separate repos per discipline | Small team; cross-cutting docs easier to keep consistent | Accepted |
| D-001 | 2026-10-27 | 0c | Solar node compute | Low-power microcontroller (nRF52840/nRF5340/STM32U5 class) | Microcontroller sentinel + Pi Zero 2 W on demand | An always-on Pi needs 19+ Wh/day versus a ~0.4 Wh/day budget | Proposed |
| D-002 | 2026-10-27 | 0c | Stage A field recorder | Pi Zero 2 W + I²S mics + GPS PPS injected into an audio channel, 12 V battery | AudioMoth + separate GPS logger | Fastest path to range/TDOA data; Python; sample-accurate sync | Proposed |
| D-003 | 2026-10-27 | 0c | Pilot gateway | Pi 4/CM + RAK2287/5146 + 4G router + ChirpStack | RAK WisGate Edge outdoor (certified) | Cheap and flexible for the pilot; switch to certified for sale | Proposed |
| D-004 | 2026-10-27 | 0c | Enclosure manufacture to ~300–500/yr | Home print farm: 2–3 enclosed printers, ASA + TPU gaskets, per-unit leak test | Outsourced print bureau (e.g. SLS PA12) | Low capex, fast iteration; ASA needs an enclosure | Proposed |
