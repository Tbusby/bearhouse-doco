# Backup and Recovery Plan

## Purpose

Define what parts of the homelab platform are rebuilt from code, what parts require backup, and how recovery should be approached.

## Scope

Applies to the current homelab platform, including:

- infrastructure and VM roles
- Kubernetes cluster state
- persistent application data
- platform configuration artifacts
- operational documentation
- secrets and key material

## Recovery Strategy

The platform should prefer:

- rebuilding infrastructure from code where practical
- restoring only stateful data that cannot be easily re-created
- using documentation and runbooks as part of recovery
- testing recovery workflows rather than assuming backups are valid

## Recovery Model

| Area | Preferred Recovery Method | Notes |
| ---- | ------------------------- | ----- |
| Terraform-managed infrastructure | Rebuild from code | <https://github.com/Tbusby/bearhouse-infra-proxmox.git> |
| Ansible-managed configuration | Rebuild from code | <https://github.com/Tbusby/bearhouse-infra-ansible.git> |
| Documentation | Restore from Git | <https://github.com/Tbusby/bearhouse-doco.git> |
| GitOps-managed apps | Rebuild from Git | Fill in details |
| Kubernetes cluster state | Fill in | Decide rebuild vs restore |
| Persistent NFS-backed data | Restore from backup | Fill in details |
| Secrets and key material | Restore securely | Fill in details |
| Control plane critical state | Fill in | Decide etcd/PKI recovery model |

## Backup Scope

| Category | Examples | Backup Required | Recovery Method | Notes |
| -------- | -------- | --------------- | --------------- | ----- |
| Code repos | Terraform, Ansible, docs, GitOps repos | Yes | Restore from Git or mirror | |
| Kubernetes resources | Namespaces, manifests, Secrets, CRs | Fill in | Fill in | |
| Persistent data | NFS export data, app data, Grafana, Prometheus | Yes | Restore from backup | |
| Control plane state | etcd, PKI, kubeadm config | Fill in | Fill in | |
| Secrets | age key, SSH keys, tokens, bootstrap secrets | Yes | Restore securely | |

## Backup Methods

| Category | Method | Frequency | Destination | Retention | Notes |
| -------- | ------ | --------- | ----------- | --------- | ----- |
| Code repos | Fill in | Fill in | Fill in | Fill in | |
| NFS-backed data | Fill in | Fill in | Fill in | Fill in | |
| Kubernetes resources | Fill in | Fill in | Fill in | Fill in | |
| etcd snapshots | Fill in | Fill in | Fill in | Fill in | |
| Secrets and keys | Fill in | Fill in | Fill in | Fill in | |

## Recovery Scenarios

| Scenario | Expected Recovery Path | Related Runbook |
| -------- | ---------------------- | --------------- |
| Loss of `ops1` | Rebuild from code | [Rebuild ops](bearhouse-doco/docs/runbooks/003-rebuild-ops.md) |
| Loss of `storage1` | Fill in | Fill in |
| Loss of a worker node | Fill in | Fill in |
| Loss of a control plane node | Fill in | Fill in |
| Loss of a PVC-backed workload | Fill in | Fill in |
| Full cluster rebuild | Fill in | Fill in |
| Loss of critical secrets | Fill in | Fill in |

## Recovery Objectives

| Recovery Item | Target RPO | Target RTO | Notes |
| ------------- | ---------- | ---------- | ----- |
| Code repositories | Fill in | Fill in | |
| `ops1` | Fill in | Fill in | |
| Kubernetes cluster | Fill in | Fill in | |
| NFS-backed data | Fill in | Fill in | |
| Observability stack | Fill in | Fill in | |

## Restore Testing

| Test Date | Scenario | Result | Notes |
| --------- | -------- | ------ | ----- |
| YYYY-MM-DD | Example restore test | Pass / Fail | Example note |

## Known Gaps

- [ ] Fill in known gap
- [ ] Fill in known gap
- [ ] Fill in known gap

## Notes

- Prefer explicit recovery gaps over assumed protection.
- Update this plan when new stateful services are added.
- If a component is fully reproducible from code, record that clearly.
