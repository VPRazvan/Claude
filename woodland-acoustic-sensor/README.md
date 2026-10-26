# Woodland Acoustic Sensor Network (working name)

Solar-powered acoustic sensor nodes for UK/EU woodland and estates. A network of nodes:

1. Detects threats on-device in real time — gunshots, chainsaws, vehicles, human voices
2. Identifies wildlife by sound — birds, deer, foxes (bats optional)
3. Locates sound sources by TDOA triangulation, using GPS-disciplined time sync
4. Sends alerts with coordinates to a phone app / dashboard within 60 s
5. Stores raw audio locally for evidence and surveys

**Customers:** private woodland owners, shooting estates, deer managers, conservation trusts, forestry companies. Later: alert API into a fieldsports SaaS.

## Phase tracker

| Phase | Scope | Status | Output |
|---|---|---|---|
| 0 | Clarifying questions | **Waiting on answers** | [docs/phase-0-questions.md](docs/phase-0-questions.md) |
| 0b | Feasibility: business, financials, in-house capability | Draft | [docs/feasibility-business-hardware.md](docs/feasibility-business-hardware.md) |
| 1 | System architecture, data flow, power budget, TDOA maths | Not started | `docs/phase-1-architecture.md` |
| 2 | Hardware selection, BOM, PCB vs dev board | Not started | `hardware/` |
| 3 | Enclosure design + OpenSCAD v1 | Not started | `enclosure/` |
| 4 | Firmware architecture, ML pipeline, OTA, security | Not started | `firmware/`, `ml/` |
| 5 | Backend, app, triangulation solver | Not started | `backend/` |
| 6 | Bench and field test plan, metrics, safe test sounds | Not started | `testing/` |
| 7 | Certification, liability, pricing, competitors, GTM | Not started | `commercial/` |

Each phase ends with risks and open questions, and waits for go-ahead before the next one starts.

## Repository layout

```
docs/         Phase write-ups, decision log, risk register
hardware/     Schematics, BOM, PCB (KiCad)
enclosure/    OpenSCAD / CAD sources, print profiles
firmware/     Node and gateway firmware
ml/           Datasets manifest (not data), training scripts, model cards, licences
backend/      Cloud services, solver, app
testing/      Bench and field test procedures, results
commercial/   Certification, pricing, competitor analysis
```

## Key documents

- [Decision log](docs/decision-log.md) — every key decision with recommendation, runner-up and reasoning
- [Risk register](docs/risk-register.md) — live list of technical, legal and commercial risks
