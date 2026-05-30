# ADR-0001: Use Proxmox as Hypervisor

- **Status:** Accepted
- **Date:** 2026-05-23

## Context

The platform requires a dedicated virtualization layer to host Kubernetes nodes and supporting VMs.

## Decision

Use Proxmox VE as the bare-metal hypervisor.

## Consequences

### Positive

- mature and widely used in homelabs
- supports templates, snapshots, and automation workflows
- suitable for dedicated single-host lab environments

### Negative

- adds a virtualization layer to manage
- not identical to all enterprise virtualization stacks

## Alternatives Considered

- VMware ESXi
- XCP-ng
- VirtualBox on a desktop OS
