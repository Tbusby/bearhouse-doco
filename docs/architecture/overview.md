# Architecture Overview

## Purpose

This document describes the target-state architecture for the homelab platform.

The platform is designed to provide a realistic, production-shaped learning environment for DevOps and platform engineering workflows.

## High-Level Design

The platform consists of the following layers:

1. Physical hardware
2. Proxmox virtualization
3. Ubuntu virtual machines
4. Kubernetes cluster
5. Platform services
6. CI/CD, GitOps, backup, and operations tooling

## Key Characteristics

- Single physical host
- Multi-VM architecture
- Kubernetes as the primary application platform
- Infrastructure as code
- Configuration as code
- GitOps-based deployment model
- Offsite backups to AWS S3
- Centralized monitoring and logging
- Documentation-first approach

## Constraints

Because the platform runs on a single physical host:
- true infrastructure HA is not possible
- host failure causes full outage
- resilience is achieved through reproducibility, automation, and backups

## Selected Technologies

| Category | Technology |
|----------|------------|
| Hypervisor | Proxmox VE |
| Guest OS | Ubuntu Server 24.04 LTS |
| Kubernetes bootstrap | kubeadm |
| Runtime | containerd |
| CNI | Cilium |
| Load balancer | MetalLB |
| Ingress | ingress-nginx |
| IaC | Terraform |
| Config management | Ansible |
| CI | GitHub Actions |
| CD / GitOps | Argo CD |
| Secrets | SOPS + age |
| Monitoring | Prometheus, Grafana, Alertmanager |
| Logging | Loki |
| Backup | restic + AWS S3 |
| Documentation | MkDocs Material |

## Related Documents

- [Principles](principles.md)
- [Decisions Summary](decisions.md)
- [Diagrams](diagrams.md)