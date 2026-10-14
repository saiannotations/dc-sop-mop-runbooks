# dc-sop-mop-runbooks

![docs](https://img.shields.io/badge/docs-7%20runbooks-blue)
![status](https://img.shields.io/badge/status-template--library-green)
![license](https://img.shields.io/badge/license-MIT-lightgrey)

A library of **data-center SOPs, MOPs, and triage runbooks** — written as generic,
site-agnostic templates demonstrating data-center operations knowledge: racking,
structured cabling, break-fix, decommissioning, facility health checks, maintenance
windows, and first-15-minutes triage.

> **Generic templates — adapt to site policy before use.** These documents are a
> personal portfolio exercise by Sai Vanka. They were not written for, and do not
> describe procedures used at, any employer. Every document carries this notice.

## Document index

| Doc | Purpose | When to use |
|---|---|---|
| [SOP-rack-and-stack.md](SOP-rack-and-stack.md) | Receiving inspection → rail install → racking → power-on → verification | Any new server install |
| [SOP-structured-cabling.md](SOP-structured-cabling.md) | Copper/fiber handling, bend radius, TIA-606-C-inspired labeling, testing | New or reworked cabling |
| [SOP-hardware-break-fix.md](SOP-hardware-break-fix.md) | Fault isolation for drive/PSU/memory/CPU, reseat-vs-replace tree, RMA flow | Hardware failure on a racked host |
| [MOP-server-decommission.md](MOP-server-decommission.md) | Change-managed decommission with NIST SP 800-88 sanitization guidance | Retiring a server permanently |
| [SOP-power-cooling-healthcheck.md](SOP-power-cooling-healthcheck.md) | Daily/weekly UPS, PDU, CRAC/CRAH checks; ASHRAE 18–27 °C envelope; escalation thresholds | Routine facility rounds |
| [MOP-planned-maintenance-window.md](MOP-planned-maintenance-window.md) | Change ticket fields, notifications, pre-work validation, rollback, post-mortem | Any planned intrusive maintenance |
| [runbook-linux-network-hardware-triage.md](runbook-linux-network-hardware-triage.md) | First-15-minutes flow: BMC → SEL → dmesg → NIC/RAID/filesystem → escalation | "Host down / degraded" page |

## How to use

1. **Print the pre-work checklists** before heading to the data hall — every doc has
   checkbox prerequisites so nothing is forgotten on-site.
2. **Adapt to site policy first.** Replace the generic thresholds, notification
   templates, and escalation contacts with your site's actual values. Nothing here
   overrides a site's change-management or safety policy.
3. **Keep the sign-off blocks.** The technician / peer-verifier / lead signature
   tables are the audit trail — fill them in for every execution.
4. **Pair the docs.** The triage runbook points into the break-fix SOP; the
   decommission MOP points into the cabling SOP. Follow the cross-references.

## Conventions used across all docs

- **Title → Purpose → Scope → Prerequisites → Procedure (numbered steps with
  checkboxes) → Safety → Rollback → Sign-off table.** Same shape everywhere, so any
  technician can pick up any doc cold.
- Checkboxes (`- [ ]`) mark actions that must be confirmed, not just read.
- "Generic template — adapt to site policy before use" appears in every document.

## Portfolio framing

This repository is part of a home-lab data-center portfolio demonstrating
technician-grade operations knowledge: physical install standards, cable plant
discipline, hardware fault isolation, media sanitization per NIST SP 800-88,
facility monitoring against ASHRAE TC 9.9, change-managed maintenance, and
structured incident triage. Companion projects cover NetBox-based rack
documentation, BMC/Redfish inventory collection, and power/thermal monitoring.

## License

MIT — see [LICENSE](LICENSE).
