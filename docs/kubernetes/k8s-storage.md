# NFS StorageClass

## Purpose

Document the Kubernetes-specific implementation details of the NFS-backed dynamic storage layer.

## Overview

The cluster now provides dynamic persistent storage using:

- an NFS server hosted on a dedicated VM
- `nfs-subdir-external-provisioner`
- a Kubernetes `StorageClass`

This allows workloads to request persistent storage through standard PVC workflows.

## Why It Was Added

The cluster needed a production-shaped storage model for persistent workloads.

Before this storage layer:

- workloads requiring persistence would need manual or ad hoc storage handling
- there was no standard dynamic `StorageClass`
- the platform was primarily suited to stateless services

After this storage layer:

- workloads can request storage through PVCs
- volumes are provisioned dynamically
- persistence is available as a platform capability

## Storage Backend

The current backend is:

- NFS served from a dedicated storage VM

Current export path:

- `/srv/nfs/k8s`

## Provisioner

The Kubernetes dynamic provisioner in use is:

- `nfs-subdir-external-provisioner`

This provisioner creates a subdirectory in the NFS export for each claim and presents it to Kubernetes as a dynamically provisioned volume.

## Namespace

The provisioner is installed in:

- `nfs-provisioner`

## StorageClass

Current dynamic StorageClass:

- `nfs-client`

The StorageClass is currently not required to be the cluster default.

## Provisioning Model

Current provisioning flow:

1. workload creates a PVC
2. PVC references `storageClassName: nfs-client`
3. provisioner creates backing directory on NFS export
4. Kubernetes creates a PV
5. PVC binds to the PV
6. pod mounts the resulting volume

## Current StorageClass Characteristics

The current implementation uses:

- dynamic provisioning enabled
- reclaim policy suitable for lab use
- directory-based allocation in the NFS export
- archived delete behavior for safer cleanup in a lab

## Current NFS Server Design

The NFS server is implemented as:

- a dedicated VM
- separate OS and data disks
- mounted export directory for Kubernetes data
- NFS export limited to the lab subnet

This keeps the storage role separate from both the hypervisor and Kubernetes nodes.

## Current Validation Performed

The storage workflow was validated by:

- configuring the NFS server
- confirming the export worked from a client
- installing the external provisioner
- confirming the `StorageClass` existed
- creating a test PVC
- confirming the PVC bound dynamically
- creating a test pod using the PVC
- writing data to the mounted volume
- recreating the pod and confirming data persisted

## Operational Checks

Check the storage provisioner:

```bash
kubectl get pods -n nfs-provisioner
```

Check the `StorageClass`:

```bash
kubectl get storageclass
kubectl describe storageclass nfs-client
```

Check PVCs:

```bash
kubectl get pvc -A
kubectl describe pvc <pvc-name> -n <namespace>
```

Check PVs:

```bash
kubectl get pv
kubectl describe pv <pv-name>
```

Check test pod volume usage:

```bash
kubectl get pod -n <namespace>
kubectl exec -it <pod-name> -n <namespace> -- sh
```

## Good State

The storage layer is considered healthy when:

- the NFS provisioner pod is running
- the `nfs-client` `StorageClass` exists
- PVCs bind automatically
- PVs are created automatically
- pods can mount the provisioned volumes
- data persists across pod recreation

## Risks and Notes

Important notes for this cluster:

- current storage is backed by a single NFS VM
- this is a valid current-state design but not HA storage
- NFS client packages are required on Kubernetes nodes
- current export permissions are intentionally simplified for lab use
- current storage is appropriate for learning and general persistence, but not equivalent to advanced distributed CSI storage

## Next Step

Future storage-related improvements may include:

- making the StorageClass default
- migrating selected platform components to persistent storage
- tightening NFS export options
- evaluating more advanced storage platforms later for learning purposes