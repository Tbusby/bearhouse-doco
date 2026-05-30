# Node Bootstrap

## Overview

Node bootstrap is currently performed using Ansible after VM provisioning through Terraform.

The node bootstrap process prepares freshly provisioned Ubuntu virtual machines for their intended role in the homelab platform.

At the current stage, the primary bootstrap target is the Kubernetes node fleet.

## Bootstrap Sequence

The current intended sequence is:

1. Provision VM with Terraform
2. Apply baseline configuration with Ansible
3. Apply Kubernetes prerequisites with Ansible
4. Install Kubernetes and container runtime components with Ansible
5. Perform later cluster bootstrap tasks separately

## Playbooks

The current playbooks are:

| Playbook | Purpose |
| -------- | ------- |
| `baseline.yml` | Applies baseline system configuration |
| `bootstrap.yml` | Pre-populates SSH `known_hosts` for new servers |
| `k8s_prereqs.yml` | Applies Kubernetes prerequisite system changes |
| `k8s_install.yml` | Installs Kubernetes packages and basic containerd configuration |
| `site.yml` | Runs the full current configuration flow |

## Baseline Configuration

The baseline playbook and role currently handle:

- common package installation
- `qemu-guest-agent` installation and service management
- time and date configuration
- SSH hardening
- root SSH login disablement
- SSH key-only authentication

This establishes a consistent base state for all managed Ubuntu systems.

## Kubernetes Prerequisites

The Kubernetes prerequisites role currently handles:

- prerequisite package installation
- basic Kubernetes host preparation
- swap disablement

These tasks prepare the nodes for later Kubernetes package installation and cluster bootstrap.

## Kubernetes Installation

The Kubernetes installation role currently handles:

- installation of Kubernetes packages
- installation and basic configuration of containerd

This creates a prepared Kubernetes node base without yet fully describing cluster bootstrap and join operations.

## Inventory Targeting

The current inventory defines these groups:

| Group | Purpose |
| ----- | ------- |
| `k8s_control` | Kubernetes control plane nodes |
| `k8s_workers` | Kubernetes worker nodes |
| `k8s_cluster` | Combined Kubernetes host group |

This enables targeted automation patterns such as:

- applying baseline configuration to all nodes
- applying Kubernetes configuration to all cluster nodes
- later applying control-plane-specific tasks only to `k8s_control`

## Current Host Inventory

| Hostname | Role | IP Address |
| -------- | ---- | ---------- |
| `k8s-cp1` | Kubernetes control plane | `192.168.1.21` |
| `k8s-cp2` | Kubernetes control plane | `192.168.1.22` |
| `k8s-cp3` | Kubernetes control plane | `192.168.1.23` |
| `k8s-w1` | Kubernetes worker | `192.168.1.31` |
| `k8s-w2` | Kubernetes worker | `192.168.1.32` |
| `k8s-w3` | Kubernetes worker | `192.168.1.33` |

## Operational Notes

- nodes are provisioned with static IP configuration through Terraform/cloud-init
- Ansible uses the static inventory defined in `inventory/lab/hosts.yml`
- the current bootstrap flow is role-based and intentionally modular
- bootstrap remains host-focused and does not yet include full cluster orchestration

## Future Enhancements

Likely future improvements include:

- explicit control-plane-specific bootstrap logic
- worker join automation
- kubeadm orchestration
- better separation between baseline, runtime, and cluster lifecycle tasks
- validation steps after bootstrap runs
