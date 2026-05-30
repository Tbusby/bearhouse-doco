
# Proxmox

## Purpose

Proxmox VE provides the virtualization layer for the platform.

It is responsible for:

- hosting all VMs
- template management
- snapshots
- storage management
- virtual networking

## Design Notes

- installed directly on bare metal
- dedicated host for the homelab platform
- minimal manual configuration after initial setup
- VM provisioning managed through Terraform where possible

## Standards

- Proxmox host should be updated on a regular maintenance schedule
- host access should use SSH keys and secured administrative access
- VM templates should be versioned and documented
- avoid creating unmanaged "pet" VMs

---

## Details

### Host Details

|                 |                        |
| --------------- | ---------------------- |
| Hostname        | proxmox01.bearhouse.cc |
| Proxmox version | 8.4.19                 |

### Network Configuration

| Interface | Type | Role | Address / CIDR | Gateway | Bridge | Notes |
| --------- | ---- | ---- | -------------- | ------- | ------ | ----- |
| `lo` | Loopback | Local host loopback | `127.0.0.1/8` | None | None | Standard loopback interface |
| `en0` | Physical NIC | Unused / Standby | None | None | none | Motherboard network port - unused |
| `enp2s0f0` | Physical NIC | Uplink for Proxmox bridge | None | None | `vmbr0` | Active physical interface connected to LAN |
| `enp2s0f1` | Physical NIC | Unused / Standby | None | None | None | Present but not currently in use (Possibly broken) |
| `vmbr0` | Linux bridge | Proxmox management + VM network | `192.168.1.10/24` | `192.168.1.1` | N/A | Main bridge for host management and guest connectivity |

### Storage Summary

| Storage Name | Type | Backing Disk | Size | Primary Use | Notes |
| ------------ | ---- | ------------ | ---- | ----------- | ----- |
| `local` | Directory | 512 GB M.2 | ~host-defined | ISOs, templates, snippets | Default Proxmox local storage on OS disk |
| `local-lvm` | LVM-thin | 512 GB M.2 | ~host-defined | Default VM disk storage | Present from Proxmox install; not preferred for main workload VMs |
| `vmdata` | LVM-thin | 1 TB SSD | ~1 TB | Main VM datastore | Primary location for Kubernetes and supporting VM disks |
| `backup-local` | Directory | 256 GB M.2 | ~256 GB | Proxmox backups, restore staging, optional ISOs/snippets | Dedicated local backup storage |

### Storage Design Notes

- `vmdata` is the preferred datastore for:

  - Kubernetes control plane VMs
  - Kubernetes worker VMs
  - `ops1`
  - `svc1`

- `backup-local` is the preferred datastore for:

  - VZDump backups
  - temporary restore operations
  - pre-maintenance backups
  - optional ISO and snippet storage if needed

- `local-lvm` remains available as part of the default Proxmox installation but is not intended to be the primary datastore for planned platform workloads.

## Future Documentation

This section should later include:

- host installation notes
- backup configuration
- maintenance procedures
