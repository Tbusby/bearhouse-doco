# Runbook: NFS Backup Job

## Purpose

Run and validate the NFS backup workflow that protects Kubernetes NFS-backed application data.

This runbook is intended for the current local snapshot-based backup process from `svc1` to the Proxmox backup disk.

## Scope

Applies to the current NFS backup workflow for:

- source host: `svc1`
- source path: `/srv/nfs`
- destination path: `/mnt/pve/backup-local/backups/` on the Proxmox host

This runbook covers:

- script placement
- manual validation
- scheduled execution
- logging
- retention behavior

This runbook does not cover:

- off-site backup
- full disaster recovery
- NFS restore procedures
- Proxmox VM backup jobs

## Prerequisites

- `svc1` is functioning as the NFS server
- the Proxmox backup disk is mounted and available at `/mnt/pve/backup-local`
- SSH key-based access from `svc1` to the Proxmox backup target is working
- the backup target directory structure exists or can be created
- the backup script has been reviewed and placed on `svc1`

## Planned Values

| Item | Value |
| ---- | ----- |
| Source host | `svc1` |
| Source path | `/srv/nfs/k8s` |
| Destination host | Proxmox host |
| Destination base path | `/mnt/pve/backup-local/backups/` |
| Script path | `/usr/local/bin/nfs-backup.sh` |
| Log file | `/var/log/nfs-backup/nfs-backup.log` |
| Retention model | keep newest configured snapshots |
| Scheduling method | root cron |

## Risks

- backup target is local to the same physical host and not off-site
- running the script without validating SSH access first
- using the wrong source or destination path
- retention deleting snapshots more aggressively than intended if misconfigured
- assuming backup success without testing restore workflows

## Procedure

### 1. Install the script

**_NOTE:_** This should be installed via ansible but worth knowing

- [ ] Place the backup script on `svc1` at:

  ```text
  /usr/local/bin/nfs-backup.sh
  ```

- [ ] Install it with appropriate ownership and permissions:

  ```bash
  sudo install -o root -g root -m 700 nfs-backup.sh /usr/local/bin/nfs-backup.sh
  ```

### 2. Validate SSH connectivity

- [ ] Confirm the execution context can access the Proxmox backup target over SSH
- [ ] If the script will run as root, test SSH access as root:

  ```bash
  sudo ssh -o BatchMode=yes backupsvc@<proxmox-host> 'echo ok'
  ```

- [ ] Confirm the destination path can be created or accessed

### 3. Run the script manually

- [ ] Run the backup script manually before scheduling it:

  ```bash
  sudo /usr/local/bin/nfs-backup.sh
  ```

- [ ] Confirm the script completes successfully

### 4. Validate backup output

- [ ] On the Proxmox host, confirm a timestamped snapshot directory exists under:

  ```text
  /mnt/pve/backup-local/backups/
  ```

- [ ] Confirm a `latest` symlink exists and points to the most recent snapshot
- [ ] Confirm expected backup data exists in the snapshot directory

### 5. Validate logging

- [ ] Confirm the log file exists on `svc1`:

  ```text
  /var/log/nfs-backup/nfs-backup.log
  ```

- [ ] Review recent log output:

  ```bash
  sudo tail -n 50 /var/log/nfs-backup/nfs-backup.log
  ```

- [ ] Confirm the log is readable and not excessively noisy

### 6. Validate retention behavior

- [ ] Confirm the configured retention count in the script matches expectations
- [ ] After multiple runs, confirm older snapshots are removed automatically
- [ ] Confirm the newest snapshots are preserved as expected

### 7. Configure nightly scheduling

- [ ] Edit root’s crontab:

  ```bash
  sudo crontab -e
  ```

- [ ] Add a nightly job, for example:

  ```cron
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  0 1 * * * /usr/local/bin/nfs-backup.sh
  ```

- [ ] Save the crontab and confirm it is installed

### 8. Configure log rotation

- [ ] Add a logrotate config for the backup log
- [ ] Confirm log rotation is installed and valid

### 9. Record the final state

- [ ] Record the script path
- [ ] Record the source and destination paths
- [ ] Record the retention count
- [ ] Record the log path
- [ ] Record the scheduling method
- [ ] Update architecture and recovery documentation

## Validation

The backup workflow is successful when:

- the script runs successfully manually
- timestamped backup directories are created on the Proxmox backup disk
- the `latest` symlink is updated
- retention removes older snapshots as expected
- logs are written locally on `storage1`
- the nightly cron job is configured

## Rollback / Fallback

- Remove the cron job if scheduled execution must be stopped
- Restore the previous version of the backup script if changes break the workflow
- Clean up test snapshot directories manually if needed
- Do not assume backup coverage is valid until restore has also been tested

## Follow-Up

- Create and validate an NFS restore procedure
- Perform a documented restore test
- Review retention settings as data volume grows
- Consider future off-host or off-site backup replication

## Notes

- This is a local backup tier, not a full disaster recovery solution
- The current design prioritizes simplicity and recoverability over maximum sophistication
- Restore testing is required to complete the recovery loop
