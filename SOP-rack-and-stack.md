# SOP — Server Rack-and-Stack (Receiving to Production Power-On)

> **Generic template — adapt to site policy before use.** This document is a personal
> portfolio exercise demonstrating data-center operations knowledge. It was not written
> for, and does not describe procedures used at, any employer.

| Field | Detail |
|---|---|
| Document ID | SOP-DCT-001 |
| Version | 1.0 |
| Last reviewed | 2026-10-14 |
| Owner (template) | Data Center Operations |
| Classification | Generic / public-safe — no site-specific credentials or secrets |

## 1. Purpose

Define the repeatable, audit-ready procedure for taking new server hardware from the
loading dock to a powered-on, verified state inside the rack. Applies to 1U/2U/4U
rackmount servers and network appliances in standard 19″ four-post racks.

## 2. Scope

**In scope:** receiving inspection, unboxing, rail-kit installation, rack insertion,
torque/labeling standards, power-on sequence, post-install verification, sign-off.

**Out of scope:** OS installation (see build standards), structured cabling (see
SOP-DCT-002), firmware upgrades beyond what is needed for power-on.

## 3. Prerequisites

- [ ] Approved install work order / change record number: __________
- [ ] Rack location assigned: Facility ____ Row ____ Rack ____ U positions ____
- [ ] Rack capacity check passed (U space, power headroom, cooling headroom)
- [ ] Server lift or two-person lift plan for equipment > 25 kg (50 lbs)
- [ ] Anti-static wrist strap, ESD mat
- [ ] Cage nuts + M6 screws (or OEM toolless rails), torque driver (10–12 in-lb / 1.1–1.4 N·m)
- [ ] Label printer with self-laminating wire/asset labels
- [ ] Camera/phone for inspection photos
- [ ] PDU receptacle types verified (C13/C19) against server PSU plugs
- [ ] EPO (Emergency Power Off) location identified on this aisle

## 4. Procedure

### 4.1 Receiving & unboxing inspection

1. [ ] Compare the shipment against the packing list / purchase order: model, quantity, serial numbers.
2. [ ] Inspect cartons for damage (crushed corners, punctures, water stains, tilted/shock indicators tripped). Photograph any damage **before** opening.
3. [ ] If damage is found, tag the unit **DO NOT RACK — DAMAGED**, notify procurement/logistics, and open an RMA ticket. Do not proceed with damaged units.
4. [ ] Open on the loading dock or staging area — never block a hot/cold aisle.
5. [ ] Inventory the box contents: server chassis, rail kit, bezel, power cords, documentation, any loose accessories.
6. [ ] Record serial number, service tag, and asset tag of each unit on the work order.
7. [ ] Remove all foam, plastic, and desiccant. Verify all internal blanks and air baffles are seated (missing baffles disrupt designed airflow).
8. [ ] Visually inspect inside the chassis: cables seated, DIMMs latched, heatsinks secure, no loose screws.

### 4.2 Rail-kit installation

1. [ ] Confirm rail kit matches the rack depth and the server model (OEM rails are not interchangeable across vendors).
2. [ ] Identify front (square-hole post with release latch) and rear of each rail.
3. [ ] For square-hole racks: press the rail pins into the selected U positions. Count **U positions from the bottom** — mark with tape: top U ___, bottom U ___.
4. [ ] For threaded-hole racks: install cage nuts at the correct U positions (use a cage-nut tool, not a screwdriver that can slip).
5. [ ] Level check: both rails at the identical U on left and right, front and rear.
6. [ ] Torque rail screws to OEM spec, typically **10–12 in-lb (1.1–1.4 N·m)**. Do not over-torque — stripped cage nuts are a rack hazard.

### 4.3 Racking the server

1. [ ] Two-person lift (or server lift) for anything over 25 kg. Lift with legs, keep the chassis level.
2. [ ] Extend the middle rail sections fully, align the chassis bearing with the rail channel, and slide in slowly until the locks engage — listen/feel for the click on both sides.
3. [ ] Verify the chassis slides in and out smoothly without binding.
4. [ ] Secure the front thumbscrews / rack ears.
5. [ ] Apply labels (see §5).

### 4.4 Cabling (power + management)

1. [ ] Connect PSU 1 to PDU-A and PSU 2 to PDU-B (A/B feed redundancy). Never both PSUs to the same PDU.
2. [ ] Use the correct cord: C13→C14 for standard PSUs, C19→C20 for high-wattage PSUs.
3. [ ] Strain-relieve cords — no tension on the PSU inlet; leave a service loop.
4. [ ] Connect the BMC/iDRAC/iLO management port to the management switch per the port-assignment sheet.
5. [ ] Cable data ports per SOP-DCT-002 (structured cabling) — or leave labeled and capped if cabling is a separate work order.

### 4.5 Power-on sequence

1. [ ] Confirm both PDUs show the expected phase/voltage on their displays.
2. [ ] Verify the server's PSU LEDs: solid green (AC present, standby OK).
3. [ ] Power on via the front power button **or** via BMC virtual power — never yank/replug cords to power-cycle.
4. [ ] Watch the front-panel health LED through POST: it should settle to solid green/blue (vendor-dependent) with no amber/red fault LEDs.
5. [ ] Note the time-to-BMC-ready: BMC should be pingable within ~3–5 minutes of AC power.

### 4.6 Post-install verification

1. [ ] BMC/iDRAC/iLO reachable at its assigned IP; default credentials changed or handed to the credential owner.
2. [ ] All expected DIMMs, CPUs, drives, and PSUs visible in the BMC hardware inventory — compare against the order spec.
3. [ ] No critical/warning entries in the BMC system event log (SEL) from the install.
4. [ ] Front bezel installed and locked (if site policy requires).
5. [ ] Rack blanking panels installed in all empty U above/below (hot/cold aisle containment).
6. [ ] Work order updated with final U positions, serials, asset tags, BMC IP.
7. [ ] Photos of the finished rack front and rear attached to the ticket.
8. [ ] Packaging removed from the data hall; cardboard goes to recycling — **no cardboard storage inside the data hall** (fire/pest policy).

## 5. Labeling standard

| Item | Label content | Placement |
|---|---|---|
| Server front | Hostname + asset tag | Bezel or left rack ear |
| Server rear | Hostname + asset tag | Right rear, visible with cords connected |
| Power cords | `PSU1 → PDU-A-<outlet>` / `PSU2 → PDU-B-<outlet>` | Both ends, 6″ from connector |
| Management port | `MGMT <hostname> → <switch>:<port>` | Cable end at server |

Example: `srv-dfw-0421 · AST-88412`

## 6. Safety

- **ESD:** wrist strap on before touching internal components; work on the ESD mat in staging.
- **Lifting:** use the server lift above 25 kg; keep aisles clear; never rack alone on a ladder.
- **Electrical:** inspect cords for damage before connecting; confirm PDU breakers are on and correctly rated; know the EPO location.
- **Thermal:** never leave a chassis open and powered in the rack; reinstall top cover before racking.

## 7. Rollback

| Trigger | Action |
|---|---|
| Damaged on arrival | Tag DO NOT RACK, photo, RMA — revert rack U to empty, update work order |
| Wrong model shipped | Do not rack; return to staging, notify procurement |
| PSU fault LED after power-on | Power down, reseat PSUs, try known-good cord; if still faulted, open vendor support case — do not force it into production |
| BMC unreachable | Reseat management cable, confirm VLAN/port assignment, retry; escalate to network team if the switch port is the issue |

## 8. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Technician (install) | | | |
| Peer verifier | | | |
| Operations lead (acceptance) | | | |

**Result:** ☐ PASS — server racked, powered, verified ☐ FAIL — see rollback notes

Notes: _______________________________________________________________
