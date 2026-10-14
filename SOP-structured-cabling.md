# SOP — Structured Cabling (Copper & Fiber)

> **Generic template — adapt to site policy before use.** This document is a personal
> portfolio exercise demonstrating data-center operations knowledge. It was not written
> for, and does not describe procedures used at, any employer.

| Field | Detail |
|---|---|
| Document ID | SOP-DCT-002 |
| Version | 1.0 |
| Last reviewed | 2026-10-14 |
| Owner (template) | Data Center Operations |

## 1. Purpose

Define standards for installing, labeling, testing, and documenting copper (Cat6/Cat6A)
and fiber-optic cabling in the data hall so every link is identifiable, testable, and
serviceable by any technician.

## 2. Scope

**In scope:** intra-rack and inter-rack copper/fiber runs, patch-panel terminations,
labeling (TIA-606-C-inspired), handling rules, testing, cleanup.

**Out of scope:** outside-plant / carrier fiber, MPO trunk engineering (refer to the
fiber plant design doc), active equipment configuration.

## 3. Prerequisites

- [ ] Approved cabling work order with the port-assignment sheet attached
- [ ] Cable type confirmed: Cat6A for 10GBase-T, OM4/OM5 or OS2 fiber per the design
- [ ] Pre-terminated or field-terminated? Field terminations require a certified installer on shift
- [ ] Tools: cable certifier or qualification tester, fiber cleaning kit (click-cleaners,
      wipes, inspection scope), VFL (visual fault locator), OTDR (for new fiber trunks),
      label printer, velcro wraps (no zip ties on data cables), cable combs
- [ ] Dust caps/plugs for all unused fiber ports and patch-panel positions
- [ ] Fiber and copper kept on **separate pathways** — never lash fiber tightly to copper bundles

## 4. Handling rules

### 4.1 Copper (Cat6 / Cat6A)

- [ ] **Bend radius:** ≥ 4× cable diameter (≈ 1″ / 25 mm for typical Cat6A). No kinks, no 90° hard bends.
- [ ] **Pull tension:** ≤ 25 lbf (110 N). If it fights, stop and re-route — do not force.
- [ ] **Untwist:** no more than 0.5″ (13 mm) of pair untwist at a termination; keep the jacket up to the punch-down point.
- [ ] **Separation:** maintain ≥ 2″ from power cables; cross power at 90° where unavoidable.
- [ ] **Bundling:** velcro only, snug but not compressing — over-tightened bundles deform pairs and fail alien-crosstalk tests. Max ~24 cables per velcro bundle in horizontal runs.
- [ ] **Length:** permanent link ≤ 90 m; channel (with patch cords) ≤ 100 m. Verify before pulling long runs.
- [ ] **No splices.** A damaged copper run is re-pulled, not spliced.

### 4.2 Fiber

- [ ] **Bend radius:** ≥ 10× cable diameter under load, ≥ 15× long-term for standard fiber;
      follow the manufacturer spec for bend-insensitive fiber (still never crease).
- [ ] **Pull tension:** per manufacturer spec (typically ≤ 100 lbf for indoor distribution); use pulling eyes, never pull by the jacket on pre-terminated assemblies.
- [ ] **End-face hygiene:** inspect → clean → inspect every connector before mating. A dirty connector is the #1 cause of "mystery" link failures.
- [ ] **Dust caps:** on every unmated connector and adapter, immediately — caps go back on the moment a cord is unplugged.
- [ ] **Never look into a fiber end** while the far end may be live. Treat all fiber as potentially carrying laser light; use the inspection scope, not your eye.
- [ ] **Polarity:** verify TX→RX polarity on every new fiber link before calling it good (Method B for duplex; check MPO polarity method per the trunk design).
- [ ] **Slack:** leave a service loop (12–18″) at each end, coiled at ≥ minimum bend radius in the slack manager — never a tight coil tie-wrapped to the rack post.

## 5. Labeling standard (TIA-606-C-inspired)

Use a hierarchical identifier that answers **where each end lives** without a lookup:

```
<facility>-<hall>-<row>-<rack>-<panel>-<port>
```

**Example label format** (printed on both ends of every cable):

```
DFW1-A-R03-RK12-PP02-17  →  DFW1-A-R03-RK14-PP01-03
CU-6A-BLU-015M   SN:88412-0317
```

Field key:

| Field | Meaning | Example |
|---|---|---|
| facility | Site code | `DFW1` |
| hall | Data hall | `A` |
| row / rack | Row and rack number | `R03` / `RK12` |
| panel | Patch panel ID | `PP02` |
| port | Panel port | `17` |
| media | Media + color + length | `CU-6A-BLU-015M` or `FO-OM4-AQU-010M` |

**Labeling rules:**

- [ ] Every cable labeled at **both ends**, within 6″ (15 cm) of the connector.
- [ ] Patch-panel ports labeled on the panel face; switch ports labeled per the port-assignment sheet.
- [ ] Labels face outward and are readable without moving adjacent cables.
- [ ] Color convention (suggested — confirm site standard):
  - Blue — copper data / server access
  - Yellow — copper management / OOB
  - Aqua — OM3/OM4 multimode fiber
  - Yellow (fiber jacket) — OS2 singlemode fiber
  - Red — console / serial
- [ ] Update the cable database / DCIM within the same shift — an unlabeled or undocumented cable is a failed install.

## 6. Procedure — new copper link

1. [ ] Confirm both endpoints on the port-assignment sheet.
2. [ ] Pull/route the cable through designated pathways (tray, ladder rack, or vertical managers) — never drape across the top of racks or through another rack's space.
3. [ ] Dress into vertical and horizontal managers; velcro every 12–18″.
4. [ ] Terminate (field-terminated) or plug (pre-terminated); keep pair untwist ≤ 0.5″.
5. [ ] Label both ends per §5.
6. [ ] Test per §8 and record results against the port-assignment sheet.

## 7. Procedure — new fiber link

1. [ ] Confirm endpoints, fiber type (OM4 vs OS2 — **never mix**), and polarity method.
2. [ ] Inspect and clean both ends; verify with the scope (no scratches, pits, or contamination in the core zone).
3. [ ] Route through fiber troughs / managers with proper bend radius; secure with velcro.
4. [ ] Mate connectors; confirm the latch clicks.
5. [ ] Verify link light at both ends and correct TX→RX polarity (swap one pair if reversed on duplex).
6. [ ] Label both ends per §5; cap any unused positions.
7. [ ] Test per §8 and record results.

## 8. Testing

### 8.1 Copper

| Test | Tool | Pass criteria |
|---|---|---|
| Wire map / continuity | Certifier or basic tester | 1:1 pinout, no opens, shorts, splits, or miswires |
| Length | Certifier | ≤ 90 m permanent link |
| Certification (new installs) | Cable certifier (e.g., DSX-class) | Pass to Cat6A channel/permanent-link limits; save the test file |
| PoE load (if applicable) | PoE tester | Negotiated class matches design |

- [ ] Save certification files named `<cable-ID>_<date>.pdf` and attach to the work order.
- [ ] Any link that fails certification is re-terminated once; if it still fails, the run is re-pulled.

### 8.2 Fiber

| Test | Tool | Pass criteria |
|---|---|---|
| Continuity + polarity | VFL + light check | Light visible at far end; TX→RX correct |
| Insertion loss (new trunks) | Light source + power meter (OLTS) | Within the loss budget (design-dependent; typical ≤ 0.75 dB per mated pair + fiber attenuation) |
| OTDR trace (new trunks / troubleshooting) | OTDR | No reflective events at connectors beyond spec; no macrobends (compare both directions — bidirectional averaging) |
| End-face inspection | Fiber scope | Pass per IEC 61300-3-35 grading |

- [ ] Store OTDR traces and OLTS results with the cable ID in the fiber plant records.

## 9. Cleanup

- [ ] All cutoffs, jacket scraps, and **fiber shards** collected — fiber shards go in a sealed sharps container, never the open trash.
- [ ] Cardboard and packaging removed from the data hall.
- [ ] Horizontal and vertical managers closed; no cables pinched in doors.
- [ ] Work area photographed; before/after photos attached to the ticket.
- [ ] Tools accounted for (nothing left in the rack or on top of cabinets).

## 10. Safety

- Fiber shards are glass — wear safety glasses when handling bare fiber; no food/drink in the work area during fiber work.
- Laser safety: never view a fiber end directly; assume live.
- Ladder safety for overhead tray work: three points of contact, tools tethered.
- Cut-resistant gloves when pulling through tray with sharp edges.

## 11. Rollback

| Trigger | Action |
|---|---|
| Link fails certification twice | Re-pull the run; quarantine the failed cable for QA review |
| Wrong media pulled (OM4 vs OS2) | Remove and re-pull correct media; update the port-assignment sheet |
| Polarity wrong on fiber | Re-terminate or swap pairs per the approved polarity method; re-test |
| Labeling error found in audit | Relabel both ends and correct DCIM the same shift |

## 12. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Technician (install/test) | | | |
| Peer verifier (spot-check) | | | |
| Operations lead (acceptance) | | | |

**Result:** ☐ PASS — links installed, labeled, tested, documented ☐ FAIL — see notes

Links installed: ______  Tested: ______  Certified: ______  DCIM updated: ☐
