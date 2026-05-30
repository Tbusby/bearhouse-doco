# Configuration Management

## Overview

Configuration management for the homelab platform is handled with Ansible.

Ansible is used after VM provisioning to apply repeatable host configuration, establish baseline system state, and prepare Kubernetes nodes for cluster bootstrap.

This separates responsibilities cleanly:

- Proxmox provides virtualization
- Terraform provisions virtual machines
- Ansible configures operating systems
- Kubernetes workload configuration will be managed separately

## Current Scope

Ansible is currently used for:

- baseline Ubuntu host configuration
- common package installation
- SSH hardening
- `qemu-guest-agent` enablement
- time and date configuration
- Kubernetes prerequisite configuration
- swap disablement
- Kubernetes package installation
- basic containerd configuration

## Current Out of Scope

At the current stage, Ansible is not yet responsible for:

- full Kubernetes cluster bootstrap with `kubeadm`
- application deployment into Kubernetes
- GitOps platform deployment
- secrets management
- advanced host hardening beyond current SSH and baseline settings

## Operating Model

Ansible is run from the management environment and targets the statically addressed Ubuntu VMs provisioned by Terraform.

The current host groups are:

- `k8s_control`
- `k8s_workers`
- `k8s_cluster`

This allows common Kubernetes tasks to target all nodes while preserving the ability to target control plane and worker nodes separately.

## Current Responsibilities by Layer

| Layer | Tool | Responsibility |
| ----- | ---- | -------------- |
| Virtualization | Proxmox | Hypervisor and VM hosting |
| Provisioning | Terraform | VM creation and cloud-init configuration |
| Configuration Management | Ansible | Host baseline and Kubernetes node preparation |
| Cluster Runtime | Kubernetes | Container orchestration |
| Future Delivery | Argo CD / CI | Planned later-stage workload delivery |

## Design Notes

The current design follows a practical infrastructure-as-code and configuration-management split:

- Terraform is used for provisioning infrastructure resources
- Ansible is used for mutable guest OS configuration
- host configuration is kept out of Terraform to avoid forcing infra and config concerns into one tool
- Kubernetes node preparation is handled before cluster bootstrap

## Execution Context

Ansible is intended to be run from the management environment against the lab inventory.

This allows:

- central control
- repeatable playbook runs
- clear separation between management tooling and managed nodes
- easier future CI/CD integration

## Current Automation Areas

| Area | Managed by Ansible | Notes |
| ---- | ------------------ | ----- |
| Common packages | Yes | Baseline package set |
| `qemu-guest-agent` | Yes | Installed and enabled |
| SSH hardening | Yes | Root login disabled, key-based auth enforced |
| Time configuration | Yes | Baseline date/time configuration |
| Swap disablement | Yes | Kubernetes prerequisite |
| Kubernetes prerequisites | Yes | Kernel/system preparation |
| Kubernetes packages | Yes | Installed on Kubernetes nodes |
| containerd config | Yes | Basic configuration currently applied |

## Future Direction

Planned future improvements include:

- stronger role separation
- more explicit node bootstrap documentation
- Kubernetes bootstrap automation
- integration with CI/CD validation
- secret handling improvements
- idempotent cluster lifecycle operations
