
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

## Future Documentation

This section should later include:
- host installation notes
- storage configuration
- network bridge configuration
- backup configuration
- maintenance procedures