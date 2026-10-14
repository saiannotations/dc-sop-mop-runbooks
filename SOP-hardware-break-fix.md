# SOP — Hardware Break-Fix (Drive / PSU / Memory / CPU)

> **Generic template — adapt to site policy before use.** This document is a personal
> portfolio exercise demonstrating data-center operations knowledge. It was not written
> for, and does not describe procedures used at, any employer.

| Field | Detail |
|---|---|
| Document ID | SOP-DCT-003 |
| Version | 1.0 |
| Last reviewed | 2026-10-14 |
| Owner (template) | Data Center Operations |

## 1. Purpose

Provide a consistent fault-isolation and repair workflow for the four most common
server hardware failures — drives, power supplies, memory, and CPUs — minimizing
mean time to repair (MTTR) while protecting data and warranty status.

## 2. Scope

**In scope:** 1U/2U rackmount servers under vendor warranty or depot contract; field-replaceable
units (FRUs) rated for technician replacement.

**Out of scope:** motherboard/system-board replacement (vendor field engineer or depot),
liquid-cooling loops, anything requiring chassis disassembly beyond the FRU level.

## 3. Prerequisites

- [ ] Incident/ticket number with the reported symptom and affected hostname: __________
- [ ] Maintenance window or change approval if the repair requires downtime (see MOP-DCT-006)
- [ ] ESD wrist strap and mat; insulated tools only
- [ ] Spare FRU on hand **before** starting — verify part number matches exactly
      (firmware/family mismatches on drives and PSUs cause repeat failures)
- [ ] BMC/iDRAC/iLO access credentials (break-glass process if needed)
- [ ] Vendor support contract number and phone/portal details
- [ ] Camera for before/after photos of LEDs and part labels
- [ ] Confirm redundancy state first: is this failure already covered by RAID / dual PSU /
      mirrored DIMMs, or is the host degraded? **Never start a repair without knowing
      the redundancy state.**

## 4. Fault isolation — read the signals first

### 4.1 Front-panel LEDs (first 60 seconds)

| Observation | Likely meaning | Next step |
|---|---|---|
| Amber drive LED, solid or blinking | Predicted or hard drive failure | Check RAID status in BMC/RAID BIOS (§5.1) |
| Amber PSU LED / PSU handle LED red | PSU fault or AC input lost | Check PDU breaker, cord, then PSU (§5.2) |
| Amber memory / system-health LED | DIMM fault (often with specific slot LED lit) | Note the slot, check SEL (§5.3) |
| All LEDs dark, no fans | No AC or failed power distribution | Check PDU, cords, then PSU; escalate if both PSUs dead |
| Blinking green/amber system LED | General fault — read the SEL | BMC system event log before touching anything |

### 4.2 BMC / iDRAC / iLO logs (before touching hardware)

1. [ ] Log in to the BMC and export the **System Event Log (SEL)** / Lifecycle Log.
2. [ ] Record: timestamp of first fault, component ID (e.g., `DIMM_B2`, `PS2`, `Disk 4`), error codes.
3. [ ] Check the hardware inventory for anything newly "absent" vs. the last known-good inventory.
4. [ ] Screenshot or export the log — attach to the ticket. **Do not clear the SEL until the vendor case is closed.**

### 4.3 Reseat vs. replace decision tree

```
Fault identified in SEL + LED
│
├─ Is the component hot-swappable AND redundant? (RAID drive, dual PSU, mirrored DIMM)
│   ├─ YES → Attempt ONE reseat while host stays up (drives/PSUs only)
│   │         Reseat fixed it? → Monitor 24h, then close ticket
│   │         Still faulted? → REPLACE the FRU (do not reseat twice)
│   └─ NO (CPU, single PSU, non-redundant DIMM, boot drive in RAID-0)
│         → Schedule downtime → REPLACE, do not reseat under load
│
├─ Intermittent fault with clean reseat and no SEL recurrence in 24h?
│   └─ Replace anyway if the part is a drive (drives lie) or PSU; monitor-only is
│      acceptable for a single corrected ECC storm that does not recur
│
└─ Same slot fails with a known-good part? → STOP. Suspect backplane / slot /
   motherboard → escalate to vendor field engineer. Do not burn through spares.
```

**Rules:**
- One reseat attempt maximum for hot-swap parts. Repeated reseats mask real failures.
- Memory/CPU faults: **no reseat without a maintenance window** unless the host is already down.
- If the SEL shows a *predictive* failure (SMART trip, rising ECC corrections), replace proactively — do not wait for hard failure.

## 5. Component procedures

### 5.1 Drive replacement

1. [ ] Confirm RAID level and state: `Optimal / Degraded / Rebuilding` via BMC or RAID utility
      (storcli / perccli / MegaRAID Storage Manager — see the triage runbook).
2. [ ] Confirm the replacement drive: same capacity or larger, same interface (SAS/SATA/NVMe),
      compatible firmware family.
3. [ ] **Hot-swap (RAID with redundancy):** unlatch the failed drive, wait 30 seconds, insert the
      replacement until it seats firmly, latch.
4. [ ] **Cold-swap (no redundancy):** power down the host gracefully first.
5. [ ] Watch the RAID controller: rebuild should start automatically. Note rebuild start time.
6. [ ] Verify rebuild completion (can take hours on large HDDs — set a check-back reminder).
      Do **not** start a second drive replacement while a rebuild is running.
7. [ ] Confirm RAID state returns to `Optimal`; clear the predictive-failure flag if the controller requires it.
8. [ ] RMA the failed drive per §7 — note: failed drives may contain data; follow the
      sanitization policy (see MOP-DCT-004) before the drive leaves the facility.

### 5.2 PSU replacement

1. [ ] Confirm the surviving PSU is carrying the load (BMC power readings; no throttling).
2. [ ] Check the easy causes first: PDU breaker tripped? Cord damaged or unseated? Wrong PDU feed?
3. [ ] Unplug the failed PSU's cord, release the latch, slide out.
4. [ ] Verify the replacement: same wattage **or higher**, same efficiency rating, same airflow
      direction (front-to-back vs. back-to-front — mixing directions overheats the chassis).
5. [ ] Slide in until the latch clicks; reconnect the cord to the **same PDU feed** it came from (preserve A/B).
6. [ ] Confirm green LEDs on both PSUs and balanced input readings in the BMC.
7. [ ] RMA the failed unit per §7.

### 5.3 Memory (DIMM) replacement

1. [ ] Requires a maintenance window unless the platform supports memory mirroring/sparing
      and the faulted DIMM is already spared out.
2. [ ] Gracefully shut down the host; disconnect AC from **both** PSUs; press and hold the
      power button 10 seconds to drain residual power.
3. [ ] ESD strap on. Open the chassis; locate the faulted slot from the SEL (e.g., `DIMM_B2`)
      and the chassis silkscreen — many vendors label slots on the air baffle.
4. [ ] Release both latches; remove the DIMM by the edges, never the gold contacts.
5. [ ] Install the replacement: match **capacity, speed, rank, and voltage**; populate per the
      vendor's population rules (balanced across channels — an unbalanced config costs performance).
6. [ ] Close the chassis, reconnect AC, power on, and watch POST: confirm full memory capacity
      with no ECC errors.
7. [ ] Run the BMC memory diagnostic or a short memtest if site policy requires; monitor SEL for 24h.
8. [ ] RMA the failed DIMM per §7.

### 5.4 CPU replacement

> CPU replacement is the highest-risk FRU procedure. If a vendor field engineer is
> available under contract, use them.

1. [ ] Maintenance window required. Full shutdown, AC disconnected, residual power drained.
2. [ ] ESD strap on; remove heatsink (note the screw loosening order — typically reverse-numeric).
3. [ ] Clean old thermal paste from the heatsink with isopropyl alcohol and lint-free wipes.
4. [ ] Release the socket latch; lift the CPU **straight up** — never twist. Inspect socket pins
      with magnification; a single bent pin = system-board replacement.
5. [ ] Align the new CPU by the triangle/notch marker; lower straight down — **zero insertion force**,
      never press.
6. [ ] Apply new thermal paste (pea-sized center dot or per vendor spec); remount the heatsink
      in the specified tightening order.
7. [ ] Power on; confirm CPU count, model, and microcode in BMC/BIOS; watch thermals for 30 minutes.
8. [ ] Run a burn-in (see §6). RMA the failed CPU per §7.

## 6. Post-repair burn-in

- [ ] Host boots cleanly to the OS or hypervisor with no SEL errors.
- [ ] 30-minute observation: temperatures, fan speeds, and power draw within normal bands.
- [ ] Drive: RAID `Optimal`; PSU: both green; Memory/CPU: full inventory + clean POST.
- [ ] For memory/CPU repairs: 24-hour SEL watch — any recurrence reopens the ticket.
- [ ] Return the host to service per the change record; update the ticket with part serials (old and new).

## 7. RMA flow

1. [ ] Open the vendor case with: serial number, error codes, SEL export, troubleshooting already performed.
2. [ ] Record the RMA number and expected advance-replacement ETA on the ticket.
3. [ ] Tag the failed part: hostname, date, symptom, RMA number. Store in the quarantine cage — **never loose on a bench**.
4. [ ] Data-bearing parts (drives): sanitize or destroy per MOP-DCT-004 **before** shipping, or use the vendor's certified data-destruction service and keep the certificate.
5. [ ] Ship in anti-static packaging with the RMA paperwork; track the shipment.
6. [ ] On receipt of the replacement: verify part number, add to spares inventory, close the RMA loop in the ticket.

## 8. Safety

- **ESD:** strap on before any internal work; handle boards and DIMMs by the edges.
- **Electrical:** disconnect both PSU cords before internal work; verify with the PSU LEDs dark.
- **Thermal:** heatsinks and CPUs can burn — allow cooldown after shutdown; wear gloves if handling a recently-run heatsink.
- **Weight/sharps:** chassis edges are sharp; server lids can pinch. Two-person lift for chassis removal.

## 9. Rollback

| Trigger | Action |
|---|---|
| Replacement part DOA or wrong part | Reinstall the original part if the host was up; reopen the vendor case; do not improvise with mismatched parts |
| Repair makes it worse (host won't POST) | Restore the original configuration, re-export SEL, escalate to vendor field engineer |
| Rebuild fails / second drive faults during rebuild | STOP — do not pull more drives; escalate immediately (potential data loss); engage backup-restore team |

## 10. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Technician (repair) | | | |
| Peer verifier | | | |
| Operations lead (return to service) | | | |

**Result:** ☐ PASS — repaired, burned in, returned to service ☐ FAIL — escalated, see notes

FRU replaced: __________ Old S/N: __________ New S/N: __________ RMA #: __________
