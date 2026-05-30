# Network Plan

## Overview

The homelab platform currently uses a simple LAN-backed network design.

All Proxmox host management traffic and all virtual machine traffic currently share the same primary LAN via the `vmbr0` bridge. Virtual machines receive their network configuration through Proxmox networking and Ubuntu cloud-init.

At this stage, the environment uses a single primary subnet for:

- Proxmox management
- management VMs
- Kubernetes control plane nodes
- Kubernetes worker nodes

This design keeps the platform simple while the core infrastructure and cluster build are being established.

---

## Current Network Model

| Network Component | Value | Notes |
| ---------------- | ----- | ----- |
| Physical uplink | `enp2s0f0` | Active Proxmox host uplink |
| Secondary NIC | `enp2s0f1` | Present but currently unused |
| Proxmox bridge | `vmbr0` | Main host and VM bridge |
| Primary subnet | `192.168.1.0/24` | Main LAN subnet |
| Default gateway | `192.168.1.1` | Primary router / gateway |
| DNS | `192.168.1.1` | Current default DNS source |
| IP assignment model | Static IPs via cloud-init | VM addresses defined in Terraform |

---

## Proxmox Host Networking

| Interface | Type | Role | Address / CIDR | Gateway | Bridge | Notes |
| --------- | ---- | ---- | -------------- | ------- | ------ | ----- |
| `lo` | Loopback | Local host loopback | `127.0.0.1/8` | None | None | Standard loopback interface |
| `enp2s0f0` | Physical NIC | Uplink for Proxmox bridge | None | None | `vmbr0` | Active physical interface connected to LAN |
| `enp2s0f1` | Physical NIC | Unused / standby | None | None | None | Available for future use |
| `vmbr0` | Linux bridge | Proxmox management and VM network | `192.168.1.10/24` | `192.168.1.1` | N/A | Main bridge for host management and guest connectivity |

---

## Address Allocation Plan

The current IP allocation follows a role-based layout within the `192.168.1.0/24` subnet.

| Range / Address | Purpose | Notes |
| --------------- | ------- | ----- |
| `192.168.1.10` | Proxmox host | Hypervisor management |
| `192.168.1.20` | Reserved | Planned Kubernetes API virtual endpoint |
| `192.168.1.21-23` | Kubernetes control planes | `k8s-cp1` to `k8s-cp3` |
| `192.168.1.31-33` | Kubernetes workers | `k8s-w1` to `k8s-w3` |
| `192.168.1.41` | Management VM | `ops1` |
| `192.168.1.42` | Reserved | Planned `svc1` utility/storage VM |
| `192.168.1.200-210` | Reserved | Planned MetalLB service IP pool |

---

## Current VM Network Inventory

| Hostname | Role | VM ID | IP Address | Network Mode | Bridge | Notes |
| -------- | ---- | ----- | ---------- | ------------ | ------ | ----- |
| `ops1` | Management / automation host | `401` | `192.168.1.41` | Static via cloud-init | `vmbr0` | Terraform, Git, admin tooling |
| `k8s-cp1` | Kubernetes control plane | `201` | `192.168.1.21` | Static via cloud-init | `vmbr0` | First control plane node |
| `k8s-cp2` | Kubernetes control plane | `202` | `192.168.1.22` | Static via cloud-init | `vmbr0` | Second control plane node |
| `k8s-cp3` | Kubernetes control plane | `203` | `192.168.1.23` | Static via cloud-init | `vmbr0` | Third control plane node |
| `k8s-w1` | Kubernetes worker | `301` | `192.168.1.31` | Static via cloud-init | `vmbr0` | First worker node |
| `k8s-w2` | Kubernetes worker | `302` | `192.168.1.32` | Static via cloud-init | `vmbr0` | Second worker node |
| `k8s-w3` | Kubernetes worker | `303` | `192.168.1.33` | Static via cloud-init | `vmbr0` | Third worker node |

---

## Planned but Not Yet Active Addresses

| Hostname / Purpose | Planned IP | Status | Notes |
| ------------------ | ---------- | ------ | ----- |
| `k8s-api` | `192.168.1.20` | Reserved | Planned stable Kubernetes API endpoint |
| `svc1` | `192.168.1.42` | Reserved | Planned utility or storage VM |
| MetalLB pool | `192.168.1.200-210` | Reserved | Planned service exposure range for Kubernetes |

---

## Network Design Notes

- All current workloads are on a single shared L2/L3 LAN segment
- No dedicated storage, replication, or cluster backend network exists at this stage
- Static IP configuration for VMs is applied through cloud-init during Terraform provisioning
- Proxmox itself remains statically addressed on `vmbr0`
- `enp2s0f1` remains unused and available for future isolated network use if required
- The current design prioritizes simplicity and predictability over segmentation

---

## Operational Guidance

- New infrastructure VMs should use the documented address allocation pattern
- New Kubernetes nodes should remain within the defined control plane and worker IP ranges unless the plan is formally revised
- MetalLB addresses should be allocated from the reserved service IP range only
- Future network segmentation should be documented before implementation
- Any change to the primary subnet, gateway, DNS, or bridge configuration should update this document and related Terraform definitions

---

## Future Enhancements

Potential future network improvements include:

- dedicated management VLAN
- dedicated storage or replication network
- internal DNS automation
- Kubernetes API virtual IP implementation
- MetalLB deployment for service exposure
- use of the secondary NIC for isolated lab traffic or storage experiments
