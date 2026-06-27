# Kubernetes Cluster Bootstrap Design

## Purpose

This document defines the planned bootstrap approach for the homelab Kubernetes cluster.

The goal of this phase is to move from prepared Ubuntu virtual machines to a functioning multi-node Kubernetes cluster using a production-shaped bootstrap process that is educational, repeatable, and aligned with industry-standard tooling.

The cluster will be bootstrapped manually first to deepen understanding of Kubernetes control plane initialization, node join workflows, networking, and core cluster architecture. Once the process is understood and documented, it can be automated using Ansible.

---

## Goals

The Kubernetes bootstrap process should:

- use industry-standard Kubernetes bootstrap tooling
- provide hands-on understanding of how a multi-node cluster is formed
- support a multi-control-plane architecture for learning HA concepts
- remain realistic for a single-host homelab
- produce a documented and repeatable bootstrap procedure
- prepare the platform for later GitOps, ingress, storage, and observability work

---

## Constraints

The cluster is deployed on a single physical Proxmox host.

Because of this:

- true infrastructure high availability is not possible
- host outage causes full cluster outage
- control plane redundancy is educational and operationally useful, but not physically fault tolerant

This design therefore focuses on:

- correct architectural patterns
- recoverability
- reproducibility
- realistic bootstrap flow

rather than claiming full physical HA.

---

## Cluster Summary

| Item | Value |
| ---- | ----- |
| Bootstrap tool | `kubeadm` |
| Runtime | `containerd` |
| CNI | Planned: `Cilium` |
| Control plane nodes | 3 |
| Worker nodes | 3 |
| Host OS | Ubuntu Server 24.04 LTS |
| Provisioning | Terraform |
| Configuration management | Ansible |
| Initial bootstrap method | Manual |
| Future bootstrap method | Ansible-assisted |

---

## Current Node Inventory

| Hostname | Role | IP Address |
| -------- | ---- | ---------- |
| `k8s-cp1` | Control plane | `192.168.1.21` |
| `k8s-cp2` | Control plane | `192.168.1.22` |
| `k8s-cp3` | Control plane | `192.168.1.23` |
| `k8s-w1` | Worker | `192.168.1.31` |
| `k8s-w2` | Worker | `192.168.1.32` |
| `k8s-w3` | Worker | `192.168.1.33` |

---

## Bootstrap Strategy

The bootstrap approach is intentionally split into two phases:

### Phase 1: Manual bootstrap

The first cluster build will be performed manually in order to learn:

- `kubeadm init`
- cluster certificate and token handling
- control plane join flow
- worker join flow
- CNI installation
- kubeconfig handling
- cluster validation and troubleshooting

### Phase 2: Automated bootstrap

Once the manual process is documented and understood, it will be translated into Ansible-based automation.

This staged approach avoids automating a process before it is well understood.

---

## Architecture Decisions

## Kubernetes Distribution

The cluster will use upstream-style Kubernetes bootstrapped with `kubeadm`.

### Rationale

- closer to real-world Kubernetes operations
- teaches control plane initialization details
- widely used learning and production-adjacent workflow
- aligns with the platform engineering and DevOps learning goals of the lab

---

## Container Runtime

The cluster uses:

- `containerd`

### Rationale

- modern standard Kubernetes runtime
- already installed as part of node preparation
- production-relevant
- appropriate for kubeadm-based clusters

---

## CNI Choice

Planned CNI:

- `Cilium`

### Rationale

- modern and widely adopted
- strong policy support
- production-relevant networking model
- useful for future NetworkPolicy and observability learning

### Note

The exact Cilium installation method will be documented during the implementation phase.

---

## Cluster API Endpoint Strategy

A stable Kubernetes API endpoint is required for a multi-control-plane kubeadm cluster.

### Planned endpoint

| Item | Value |
| ---- | ----- |
| API endpoint name | `k8s-api` |
| Planned IP | `192.168.1.20` |
| Planned role | Stable control plane endpoint |

### Planned implementation

The preferred planned approach is:

- `kube-vip`

### Rationale

- commonly used in kubeadm homelab and bare-metal setups
- appropriate for educational HA control plane scenarios
- avoids introducing a separate dedicated load balancer VM at this stage

### Note

If `kube-vip` proves unnecessarily complex for the first cluster bootstrap, the first bootstrap may temporarily use `k8s-cp1` directly and later evolve to a stable virtual endpoint. If this occurs, the implementation and rationale should be documented.

---

## Network Plan for Kubernetes

### Node network

All nodes currently exist on the primary LAN subnet:

- `192.168.1.0/24`

### Planned API endpoint

- `192.168.1.20`

### Planned MetalLB pool

Reserved for later service exposure:

- `192.168.1.200-210`

### Design note

Kubernetes node traffic currently shares the same bridged LAN as the rest of the lab infrastructure. This is acceptable for the current single-host homelab design.

---

## Planned Kubernetes Network Ranges

These values should be finalized before `kubeadm init`.

### Planned pod CIDR

- `10.244.0.0/16` or another CNI-appropriate range

### Planned service CIDR

- `10.96.0.0/12`

### Design note

The final pod CIDR must match the chosen CNI installation parameters. This value should be confirmed before initializing the cluster.

---

## Planned Bootstrap Order

The planned bootstrap order is:

1. Confirm all nodes are reachable and correctly prepared
2. Confirm required packages and runtime are installed
3. Prepare stable control plane endpoint strategy
4. Initialize first control plane on `k8s-cp1`
5. Configure `kubectl` access on the bootstrap node
6. Install CNI
7. Join additional control planes:
   - `k8s-cp2`
   - `k8s-cp3`
8. Join worker nodes:
   - `k8s-w1`
   - `k8s-w2`
   - `k8s-w3`
9. Validate cluster node health
10. Validate DNS, networking, and scheduling

---

## Bootstrap Responsibilities

| Task | Tool |
| ---- | ---- |
| VM provisioning | Terraform |
| OS preparation | Ansible |
| Cluster initialization | Manual (initially) |
| Future repeat automation | Ansible |
| Workload deployment | Later GitOps / Kubernetes manifests |

---

## Manual Bootstrap Scope

The first manual bootstrap should include:

- preparing the `kubeadm init` configuration
- running `kubeadm init` on `k8s-cp1`
- configuring `kubectl`
- generating and using join commands
- joining additional control plane nodes
- joining worker nodes
- installing and validating CNI
- testing the basic cluster state

### Explicit non-goals for the first bootstrap

The first bootstrap does not need to include:

- ingress
- MetalLB
- storage provisioner
- monitoring stack
- GitOps platform
- advanced policy or admission control

These should follow after the base cluster is healthy.

---

## Validation Criteria

The bootstrap will be considered successful when:

- all 3 control plane nodes join successfully
- all 3 worker nodes join successfully
- all nodes report `Ready`
- the CNI is healthy
- cluster DNS functions correctly
- pods can schedule to worker nodes
- `kubectl get nodes` and `kubectl get pods -A` show expected healthy state

---

## Risks

| Risk | Impact | Mitigation |
| ---- | ------ | ---------- |
| Incorrect API endpoint design | Control plane join failures | Finalize endpoint design before init |
| Pod CIDR mismatch with CNI | Broken networking | Validate CNI requirements before bootstrap |
| Incomplete node preparation | kubeadm failures | Validate Ansible prerequisites before bootstrap |
| Manual bootstrap errors | Delays or partial cluster state | Document commands carefully and proceed in order |
| Single host outage | Full outage | Accept as design constraint; rely on rebuildability |
| Certificate/join token confusion | Join failures | Capture and document commands during bootstrap |

---

## Documentation Requirements

During the first manual bootstrap, the following should be recorded:

- exact `kubeadm init` command or config file used
- chosen API endpoint method
- chosen pod and service CIDRs
- CNI installation command or manifest source
- exact join commands used
- any issues encountered and their resolutions
- final validation commands and outputs

This documentation will form the basis of later Ansible automation.

---

## Future Automation Plan

After the manual bootstrap is complete and validated, the process should be translated into Ansible automation.

Likely future automation areas include:

- rendering `kubeadm` config files
- running `kubeadm init`
- distributing kubeconfig
- generating and executing join commands
- installing CNI
- post-bootstrap validation tasks

The initial automation should aim for:

- clarity
- repeatability
- minimal hidden logic

---

## Future Platform Steps After Bootstrap

Once the cluster is healthy, the likely next platform phases are:

1. install CNI if not already complete
2. install metrics-server
3. install MetalLB
4. install ingress-nginx
5. install cert-manager
6. install storage provisioner
7. install observability stack
8. install Argo CD

---

## Open Decisions

The following items should be finalized before the first cluster bootstrap:

- final API endpoint implementation (`kube-vip` or temporary direct bootstrap)
- final pod CIDR
- final service CIDR
- exact Cilium installation method
- whether initial `kubectl` administration lives temporarily on `k8s-cp1`, `ops1`, or both

---

## Success Criteria

This design will be considered successfully implemented when:

- the cluster can be bootstrapped manually from the documented procedure
- the resulting cluster is healthy and usable
- the bootstrap process is captured clearly enough to automate later with Ansible
- the design remains aligned with the intended production-shaped learning goals of the homelab

---

## Related Documents

- [Cluster Design](../kubernetes/cluster-design.md)
- [Node Bootstrap](../operations/node-bootstrap.md)
- [Configuration Management](../operations/configuration-management.md)
- [Network Plan](../infrastructure/networking.md)
- [Run Ansible Configuration Management](../runbooks/run-ansible-configuration-management.md)
