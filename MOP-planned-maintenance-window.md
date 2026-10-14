# MOP — Planned Maintenance Window (Change-Managed)

> **Generic template — adapt to site policy before use.** This document is a personal
> portfolio exercise demonstrating data-center operations knowledge. It was not written
> for, and does not describe procedures used at, any employer.

| Field | Detail |
|---|---|
| Document ID | MOP-DCT-006 |
| Version | 1.0 |
| Last reviewed | 2026-10-14 |
| Owner (template) | Data Center Operations |
| Change class | Normal (CAB approval required) |

## 1. Purpose

Provide the end-to-end method of procedure for any planned maintenance that carries
outage or degradation risk — firmware upgrades, PDU work, network maintenance,
host evacuations — from change ticket through execution, verification, rollback,
and post-mortem.

## 2. Scope

**In scope:** all planned intrusive work in the data hall or on supporting infrastructure.

**Out of scope:** emergency break-fix (follow the incident process, then document);
routine non-intrusive checks (see SOP-DCT-005).

## 3. Change ticket fields (required before scheduling)

- [ ] **Title:** `[MOP] <system> — <work description> — <date>`
- [ ] **Change class:** Normal / Standard / Emergency
- [ ] **Risk & impact:** what could break, who is affected, expected degradation
- [ ] **Affected CIs:** hostnames, racks, PDUs, network devices (linked CMDB records)
- [ ] **Execution window:** start/end with timezone, e.g. `2026-11-02 01:00–05:00 America/Chicago`
- [ ] **Implementer(s)** and **on-site verifier** (must be different people for high-risk work)
- [ ] **Pre-work validation** steps (§6) listed explicitly
- [ ] **Execution steps** (§7) numbered and ordered
- [ ] **Verification steps** (§8) with pass criteria
- [ ] **Rollback plan** (§9) with decision triggers and time budget
- [ ] **Communications plan** (§5): who is notified, when, via what channel
- [ ] **Approvals:** technical approver, service owner, change manager / CAB

## 4. Prerequisites

- [ ] CAB approval recorded on the ticket — **no approval, no work**
- [ ] Backups / snapshots verified current for everything in scope
- [ ] Rollback artifacts staged and tested (firmware images, configs, spare parts on-site)
- [ ] Monitoring silenced or annotated for the window (with an expiry — never indefinite)
- [ ] Implementer has physical access, credentials, and vendor support numbers
- [ ] Go/no-go call completed 24h before (and again at window start)

## 5. Notification templates

### 5.1 Advance notice (send at T-5 business days, repeat at T-24h)

> **Subject:** Planned maintenance — <system> — <date> <start>–<end> <TZ>
>
> Hello,
>
> Planned maintenance is scheduled for **<system>** on **<date>** from
> **<start>** to **<end> <TZ>**.
>
> **Expected impact:** <none / degraded / outage — be specific about what users will experience>
> **Affected services:** <list>
> **Change record:** <ticket number>
>
> No action is required from you. If you observe issues after <end + 1h>, contact
> <on-call contact / NOC number>.
>
> Thank you,
> <implementer>, Data Center Operations

### 5.2 Window start / end updates

> **Start:** "Maintenance on <system> has begun. Expected completion <time> <TZ>. Updates every <60> min."
>
> **End:** "Maintenance on <system> is complete. Verification passed. <system> is returned to normal service as of <time> <TZ>."

### 5.3 Rollback notice (if triggered)

> "Maintenance on <system> encountered <brief reason> and is being rolled back per the
> approved plan. Current status: <status>. Next update in <30> min. Change record: <ticket>."

## 6. Pre-work validation (day of, before touching anything)

1. [ ] Go/no-go: all approvers confirm; no conflicting changes in the window; staffing confirmed.
2. [ ] Verify backups/snapshots completed successfully **today**.
3. [ ] Verify rollback artifacts are present and readable (checksums match).
4. [ ] Baseline capture: current firmware versions, config backups, key health metrics
      (screenshot or export — this is what "normal" looks like for §8).
5. [ ] Confirm monitoring annotations/silences are active with correct expiry.
6. [ ] Confirm the notification in §5.1 was sent and acknowledged by stakeholders.
7. [ ] Safety briefing: EPO locations, ESD, who calls the abort.

**Abort criteria at this stage:** any missing prerequisite = no-go. Reschedule; do not improvise.

## 7. Execution steps

> Fill in per maintenance. The template below shows the required structure.

1. [ ] Step 1 — <action>. Expected result: <result>. Timebox: <min>.
2. [ ] Step 2 — <action>. Expected result: <result>. Timebox: <min>.
3. [ ] ...
4. [ ] After each step: confirm the expected result before proceeding. **If a step's result
      does not match, stop and assess — do not push through to the next step.**
5. [ ] Log actual start/end times and any deviations next to each step.

**Example — firmware upgrade on a top-of-rack switch pair:**

1. [ ] Confirm MLAG/vPC peer is healthy and carrying traffic. Expected: both peers up, no errors.
2. [ ] Upgrade the standby peer; reload. Expected: peer rejoins, firmware version = target.
3. [ ] Verify traffic on the upgraded peer for 15 min. Expected: zero unexpected drops.
4. [ ] Fail traffic over; upgrade the former active peer; reload. Expected: same as step 2.
5. [ ] Restore normal forwarding; verify. Expected: both peers on target firmware, balanced traffic.

## 8. Verification (before declaring success)

- [ ] Every affected system shows the intended end state (firmware version, config, service status).
- [ ] Key health metrics match or beat the §6 baseline (temps, errors, latency, throughput).
- [ ] End-to-end functional test: representative user/system transaction succeeds.
- [ ] Monitoring is green with silences removed; alerting confirmed working (trigger a test alert if policy allows).
- [ ] No unexpected entries in system/BMC/switch logs from the window.
- [ ] Stakeholder confirmation for user-facing services ("can you verify your application?").

**All boxes checked = success. Any box failing = assess rollback (§9).**

## 9. Rollback plan

| Element | Detail |
|---|---|
| Rollback trigger | Any verification step fails and cannot be fixed within <30> min; or any unexpected outage occurs |
| Rollback decision maker | <name / role> — one person, named in advance |
| Rollback steps | 1. <reverse step N> … 2. <restore config/firmware from §6 baseline> … 3. <verify restored state> |
| Rollback time budget | Must complete by <window end − 60 min>; if rollback cannot complete in time, invoke the incident process |
| Data consideration | Rollback restores service state; any data written during the window is <preserved / reconciled per app owner> |
| Communication | Send the §5.3 rollback notice immediately on decision |

**Rule:** a rollback plan that has never been read by the implementer is not a plan.
Walk through it in the go/no-go call.

## 10. Post-mortem / closeout

Within 2 business days of the window:

- [ ] Ticket updated: actual times, step log, verification evidence, deviations.
- [ ] **What went well / what didn't / what we'll change** — three bullets minimum, blameless.
- [ ] Action items filed as follow-up tickets with owners and dates.
- [ ] If rollback was used: root-cause analysis attached; repeat maintenance re-planned, not retried blindly.
- [ ] Document review: does this MOP or a related SOP need an update based on what was learned?

## 11. Safety

- Change discipline is safety: unapproved work is how outages happen.
- Two-person rule for high-risk steps (power, firmware on shared infrastructure).
- Fatigue management: windows in the small hours still require a rested implementer and a reachable backup.

## 12. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Technical approver | | | |
| Service owner | | | |
| Change manager / CAB | | | |
| Implementer | | | |
| Verifier (independent) | | | |

**Result:** ☐ SUCCESS — verified, closed ☐ ROLLED BACK — see post-mortem ☐ ABORTED pre-work — rescheduled

Window: __________ Actual start/end: __________ Ticket: __________
