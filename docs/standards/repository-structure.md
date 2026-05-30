# Repository Structure

## Repositories

### homelab-infra-proxmox

Terraform definitions for VM provisioning and infrastructure resources.

### homelab-ansible

Ansible playbooks and roles for host bootstrap and configuration.

### homelab-k8s-platform

GitOps-managed Kubernetes platform services and core manifests.

### homelab-apps

Application manifests, Helm values, and sample workloads.

### homelab-docs

Architecture, standards, runbooks, ADRs, and operations documentation.

## Principles

- separate infrastructure from workload manifests
- keep documentation in its own repo
- store decisions alongside implementation guidance
- treat repositories as product components, not dumping grounds