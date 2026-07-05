# Kubernetes Storage

## Purpose

Describe the role of persistent storage in the homelab platform architecture and explain why an NFS-backed dynamic provisioning model was introduced.

## Summary

The platform now includes a Kubernetes persistent storage layer built on:

- a dedicated storage VM
- an NFS export
- a Kubernetes NFS external provisioner
- a `StorageClass` for dynamic provisioning

This provides a practical and production-shaped storage model for the current homelab stage.

## Why Storage Was Added

The platform already had a strong set of core services:

- working Kubernetes cluster
- `metrics-server`
- MetalLB
- `ingress-nginx`
- `cert-manager`
- observability stack

However, persistent workload storage was still missing as a first-class platform capability.

Without a storage layer, stateful workloads would require ad hoc approaches such as:

- manual PV creation
- node-local storage workarounds
- hostPath usage
- application-specific storage hacks

That would work for limited testing, but it would not align with the platform goals of:

- reproducibility
- realistic operational workflows
- platform engineering learning
- clean workload abstractions

## Architectural Role

The storage layer provides Kubernetes-native persistent storage through:

- `PersistentVolumeClaim`
- `PersistentVolume`
- `StorageClass`

Workloads can now request persistent storage declaratively, and the storage backing is provisioned dynamically.

This gives the platform a storage model suitable for:

- persistent dashboards
- databases
- stateful applications
- future platform services requiring durable volumes

## Chosen Design

The current storage design uses:

- a dedicated Ubuntu storage VM
- a separate attached data disk
- an NFS export from that VM
- `nfs-subdir-external-provisioner` inside Kubernetes
- a dynamic `StorageClass`

This creates a simple but realistic workflow:

1. a workload requests a PVC
2. the NFS provisioner receives the request
3. a backing directory is created in the NFS export
4. a PV is created and bound
5. the pod mounts the resulting storage

## Why NFS Was Chosen

NFS was chosen because it is a strong fit for the current homelab environment.

Benefits:

- simple to understand
- easy to validate end to end
- practical for a single-host lab
- good learning value for PV/PVC/StorageClass concepts
- much lower operational complexity than distributed storage systems

This choice intentionally favors clarity and operability over maximum sophistication.

## Why a Dedicated Storage VM Was Chosen

The storage service is hosted on a dedicated VM rather than:

- the Proxmox host
- a Kubernetes node

This was chosen to preserve platform separation of concerns.

Benefits:

- cleaner architecture
- better reproducibility
- easier documentation and lifecycle management
- better alignment with the principle of keeping the Proxmox host focused on virtualization

## Why Other Options Were Not Chosen

Other storage options were considered, but not chosen at this stage.

### Longhorn

Longhorn is a useful Kubernetes-native storage system, but it introduces more complexity and distributed storage concepts than are necessary for the current phase of the lab.

Because the cluster runs on VMs hosted by a single physical machine, the failure-domain benefits of distributed storage would also be limited.

### hostPath / local-path style storage

These approaches are simple, but they are less production-shaped and can create undesirable coupling between workloads and individual nodes.

### Ceph / Rook-Ceph

This would add significant complexity and is not a good fit for the current single-host homelab stage.

## Current Storage Model

The current storage model includes:

- dedicated storage VM: `storage1`
- dedicated data disk mounted for Kubernetes storage
- NFS export path: `/srv/nfs/k8s`
- Kubernetes provisioner namespace: `nfs-provisioner`
- dynamic StorageClass: `nfs-client`

## Dynamic Provisioning Model

Dynamic provisioning is enabled through `nfs-subdir-external-provisioner`.

When a workload requests storage using the `nfs-client` `StorageClass`:

- a subdirectory is created inside the NFS export
- a PV is created automatically
- the PVC is bound automatically
- the pod receives mounted storage without manual PV creation

This is a major improvement over manually managing per-application volumes.

## Access Model

The current storage backend is file-based shared storage over NFS.

This is well suited to:

- general persistent application data
- simple stateful workloads
- shared file-style access patterns

It is not intended to behave like a fully distributed block storage system.

## Current Validated State

The storage layer is installed and functioning.

Validated outcomes include:

- `storage1` is provisioned and configured
- the NFS export is active
- Kubernetes nodes can use the export
- the NFS provisioner is running
- the `nfs-client` `StorageClass` exists
- PVCs can be created and bound dynamically
- a test pod can write data to a PVC
- data persists across pod recreation

## Risks and Limitations

The current design has known limitations:

- single NFS server
- no storage high availability
- underlying failure domain still depends on one physical host
- NFS performance and semantics are simpler than more advanced storage systems

These are acceptable tradeoffs for the current homelab stage.

## Homelab Shortcuts

The current NFS implementation includes some lab-friendly simplifications, such as:

- permissive export directory permissions
- `no_root_squash` in export options
- single-server storage backend

These are documented as deliberate shortcuts and should not be treated as the ideal end-state for more security-sensitive environments.

## Architectural Benefits

Adding this storage layer gives the platform:

- a real persistence model for workloads
- dynamic provisioning through `StorageClass`
- a clean foundation for stateful applications
- the ability to improve persistence for existing platform services later
- a better platform base for future GitOps and application delivery workflows

## Future Direction

Possible future improvements include:

- making `nfs-client` the default `StorageClass` if desired
- tightening NFS export permissions and options
- moving selected platform components to persistent volumes
- revisiting more advanced storage systems later for comparison and learning
