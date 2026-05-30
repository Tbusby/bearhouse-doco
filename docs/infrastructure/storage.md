# Storage

## Hypervisor Storage

The Proxmox host uses local NVMe storage for:

- Proxmox OS
- VM disks
- templates
- snapshots

## Kubernetes Persistent Storage

Initial persistent storage will be provided through:

- NFS server on `svc1`
- `nfs-subdir-external-provisioner` in Kubernetes

## Rationale

NFS is selected initially because it is:

- simple
- stable
- easy to understand
- practical in a single-host environment

## Limitations

This storage design is not resilient to physical host failure.

For this environment, resilience depends on:

- backups
- documented restore procedures
- reproducible infrastructure

## Future Options

- Longhorn
- MinIO
- additional backup tiers
