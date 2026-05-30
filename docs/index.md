# Homelab Platform Docs

Welcome to the documentation for the Bearhouse homelab.

This repository documents a production-shaped homelab environment designed to support learning and demonstration of modern DevOps, SRE, and platform engineering practices.

## Objectives

- Build a useful, well-structured homelab platform
- Use industry-standard tools where practical
- Treat the environment as production-like
- Fully automate provisioning, configuration, and deployment
- Use Git as the source of truth
- Document architecture, standards, and operations
- Implement backup and restore procedures
- Learn modern tooling for devops

## Core Technologies

- Proxmox VE
- Ubuntu Server
- Kubernetes via kubeadm
- containerd
- Cilium
- MetalLB
- ingress-nginx
- Terraform
- Ansible
- GitHub Actions
- Argo CD
- SOPS + age
- Prometheus / Grafana / Alertmanager
- Loki
- restic
- AWS s3

## Mission Statement

> This homelab provides a production-shaped, fully documented, infrastructure-as-code-driven Kubernetes platform used to learn and demonstrate modern DevOps, SRE, and platform engineering practices using industry-standard tools.

## Documentation Map

- [Architecture Overview](architecture/overview.md)
- [Infrastructure](infrastructure/proxmox.md)
- [Kubernetes](kubernetes/cluster-design.md)
- [Operations](operations/backup-recovery.md)
- [Standards](standards/naming-standards.md)
- [Architecture Decisions](adr/index.md)
