# Runbook — Linux / Network / Hardware Triage (First 15 Minutes)

> **Generic template — adapt to site policy before use.** This document is a personal
> portfolio exercise demonstrating data-center operations knowledge. It was not written
> for, and does not describe procedures used at, any employer.

| Field | Detail |
|---|---|
| Document ID | RB-DCT-007 |
| Version | 1.0 |
| Last reviewed | 2026-10-14 |
| Owner (template) | Data Center Operations |

## 1. Purpose

Give the on-site technician a deterministic first-15-minutes flow for any
"host down / host degraded" page: establish what's reachable, capture evidence
**before** rebooting, and escalate with a complete picture instead of a guess.

## 2. Scope

Linux hosts (bare metal or hypervisor) with BMC/iDRAC/iLO out-of-band management.
Network triage covers the host's NICs and its switch ports; deeper network
diagnosis belongs to the network team.

## 3. Golden rules

1. **Capture before you reboot.** Logs are evidence; a reboot destroys the crime scene.
2. **Out-of-band first.** If the BMC answers, you have eyes even when the OS is dead.
3. **One change at a time.** Note what you changed and when — the ticket timeline is sacred.
4. **Know your escalation point before you need it** (§9).

## 4. Minute 0–3 — Is it reachable?

- [ ] **Ping the BMC** (out-of-band IP). Reachable → you have a management path; go to §5.
- [ ] **Ping the host OS IPs.** OS answers but service is down → likely OS/service issue, not hardware; still check BMC SEL for hardware events.
- [ ] **Neither answers:** check from a second source (jump host vs. laptop) to rule out your own network path.
- [ ] **BMC answers, OS doesn't:** the OS is hung, crashed, or powered off — proceed to §5, then §6.
- [ ] **Nothing answers (BMC + OS):** possible power loss or network-path failure:
  1. [ ] Check the rack PDU display: is the outlet energized? Breaker tripped?
  2. [ ] Check front-panel LEDs on the host (see SOP-DCT-003 §4.1).
  3. [ ] Check the switch port LEDs for the BMC and host links.
  4. [ ] If the whole rack/row is dark → facility power event: escalate per SOP-DCT-005 thresholds immediately.

## 5. Minute 3–7 — BMC interrogation (do this before touching the OS)

1. [ ] **Power state:** is the host actually powered on? (BMC power status vs. "soft-off".)
2. [ ] **System Event Log (SEL):** export it. Look for the *first* error in the timeline —
      the first fault is usually the cause; everything after is collateral.
      - Memory ECC errors → DIMM fault (SOP-DCT-003 §5.3)
      - Drive predictive failure / RAID degraded → drive fault (SOP-DCT-003 §5.1)
      - PSU input lost / fault → power path (SOP-DCT-003 §5.2)
      - CPU CATERR / machine-check → CPU fault; do not reboot-loop — capture and escalate
      - Temperature excursions → cooling issue; check SOP-DCT-005
3. [ ] **Sensor readings:** inlet temp, CPU temps, fan RPMs, PSU input watts. Screenshot the abnormal ones.
4. [ ] **Hardware inventory:** anything showing "absent" that should be present?
5. [ ] **Virtual console:** launch the BMC remote console — what is on screen?
      - GRUB prompt → boot device issue (§7)
      - Kernel panic → capture the panic text (photo/screenshot) before rebooting
      - Login prompt → OS is up; the problem is network or service-level (§6)
      - Black screen, no POST → hardware or power; check SEL first

## 6. Minute 7–11 — OS & network checks (if the OS is reachable)

Run over SSH or the BMC virtual console / serial-over-LAN. Capture output to the ticket.

**Boot & system state:**

```bash
# How long has it been up? (a recent unexpected reboot is itself a clue)
uptime; who -b

# Recent errors across the journal
journalctl -p err -b --no-pager | tail -50

# Kernel/hardware messages (the single most useful command in triage)
dmesg -T | tail -100
# Look for: mce:, EDAC, I/O error, EXT4-fs error, nvme, ata errors, link down
```

**NIC link status:**

```bash
# Link state per interface
ip link show
ethtool eth0 | grep -E 'Link detected|Speed|Duplex'
# Link detected: no  → physical problem: cable, switch port, or NIC

# Errors and drops (rising = real problem, static = historical)
ip -s link show eth0
ethtool -S eth0 | grep -iE 'error|drop|crc|missed'
```

- [ ] If `Link detected: no`: reseat the cable at both ends, try a known-good cable,
      check the switch port status/LEDs, then escalate to the network team with the
      switch name + port number.
- [ ] If link is up but traffic fails: check IP config, gateway ARP, and whether the
      switch port shows errors — then escalate with `ethtool -S` output attached.

**RAID status:**

```bash
# Broadcom/LSI MegaRAID (storcli)
storcli /c0/v0 show          # virtual drive state: Optl / Dgrd / Rbld
storcli /c0/eall/sall show   # physical drive states: Onln / Failed / Rbld

# Dell PERC (perccli — same syntax family)
perccli /c0/v0 show

# Older MegaCLI
megacli -LDInfo -Lall -aAll | grep -i state
```

- [ ] `Degraded` + one `Failed` drive → drive replacement per SOP-DCT-003 §5.1.
- [ ] Rebuild in progress → do NOT pull drives; note % complete and check back.
- [ ] Multiple failed drives → STOP, escalate immediately (possible data loss).

**Filesystem:**

```bash
df -h                          # full filesystems cause mysterious failures
dmesg -T | grep -iE 'ext4| xfs|I/O error|Buffer I/O'
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
```

- [ ] 100% full on `/` or `/var` → clear logs / expand before anything else; check what filled it.
- [ ] I/O errors in dmesg → storage problem; correlate with RAID status above.

## 7. Minute 11–15 — Decide: fix, reboot, or escalate

| Situation | Action |
|---|---|
| Cause found and it's a hot-swap FRU (drive/PSU) with redundancy intact | Replace per SOP-DCT-003; no reboot needed |
| OS hung, BMC healthy, no hardware errors in SEL | Capture `dmesg` + `journalctl` + console screenshot, **then** BMC power-cycle (graceful first, hard reset only if graceful fails) |
| Kernel panic captured | One reboot attempt; if it panics again, stop rebooting and escalate with the panic text |
| Boot device missing / GRUB rescue | Check RAID + boot order in BIOS; do not reinstall anything without the service owner's approval |
| CPU machine-check (MCE) in SEL/dmesg | Do not loop reboots — capture everything, escalate to L2/vendor |
| Cause unknown after 15 minutes | Escalate per §9 with the evidence bundle (§8) — 15 minutes of structured triage beats an hour of guessing |

## 8. Evidence bundle — capture BEFORE rebooting

Attach all of the following to the ticket before any reboot or repair:

- [ ] BMC SEL export (full, not just the tail)
- [ ] Sensor screenshot showing the fault state
- [ ] Remote-console screenshot/photo of what's on screen
- [ ] `dmesg -T` tail (100+ lines) and `journalctl -p err -b` output
- [ ] `ip link show` + `ethtool` output for affected NICs
- [ ] RAID controller status output
- [ ] Timeline: when the alert fired, what you checked, what you changed, exact times

## 9. Escalation matrix

| Level | Who | When |
|---|---|---|
| **L1** (you) | On-site technician | First 15 min triage, FRU replacement, evidence capture |
| **L2** | Senior systems / on-call engineer | Unknown cause after 15 min, OS-level issues beyond triage, repeat failures |
| **Network team** | NetOps on-call | Switch-port errors, link-down with good cable, VLAN/routing issues |
| **Facilities** | Facilities on-call | Power/cooling anomalies per SOP-DCT-005 thresholds |
| **Vendor** | Hardware vendor support (contract # on file) | Confirmed FRU fault needing RMA, CPU/MCE faults, backplane suspicions |

**Escalation message format (use this verbatim structure):**

> Host <hostname> <down/degraded> since <time TZ>. BMC <reachable/unreachable>.
> First SEL fault: <text> at <time>. OS state: <up/hung/panic/off>.
> Evidence attached: <list>. Actions taken: <list>. Request: <specific ask>.

## 10. Safety

- No internal hardware work without ESD protection and (for non-hot-swap parts) full power-down.
- Never reboot a host with an active RAID rebuild unless explicitly directed by L2.
- A host that panics twice on boot stays down until L2/vendor reviews the panic — reboot loops destroy evidence and can worsen storage corruption.

## 11. Sign-off (per incident)

| Field | Detail |
|---|---|
| Ticket | |
| Technician | |
| Triage start/end | |
| Outcome | ☐ Resolved on-site ☐ Escalated to: __________ |
| Evidence attached | ☐ |

Notes: _______________________________________________________________
