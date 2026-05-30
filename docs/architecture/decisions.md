# Architecture Decisions Summary

This page summarizes the major architecture decisions for the platform.

## Core Decisions

- Proxmox VE will be used as the hypervisor
- Ubuntu Server 24.04 LTS will be used for VMs
- Kubernetes will be bootstrapped with kubeadm
- containerd will be used as the Kubernetes runtime
- Cilium will be used as the CNI
- MetalLB will provide LoadBalancer support
- ingress-nginx will be used as the ingress controller
- Terraform will provision infrastructure
- Ansible will bootstrap and configure systems
- Argo CD will manage GitOps-based deployments
- GitHub Actions will provide CI pipelines
- SOPS + age will be used for secrets encryption
- NFS will be used for initial persistent storage
- restic will back up critical data to AWS S3

## ADR References

- [ADR-0001: Use Proxmox as hypervisor](../adr/0001-use-proxmox-as-hypervisor.md)
- [ADR-0002: Use kubeadm for Kubernetes bootstrap](../adr/0002-use-kubeadm-for-kubernetes-bootstrap.md)
- [ADR-0003: Use Cilium as CNI](../adr/0003-use-cilium-as-cni.md)
- [ADR-0004: Use NFS for initial persistent storage](../adr/0004-use-nfs-for-initial-persistent-storage.md)
- [ADR-0005: Use GitHub Actions and Argo CD for delivery](../adr/0005-use-github-actions-and-argocd-for-delivery.md)
- [ADR-0006: Use SOPS and age for secrets](../adr/0006-use-sops-and-age-for-secrets.md)
