# ADR-0011: Adopt an NFS-Backed Dynamic StorageClass with a Dedicated Storage VM

- **Status:** Accepted
- **Date:** 2026-07-05

## Context

The homelab Kubernetes cluster now includes several foundational platform services, including ingress, certificate management, and observability. However, the platform still requires a standard persistent storage capability for workloads.

Without a platform storage solution, persistent workloads would require:

- manual PV management
- host-local storage workarounds
- application-specific storage handling
- inconsistent or non-reproducible persistence patterns

This would not align with the platform goals of:

- infrastructure as code
- configuration management
- reproducibility
- realistic platform operations
- production-shaped learning

The platform therefore needs a persistent storage model that can:

- support PVC-driven workload storage
- provide dynamic provisioning
- remain understandable and operationally practical in a single-host homelab
- avoid overcomplicating the current platform stage

## Decision

Adopt an NFS-backed dynamic provisioning model for Kubernetes persistent storage.

The storage design will use:

- a dedicated storage VM
- a separate attached data disk
- an NFS export from that VM
- `nfs-subdir-external-provisioner` inside Kubernetes
- a dynamic `StorageClass` named `nfs-client`

The storage VM will remain separate from both the Proxmox host and the Kubernetes nodes.

## Consequences

### Positive

- Introduces a practical persistent storage capability for workloads
- Provides a standard PVC / PV / StorageClass workflow
- Enables dynamic provisioning without per-application manual PV creation
- Keeps the storage role operationally separate from the hypervisor
- Is significantly simpler to understand and operate than distributed storage platforms
- Fits the current single-host homelab failure-domain reality well
- Creates a useful base for future stateful workloads and persistence improvements

### Negative

- Storage depends on a single NFS server VM and is not highly available
- Current performance and semantics are limited by NFS
- Export permissions and options may be more permissive than ideal in the first lab implementation
- This is not equivalent to a distributed CSI-backed storage system
- Future platform growth may justify re-evaluating more advanced storage options

## Alternatives Considered

- **Use hostPath or local-path style storage**
- **Adopt Longhorn as the first storage platform**
- **Adopt Ceph / Rook-Ceph**
- **Delay persistent storage until later and continue using only stateless workloads**
