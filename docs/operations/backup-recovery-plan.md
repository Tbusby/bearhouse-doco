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
| Terraform-managed infrastructure | Rebuild from code | Proxmox VMs should be recreated from the Terraform repo: `bearhouse-infra-proxmox` |
| Ansible-managed configuration | Rebuild from code | Host and service configuration should be re-applied from the Ansible repo: `bearhouse-infra-ansible` |
| Documentation | Restore from Git | MkDocs documentation is authoritative in Git: `bearhouse-doco` |
| GitOps-managed apps | Rebuild from Git | Argo CD app state should increasingly be rehydrated from Git once more workloads are managed there |
| Kubernetes cluster state | Rebuild from code and documented bootstrap process | Preferred model is rebuild with Terraform, Ansible, and kubeadm-based cluster recreation rather than etcd restore |
| Persistent NFS-backed data | Restore from backup | NFS-backed application data is the highest-priority persistent data to preserve |
| Secrets and key material | Restore securely | SOPS/age keys, automation SSH keys, and local credential material must be recoverable from trusted systems |
| Control plane critical state | Rebuild from documented kubeadm process | etcd snapshots are not currently used; current preference is rebuild rather than snapshot restore |

## Backup Scope

| Category | Examples | Backup Required | Recovery Method | Notes |
| -------- | -------- | --------------- | --------------- | ----- |
| Code repos | Terraform, Ansible, docs, future GitOps repos | Yes | Restore from Git | GitHub is assumed to survive and is treated as trusted remote source of truth |
| Kubernetes resources | Namespaces, manifests, Secrets, CRs | Limited for now | Rebuild from Git / code where possible | Current preference is rebuild rather than full cluster-state restore |
| Persistent data | NFS export data, future app data, Grafana state if desired | Yes | Restore from backup | This is the most important current backup area |
| Control plane state | etcd, PKI, kubeadm config | Partial / TBD | Rebuild from documented process | etcd snapshot workflow is not yet implemented |
| Secrets | age key, SSH keys, tokens, bootstrap secrets | Yes | Restore securely | Some secrets are recoverable from SOPS, others rely on trusted machine copies |

## Backup Methods

| Category | Method | Frequency | Destination | Retention | Notes |
| -------- | ------ | --------- | ----------- | --------- | ----- |
| Code repos | GitHub plus local clones | Continuous via Git workflow | GitHub, laptop, `ops1` | Long-term | Local clones may drift from remote if not updated |
| NFS-backed data | Planned simple copy / rsync-style workflow | TBD | AWS S3 | TBD | Current intended backup target is AWS S3; exact implementation still to be built |
| Kubernetes resources | Not yet implemented | N/A | N/A | N/A | Current recovery preference is rebuild rather than resource backup |
| etcd snapshots | Not implemented | N/A | N/A | N/A | Not currently part of the recovery model |
| Secrets and keys | Trusted system copies and SOPS-encrypted storage where applicable | As updated | Laptop, `ops1`, Git for encrypted key material | Long-term | Proxmox API credentials are stored outside Git in ignored tfvars files on trusted systems |

## Recovery Scenarios

| Scenario | Expected Recovery Path | Related Runbook |
| -------- | ---------------------- | --------------- |
| Loss of `ops1` | Rebuild VM from Terraform, reapply Ansible, restore required secrets and tooling access | `ops1` rebuild runbook |
| Loss of `storage1` | Rebuild VM, reapply Ansible NFS configuration, restore NFS-backed application data from backup | Storage runbook / future NFS recovery runbook |
| Loss of a worker node | Rebuild node from Terraform and Ansible, rejoin cluster, allow workloads to reschedule | Kubernetes bootstrap / node rebuild runbook |
| Loss of a control plane node | Rebuild node and rejoin control plane using documented kubeadm process | Kubernetes bootstrap / control-plane recovery runbook |
| Loss of a PVC-backed workload | Recreate workload from code or GitOps and restore NFS-backed application data if required | Storage runbook / future workload restore runbook |
| Full cluster rebuild | Recreate VMs, rebuild cluster with Terraform + Ansible + kubeadm process, restore NFS application data as needed | Kubernetes bootstrap runbook plus platform rebuild documentation |
| Loss of critical secrets | Restore from SOPS where applicable or from trusted machine copies | SOPS / secrets management docs |

## Recovery Objectives

| Recovery Item | Target RPO | Target RTO | Notes |
| ------------- | ---------- | ---------- | ----- |
| Code repositories | Very low | Low | GitHub is primary trusted source of truth |
| `ops1` | 24h | 1h | `ops1` should remain highly reproducible and not store critical irreplaceable data |
| Kubernetes cluster | 1h | 1h | Rebuild-focused model; app data loss should be minimized through NFS backup strategy |
| NFS-backed data | 1h | 1h | Most important current recovery target |
| Observability stack | Low priority for data | Medium | Prometheus history may be lost if necessary; Grafana state is less critical if reproducible from code |

## Restore Testing

| Test Date | Scenario | Result | Notes |
| --------- | -------- | ------ | ----- |
| Existing completed test | `ops1` rebuild from alternate control machine | Pass | Confirms `ops1` is reproducible and not a snowflake |
| YYYY-MM-DD | NFS-backed data restore test | Planned | Should validate restore of application data from backup target |
| YYYY-MM-DD | Full cluster rebuild using Terraform + Ansible + kubeadm process | Planned | Should validate rebuild-based recovery model end to end |
| YYYY-MM-DD | Secret recovery validation | Planned | Should confirm required keys and credentials are recoverable from trusted systems |

## Known Gaps

- [ ] NFS-backed data backup workflow to AWS S3 is not yet implemented
- [ ] Kubernetes-aware backup tooling is not implemented because current preference is rebuild over in-cluster restore
- [ ] etcd snapshot workflow is not implemented
- [ ] kubeadm cluster bootstrap and join process should be automated further in Ansible
- [ ] Secret recovery should be reviewed to ensure all critical material exists on more than one trusted system
- [ ] Consider storing critical recovery keys on an additional offline medium such as encrypted external USB storage

## Related Runbooks

- Proxmox recovery runbook
- `ops1` rebuild runbook
- Kubernetes bootstrap runbook
- Storage runbook
- Future backup/restore runbooks

## Notes

- Current plan reflects current implemented reality rather than future aspirational tooling.
- The platform currently prefers rebuild from Terraform, Ansible, and kubeadm rather than restore from full cluster snapshots.
- User-facing application data is expected to become the most important backup target as more real workloads are added.
- Future GitOps repo design will make application recovery cleaner as more workloads move under Argo CD management.
