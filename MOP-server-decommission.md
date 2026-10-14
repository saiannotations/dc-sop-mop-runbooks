# MOP — Server Decommission (Change-Managed)

> **Generic template — adapt to site policy before use.** This document is a personal
> portfolio exercise demonstrating data-center operations knowledge. It was not written
> for, and does not describe procedures used at, any employer.

| Field | Detail |
|---|---|
| Document ID | MOP-DCT-004 |
| Version | 1.0 |
| Last reviewed | 2026-10-14 |
| Owner (template) | Data Center Operations |
| Change class | Standard / Normal (site CAB determines) |

## 1. Purpose

Define the controlled, auditable process for permanently removing a server from
service — covering approvals, backup verification, data sanitization per
NIST SP 800-88, asset/CMDB updates, cable removal, and rack restoration — so no
decommissioned asset leaves residual data or orphaned records behind.

## 2. Scope

**In scope:** full server decommissions (host leaves the fleet). Partial repurposes
follow the same sanitization steps before reassignment.

**Out of scope:** temporary host evacuation for maintenance (host returns to service);
storage-array decommissions (separate MOP); off-site e-waste logistics beyond handoff.

## 3. Prerequisites

- [ ] Change ticket approved with all sign-offs in §9 — **no decommission without written approval**
- [ ] Asset details confirmed: hostname, asset tag, serial number, rack/U position, BMC IP
- [ ] Service owner identified and has confirmed the host carries no production dependency
      (DNS, monitoring, backup jobs, load-balancer pools, CMDB relationships checked)
- [ ] Backup verification completed (§5) — restore test or backup-team sign-off on file
- [ ] Sanitization method selected per §6 and media inventory taken (every drive accounted for,
      including M.2 boot devices and any USB media left in the chassis)
- [ ] Downtime/impact: none expected (host already drained) — confirm with the service owner
- [ ] Tools: crash cart or BMC access, bootable sanitization media (if using Purge via software),
      degausser/crusher access (if using Destroy), label printer, camera

## 4. Procedure

### 4.1 Approvals & pre-work

1. [ ] Service owner confirms in the change ticket: *"Host <hostname> is cleared for decommission;
      no production dependencies remain."*
2. [ ] Backup administrator confirms (§5).
3. [ ] Security/data-governance approver confirms the sanitization method (§6) matches the
      data classification of the host.
4. [ ] Change manager / CAB approves the execution window.
5. [ ] Notify monitoring/NOC of the planned host removal so alerts don't page during the work.

### 4.2 Drain & power down

1. [ ] Verify the host is drained: no workloads, removed from load-balancer pools, cluster
      membership revoked, monitoring set to maintenance or decommissioned state.
2. [ ] Graceful OS shutdown. Confirm via BMC that the host is powered off (not just OS-halted).
3. [ ] Photograph the rack position and cabling before touching anything.

### 4.3 Backup verification

1. [ ] Identify the last good backup (or replica/export) covering this host's data.
2. [ ] Verify backup integrity: backup job status `Success`, and — for any host whose data
      cannot be regenerated — a **test restore** to an isolated location with checksum comparison.
3. [ ] Record on the ticket: backup system, job ID, timestamp, and who verified it.
4. [ ] **Rule: no verified backup (or documented owner waiver for ephemeral hosts) = no decommission.**
      Hosts with regulated data additionally require the retention-policy sign-off before destruction.

### 4.4 Data sanitization (NIST SP 800-88 Rev. 1)

Select the method by media type and data classification. When in doubt, go one level stronger.

| Method | What it does | Use when |
|---|---|---|
| **Clear** | Overwrite user-addressable areas (e.g., single-pass overwrite, factory reset for flash via vendor utility) | Low-sensitivity data, media staying in-house for reuse |
| **Purge** | Firmware-level commands: ATA Secure Erase, NVMe Format / Sanitize, SCSI SANITIZE, cryptographic erase (crypto-erase) by destroying the media encryption key | Sensitive data; SSDs/NVMe (overwrite is unreliable on flash due to over-provisioning — **never rely on Clear alone for SSDs**) |
| **Destroy** | Disassemble/shred/crush/incinerate to make recovery infeasible | Top-sensitivity data, failed media that cannot be Purged, regulatory requirement, or any drive leaving the facility without a trusted chain of custody |

**Decision guidance:**

- [ ] HDD (spinning): Clear is acceptable for low-sensitivity; Purge (overwrite + verify) for
      sensitive; Destroy for regulated/failed drives.
- [ ] SSD / NVMe / flash (including M.2 boot drives): **Purge via crypto-erase or NVMe Sanitize
      minimum** — block-level overwrite does not reach over-provisioned areas. If Purge commands
      fail or are unsupported, Destroy.
- [ ] Failed/dead drives that cannot be wiped: **Destroy** — no exceptions for sensitive data.
- [ ] Record per-drive: serial number, method used, tool/command, verification result
      (e.g., read-back sample or tool pass certificate), technician, date.
- [ ] For Destroy: witness sign-off and certificate of destruction filed with the ticket.
- [ ] Encryption note: crypto-erase is only valid if encryption was enabled for the drive's
      entire service life — verify before relying on it.

### 4.5 Asset / CMDB update

1. [ ] Update the asset record: status → `Decommissioned`, date, ticket number, sanitization method.
2. [ ] Update CMDB: host CI status → retired; remove relationships (do not delete history).
3. [ ] Remove DNS entries (forward and reverse), DHCP reservations, and monitoring objects.
4. [ ] Release the IP addresses back to the pool; update IPAM.
5. [ ] Remove the BMC from management VLAN inventory / jump-host allow-lists.

### 4.6 Physical removal

1. [ ] Disconnect power cords (both feeds) and coil them for reuse or e-waste.
2. [ ] Disconnect and remove all data/management cables per the label IDs; update the cable
      database (see SOP-DCT-002).
3. [ ] Slide the chassis out on its rails; two-person lift (or server lift) down.
4. [ ] Remove rail kits if the U is not being immediately reused; otherwise leave rails for the next install.
5. [ ] Install **blanking panels** in all vacated U (airflow containment — never leave open U).
6. [ ] Clean the vacated U: wipe rails, remove old labels, vacuum dust.
7. [ ] Stage the chassis in the decommission holding area with its asset tag and ticket number attached.
8. [ ] Final rack photo (front and rear) attached to the ticket.

## 5. Verification checklist (before ticket close)

- [ ] Service-owner decommission approval on file
- [ ] Backup verified (job ID + verifier recorded) or documented waiver
- [ ] Every data-bearing device sanitized per §4.4 with per-drive records
- [ ] CMDB/asset/DNS/DHCP/monitoring/IPAM updated
- [ ] Cables removed and cable database updated
- [ ] Blanking panels installed; rack photographed
- [ ] Chassis staged in holding with asset tag + ticket number

## 6. Safety

- **Data safety:** sanitization records are the audit trail — incomplete records = failed decommission.
- **Electrical:** confirm the host is fully powered off via BMC before pulling cords; treat every cord as live until verified.
- **Lifting:** chassis + rails are heavy and awkward — two-person lift or server lift, clear aisle.
- **Media handling:** crushed/shredded media goes to the secure e-waste stream with chain-of-custody paperwork; failed drives never go in general trash.

## 7. Rollback

| Trigger | Action |
|---|---|
| Service owner reports a missed dependency mid-work | STOP immediately; restore power and cabling from the before-photos; re-add to monitoring; reopen the change ticket — do not proceed until the dependency is resolved |
| Sanitization tool fails on a drive | Quarantine the drive, mark it for Destroy, continue with the rest — never skip a drive |
| Backup verification fails | Halt the decommission; engage the backup team; reschedule |

> Note: once sanitization is complete, data recovery is intentionally impossible — the
> "rollback" for a wrongly-decommissioned host is restore-from-backup, which is exactly
> why §4.3 is a hard gate.

## 8. Post-mortem / closeout

- [ ] Ticket updated with all checklist evidence (photos, sanitization log, approvals).
- [ ] Holding-area handoff signed: asset tag, ticket number, sanitization status.
- [ ] If any step deviated from this MOP, note the deviation and reason — feed it back into the next document review.

## 9. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Service owner (dependency clearance) | | | |
| Backup administrator (backup verified) | | | |
| Security / data governance (sanitization method) | | | |
| Technician (execution) | | | |
| Operations lead (closeout) | | | |

**Result:** ☐ PASS — host decommissioned, sanitized, records updated ☐ FAIL — halted, see notes

Hostname: __________ Asset tag: __________ Ticket: __________ Sanitization method(s): __________
