# Backup and Recovery

## Goals

- protect critical data offsite
- enable recovery from host loss
- validate that restore procedures work
- keep AWS costs low

## Backup Strategy

### Offsite Target

- AWS S3 bucket dedicated to homelab backups

### Primary Backup Tool

- restic

### Secondary Learning Tool

- Velero

## Backup Scope

Back up:

- NFS data from `svc1`
- selected configuration exports
- backup keys in secure form
- optional etcd snapshots

Git repositories are already offsite in GitHub and are part of the recovery strategy.

## Recovery Order

1. restore access to Git repositories
2. rebuild Proxmox
3. provision VMs with Terraform
4. configure VMs with Ansible
5. rebuild Kubernetes
6. restore platform services
7. restore persistent data

## Restore Testing

Backups are not considered valid until restore procedures have been tested.
