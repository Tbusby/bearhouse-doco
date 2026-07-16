# NFS Backup

## Purpose

Describe the backup architecture used to protect NFS-backed Kubernetes application data in the homelab platform.

## Summary

The platform now includes a local backup workflow for NFS-backed workload data.

The backup model uses:

- `svc1` as the source of persistent application data
- a dedicated backup disk attached to the Proxmox host
- timestamped `rsync` backups with hard-link snapshots
- retention-based cleanup
- local logging on the NFS server

This provides a practical first backup layer for the most important current platform data.

## Why This Was Added

The platform now contains meaningful persistent state, especially application and workload data stored on the NFS export used by Kubernetes.

Without a backup workflow, loss or corruption of NFS-backed data would create unnecessary recovery risk for:

- current persistent platform data
- future application data
- stateful workloads deployed later

This backup layer was added because NFS-backed application data is currently the highest-priority data to preserve.

## Architectural Role

The backup workflow protects the current storage layer by creating point-in-time filesystem snapshots of the NFS export.

Its role is to provide:

- a recoverable local copy of NFS-backed application data
- multiple restore points
- retention-managed backup history
- a simple operational backup model that can be tested and understood easily

This supports the broader platform recovery strategy, which prefers:

- rebuilding infrastructure from code
- restoring only stateful data that cannot be easily re-created

## Current Backup Design

The current design uses:

- source host: `svc1`
- source path: `/srv/nfs`
- destination host: Proxmox host
- destination path: `/mnt/pve/backup-local/backups/`
- transport: `rsync` over SSH
- snapshot style: timestamped directories
- snapshot optimization: hard-link snapshots using `rsync --link-dest`
- retention: automatic deletion of older snapshots
- execution: scheduled local job on `svc1`

## Why This Design Was Chosen

This design was chosen because it is:

- simple to understand
- easy to validate manually
- appropriate for the current size of the lab
- aligned with the current rebuild-plus-restore recovery model
- easier to operate than introducing cloud backup or more advanced backup software immediately

This keeps the first backup implementation practical and testable.

## Why the Backup Target Is Local

The backup destination is a dedicated backup disk installed in the Proxmox host.

This was chosen because it is:

- fast to implement
- low cost
- operationally simple
- sufficient for an initial local backup tier

This is intentionally a **local backup tier**, not a full disaster-recovery or off-site backup solution.

## Known Limitation

The backup target shares the same physical host as the rest of the homelab platform.

This means it helps protect against:

- accidental deletion
- some forms of data corruption
- workload-level recovery needs
- storage VM rebuild scenarios

It does **not** fully protect against:

- total host failure
- physical theft
- catastrophic hardware loss affecting the whole host

This limitation is acceptable for the current phase of the lab and should remain documented.

## Snapshot Model

Backups are stored as timestamped snapshot directories.

Each run creates a new directory such as:

```text
/mnt/pve/backup-local/backups/2026-07-16_010000/
```

A `latest` symlink is also maintained to point to the newest snapshot.

To reduce space usage, unchanged files are hard-linked from the previous snapshot using `rsync --link-dest`.

This provides:

- point-in-time restore directories
- reduced storage growth for unchanged data
- simple browsing and restore behavior

## Retention Model

The backup script automatically keeps only the newest configured number of snapshots and deletes older ones.

This provides:

- predictable local disk usage
- simple automated retention
- low operational overhead

Retention count should be reviewed as data volume and backup frequency evolve.

## Logging Model

Backup execution logs are written locally on `svc1`.

Current log path:

```text
/var/log/nfs-backup/nfs-backup.log
```

Log rotation is handled separately through `logrotate`.

This keeps backup activity observable without requiring external logging infrastructure.

## Failure Alerting

The backup script includes failure alerting behavior.

If the backup job fails:

- the error is written to the local log
- the script attempts to send an alert via the configured local mail mechanism (Mail is currently not implemented)

This provides a basic initial operational failure signal.

## Current Validated State

The NFS backup workflow is installed and functioning.

Validated outcomes include:

- backup destination path exists on the Proxmox host backup storage
- `svc1` can reach the destination over SSH
- backups are written as timestamped snapshot directories
- hard-link snapshots reduce storage growth for unchanged files
- retention cleanup runs automatically
- local logs are written
- backup output noise has been reduced to cleaner summary logging

## Architectural Benefits

Adding this backup layer gives the platform:

- a first real recovery path for stateful application data
- multiple restore points for NFS-backed workloads
- a simple and auditable local backup model
- better alignment with the documented rebuild-plus-restore recovery strategy

## Future Direction

Possible future improvements include:

- documented restore testing and recovery drills
- additional backup validation
- off-host or off-site replication
- stronger alerting integration
- backup coverage expansion for other data classes if needed
