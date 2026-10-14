# SOP — Power & Cooling Health Check (Daily / Weekly)

> **Generic template — adapt to site policy before use.** This document is a personal
> portfolio exercise demonstrating data-center operations knowledge. It was not written
> for, and does not describe procedures used at, any employer.

| Field | Detail |
|---|---|
| Document ID | SOP-DCT-005 |
| Version | 1.0 |
| Last reviewed | 2026-10-14 |
| Owner (template) | Data Center Operations / Facilities |

## 1. Purpose

Define the routine facility checks that catch power and cooling problems before they
become outages: UPS loading, PDU phase balance, CRAC/CRAH temperatures against the
ASHRAE envelope, and standby-power readiness.

## 2. Scope

**In scope:** daily walkthrough checks and weekly deeper checks for the data hall's
electrical and mechanical support systems.

**Out of scope:** hands-on work inside UPS cabinets, switchgear, or generators
(licensed electrician / vendor only); BMS configuration changes.

## 3. Prerequisites

- [ ] Access to the BMS/DCIM dashboard and read-only credentials for UPS/PDU/CRAC interfaces
- [ ] IR thermometer or thermal camera for spot checks
- [ ] The site's escalation matrix and on-call numbers (posted, not just in a wiki)
- [ ] Know the EPO locations and the "do not touch" boundaries for this role

## 4. Reference — ASHRAE TC 9.9 thermal envelope

ASHRAE Thermal Guidelines for Data Processing Environments (Classes A1–A4).
The **recommended** envelope is where equipment should normally operate; the
**allowable** envelope is the wider survival range.

| Class | Recommended dry-bulb | Allowable dry-bulb | Notes |
|---|---|---|---|
| A1 (enterprise) | **18–27 °C (64.4–80.6 °F)** | 15–32 °C | Typical enterprise servers |
| A2 | 18–27 °C | 10–35 °C | — |
| A3 | 18–27 °C | 5–40 °C | — |
| A4 | 18–27 °C | 5–45 °C | — |

- Recommended humidity: 5.5 °C dew point to 60% RH (avoids both ESD and condensation risk).
- Measure at the **server air inlet** (front of rack), 1U–2U from the top of the rack and at mid-height — not at the CRAC return.
- Sustained operation above the recommended envelope is a finding, not an emergency;
  above the allowable envelope is an escalation.

## 5. Daily checks (walkthrough, ~20 minutes)

### 5.1 Electrical

- [ ] **UPS load %:** record per-UPS kW and % load. Healthy: < 80% of rated capacity
      (leaves headroom for a UPS module failure in N+1 designs).
- [ ] **UPS status:** no active alarms; battery `Float/Resting`, not `Discharging`;
      battery temperature within vendor spec.
- [ ] **PDU phase balance:** check L1/L2/L3 current on each rack PDU. Imbalance > 20%
      between phases is a finding — plan a load rebalance.
- [ ] **PDU displays:** expected voltage present on A and B feeds; no tripped breakers
      (flag handle in the tripped/mid position, not just off).
- [ ] **Abnormal sounds/smells:** buzzing breakers, hot-plastic odor — investigate immediately.

### 5.2 Cooling

- [ ] **CRAC/CRAH supply air temp:** within the site's setpoint ± 1 °C; note any unit in alarm or standby that should be running.
- [ ] **Return air temp:** compare against supply — a collapsing delta-T can indicate airflow bypass or a failed fan.
- [ ] **Hot-aisle spot checks:** IR thermometer on 3–4 rack inlets per walkthrough (rotate coverage daily);
      any inlet above 27 °C is logged as a finding.
- [ ] **Containment:** doors/curtains closed; blanking panels in place; no open floor tiles in the cold aisle.
- [ ] **Water/leak detection:** check leak-detection panel for alarms; visually check under
      CRAC units and any overhead piping for drips or staining.

### 5.3 General

- [ ] Log all readings in the daily check sheet / DCIM with timestamp and initials.
- [ ] Photograph anything abnormal (discolored breaker, water stain, alarm screen).

## 6. Weekly checks (deeper, ~60 minutes)

- [ ] **UPS battery:** visual inspection (no swelling, leaking, or corrosion on terminals);
      confirm the last battery test date and next scheduled test.
- [ ] **Generator:** confirm `Auto` mode, fuel level > site minimum (commonly 72-hour runtime),
      no alarms on the controller, last test-run date within policy.
- [ ] **ATS (automatic transfer switch):** normal source indicated; last transfer test date logged.
- [ ] **PDU trending:** review the week's load trend — any PDU climbing toward 80% gets a
      capacity-planning flag before it becomes urgent.
- [ ] **CRAC/CRAH:** check filter status / differential pressure; condensate drains clear;
      review runtime hours for lead/lag rotation.
- [ ] **Thermal imaging sweep:** one full-aisle IR pass weekly; compare hotspots against baseline.
- [ ] **E-stop/EPO:** visual check only (covers intact, signage legible) — never test without a planned MOP.
- [ ] Review the week's daily sheets for trends a single day wouldn't show.

## 7. Escalation thresholds

| Reading | Level | Action |
|---|---|---|
| Rack inlet temp 27–30 °C sustained > 30 min | **Warning** | Log, increase monitoring, notify facilities lead |
| Rack inlet temp > 32 °C (A1 allowable limit) or rising fast | **Critical** | Escalate to facilities + operations lead immediately; consider load reduction |
| UPS load > 80% | **Warning** | Capacity review; no new loads on that UPS without approval |
| UPS on battery / discharging unexpectedly | **Critical** | Escalate immediately; prepare for controlled load shed per site plan |
| PDU phase imbalance > 20% | **Warning** | Schedule a rebalance within the week |
| Tripped breaker | **Warning** | Do NOT reset blindly — investigate the cause first (overload? fault?); reset once, and if it trips again, escalate to electrician |
| Leak-detection alarm or visible water | **Critical** | Escalate immediately; protect equipment (power down only on explicit instruction) |
| Generator not in Auto / fuel below minimum | **Critical** | Escalate to facilities lead immediately |
| Burning smell, arcing, smoke | **Emergency** | EPO only if trained and authorized per site policy; evacuate; call emergency services |

**Golden rule:** a tripped breaker is a symptom, not the problem. Find the cause before re-energizing.

## 8. Safety

- Read-only checks only: no opening energized panels, no resetting breakers without investigation,
  no touching switchgear.
- Maintain arc-flash boundaries and posted clearances around electrical gear.
- Report, don't repair, anything inside the UPS/generator/switchgear envelope.

## 9. Rollback

Not applicable in the usual sense — this SOP is observational. If a check itself causes
an issue (e.g., a PDU display button press changes a setting), immediately notify the
facilities lead and document exactly what was pressed.

## 10. Sign-off (weekly sheet)

| Week of | Daily sheets complete (7/7) | Findings open | Findings closed | Technician | Facilities lead |
|---|---|---|---|---|---|
| | ☐ | | | | |

Open findings carried forward: _______________________________________________
