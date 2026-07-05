# Install NFS-Backed StorageClass

## Purpose

Install and validate an NFS-backed dynamic storage layer for Kubernetes using a dedicated storage VM and `nfs-subdir-external-provisioner`.

This runbook is intended for the initial implementation and validation of the storage platform layer.

## Scope

Applies to the current kubeadm-based homelab Kubernetes cluster.

This runbook covers:

- provisioning and preparing a dedicated storage VM
- configuring NFS export storage
- validating NFS from a client
- ensuring Kubernetes nodes can use NFS
- installing the NFS external provisioner
- validating dynamic PVC provisioning

This runbook does not cover:

- high availability storage
- Longhorn or Ceph
- public object storage
- automated backup design

## Prerequisites

- Cluster is healthy and all nodes are `Ready`
- `ops1` has working cluster admin access
- Kubernetes nodes can be updated through Ansible
- Dedicated storage VM exists or can be provisioned
- Dedicated storage data disk exists
- The lab subnet and storage VM IP are known
- User understands this is a simple NFS-based first storage implementation

## Planned Values

| Item | Value |
| ---- | ----- |
| Storage VM | `storage1` |
| Export path | `/srv/nfs/k8s` |
| Provisioner namespace | `nfs-provisioner` |
| StorageClass | `nfs-client` |
| Provisioner | `nfs-subdir-external-provisioner` |
| Export network | `192.168.1.0/24` |

## Risks

- targeting the wrong disk during initial disk preparation
- overlapping NFS responsibilities with the Proxmox host
- proceeding to Kubernetes before validating the NFS export from a client
- forgetting that NFS client packages are required on cluster nodes
- assuming the first implementation is highly available when it is not

## Procedure

### 1. Provision the storage VM

- [ ] Create a dedicated Ubuntu storage VM
- [ ] Assign a static IP
- [ ] Attach a dedicated data disk for NFS-backed storage
- [ ] Apply baseline host configuration with Ansible

### 2. Prepare the data disk

- [ ] Identify the dedicated data disk correctly
- [ ] Create a partition table and data partition
- [ ] Create the filesystem
- [ ] Mount the data disk at:

  ```text
  /srv/nfs/k8s
  ```

- [ ] Ensure the mount persists across reboot

### 3. Configure the NFS server

- [ ] Install NFS server packages
- [ ] Set export directory ownership and permissions
- [ ] Configure `/etc/exports`
- [ ] Enable and start `nfs-kernel-server`
- [ ] Reload exports

### 4. Validate the NFS export from a client

- [ ] Install NFS client tools on a Linux client such as `ops1`
- [ ] Mount the export temporarily
- [ ] Confirm files can be written and listed
- [ ] Unmount the test mount
- [ ] Do not proceed until client-side NFS validation works

### 5. Ensure Kubernetes nodes can use NFS

- [ ] Confirm `nfs-common` is installed on Kubernetes nodes
- [ ] Validate node-level NFS client capability through configuration management

### 6. Install the NFS provisioner Helm repo

- [ ] Add the repo:

  ```bash
  helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
  helm repo update
  ```

### 7. Create the provisioner namespace

- [ ] Create the namespace:

  ```bash
  kubectl create namespace nfs-provisioner
  ```

### 8. Create provisioner values

- [ ] Create a values file with the NFS server IP and export path
- [ ] Confirm the `StorageClass` name is:

  ```text
  nfs-client
  ```

- [ ] Confirm default-class behavior is set intentionally
- [ ] Confirm reclaim and archive behavior are set intentionally

### 9. Install the provisioner

- [ ] Install the Helm release:

  ```bash
  helm install nfs-subdir-external-provisioner \
    nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --namespace nfs-provisioner \
    -f values.yaml
  ```

### 10. Validate the provisioner

- [ ] Check provisioner pod health:

  ```bash
  kubectl get pods -n nfs-provisioner
  ```

- [ ] Check the `StorageClass`:

  ```bash
  kubectl get storageclass
  kubectl describe storageclass nfs-client
  ```

### 11. Validate dynamic PVC provisioning

- [ ] Create a test PVC that uses `storageClassName: nfs-client`
- [ ] Confirm the PVC becomes `Bound`
- [ ] Confirm a PV is created automatically

### 12. Validate persistent pod storage

- [ ] Create a test pod using the test PVC
- [ ] Write a file to the mounted volume
- [ ] Delete and recreate the pod
- [ ] Confirm the file still exists

### 13. Record the final state

- [ ] Record the storage VM details
- [ ] Record the export path
- [ ] Record the `StorageClass` name
- [ ] Record whether the StorageClass is default
- [ ] Record the validation results
- [ ] Update architecture and Kubernetes documentation

## Validation

The installation is successful when:

- the storage VM exports NFS correctly
- clients can mount the export
- Kubernetes nodes can use NFS
- the NFS provisioner pod is healthy
- the `nfs-client` `StorageClass` exists
- PVCs bind dynamically
- a test pod can write and persist data

## Rollback / Fallback

- Remove the test PVC and test pod if only validation cleanup is needed
- Uninstall the NFS provisioner Helm release if the Kubernetes-side storage layer must be removed
- Remove or disable the NFS export if the storage server must be rolled back
- Do not treat failed or partial NFS validation as acceptable before using the StorageClass for real workloads

## Follow-Up

- Decide whether to make `nfs-client` the default StorageClass
- Migrate selected workloads to persistent storage where appropriate
- Tighten export permissions and options if desired
- Add automation and lifecycle management for the provisioner if not already done
- Consider future backup strategy for NFS-backed workload data

## Notes

- This is a valid and practical first persistent storage implementation for the lab
- The current design is not highly available
- Dynamic provisioning is the main capability gained here
- Current export permissions and options should be documented as homelab-friendly simplifications
