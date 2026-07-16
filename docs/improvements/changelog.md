# Platform Changelog

## Purpose

Track meaningful completed changes to the homelab platform in a concise, table-based format.

This page is intended to capture high-level platform evolution over time, including:

- new platform services
- major configuration changes
- architecture changes
- upgrades
- storage and networking changes
- observability and GitOps milestones
- backup and recovery improvements
- security-related improvements

Use the backlog for planned or deferred work.
Use this changelog for completed work.

## Suggested Change Types

Recommended values:

- `Added`
- `Changed`
- `Fixed`
- `Removed`
- `Security`
- `Docs`
- `Ops`

## Changelog

| Date | Type | Area | Change | Summary | Related Docs / ADRs |
| ---- | ---- | ---- | ------ | ------- | ------------------- |
| YYYY-MM-DD | Added | Example Area | Example change title | Short summary of what changed and why it mattered | Example page / ADR |
| 2026-07-16 | Added | Backup / Storage | Implemented local NFS backup workflow | Added scheduled NFS backup from `storage1` to the Proxmox backup disk using timestamped rsync hard-link snapshots, retention cleanup, file logging, and basic failure alerting | Architecture: NFS Backup / Runbook: NFS Backup Job |

## Notes

- Keep entries short and readable.
- Add newest completed changes near the top of the table.
- Prefer one row per meaningful platform change.
- If a change is large, summarize it here and link the detailed documentation or ADR.
