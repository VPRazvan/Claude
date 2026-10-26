# Risk register

Likelihood and impact use a 1–5 scale. Score = L × I.

| ID | Risk | L | I | Score | Mitigation | Owner | Phase |
|---|---|---|---|---|---|---|---|
| R-001 | Winter energy harvest under canopy too low for continuous operation | 4 | 5 | 20 | Duty-cycled detection, remote panel option, measured power budget (Phase 1) | — | 1 |
| R-002 | Commercial ML licence conflict (e.g. BirdNET CC BY-NC-SA) | 4 | 4 | 16 | Licence audit; own-trained models or commercial licence | — | 4 |
| R-003 | GDPR exposure from voice capture near public footpaths | 3 | 4 | 12 | Voice-presence-only detection, DPIA template, signage guidance | — | 4/7 |
| R-004 | IP65 + acoustic transparency conflict in FDM enclosure | 3 | 4 | 12 | ePTFE acoustic vents, gasketed design, early spray/immersion tests | — | 3/6 |
| R-005 | False-positive alerts erode customer trust | 4 | 4 | 16 | Multi-stage pipeline with cross-node confirmation; measured FP/day target | — | 4/6 |
| R-006 | Over-claiming firearm/poacher detection in marketing | 3 | 4 | 12 | Reviewed claim wording; documented performance limits | — | 7 |
| R-007 | Willingness to pay unproven (target £4.8k kit + £60/month) | 3 | 5 | 15 | 20 customer interviews with kill criteria before custom PCB | — | 0b |
| R-008 | Legal shooting (shoot days, stalking, foxing) generates unwanted gunshot alerts | 5 | 3 | 15 | Authorised-activity calendar and mute zones in app; alert = "impulsive event", not "poacher" | — | 4/5 |
| R-009 | RED cybersecurity requirements (EN 18031) add cost and design work | 4 | 3 | 12 | Secure boot, encrypted links, no default passwords from day one; budget in certification | — | 4/7 |
| R-010 | Continuous raw-audio storage is infeasible on power and SD capacity | 5 | 2 | 10 | Rolling buffer + event clips + scheduled survey windows | — | 1/2 |
| R-011 | Mesh relaying keeps radios listening (~15 mW), breaking the power budget | 4 | 3 | 12 | LoRaWAN star to a mast-mounted gateway; relaying only where unavoidable | — | 1 |
