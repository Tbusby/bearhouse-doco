# Virtual Machines

## VM Inventory

| VM Name | Role | vCPU | RAM | Disk |
|---------|------|------|-----|------|
| k8s-cp1 | Control plane 1 | 2 | 4 GB | 40 GB |
| k8s-cp2 | Control plane 2 | 2 | 4 GB | 40 GB |
| k8s-cp3 | Control plane 3 | 2 | 4 GB | 40 GB |
| k8s-w1 | Worker 1 | 4 | 8 GB | 80 GB |
| k8s-w2 | Worker 2 | 4 | 8 GB | 80 GB |
| k8s-w3 | Worker 3 | 4 | 8 GB | 80 GB |
| ops1 | Management / jump host | 2 | 4 GB | 40 GB |
| svc1 | Utility / NFS | 2 | 4 GB | 100 GB |

## Design Rationale

The VM design separates:
- Kubernetes control plane
- worker capacity
- management tooling
- shared utility services

This supports:
- clearer operational boundaries
- easier troubleshooting
- realistic platform structure
- automation practice

## Provisioning Model

- base VM template created in Proxmox
- VMs provisioned by Terraform
- OS configuration applied by Ansible