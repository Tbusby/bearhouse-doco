# Networking

## Overview

The platform will initially use a bridged Proxmox network connected to the local LAN.

## Proposed Addressing

| Hostname | IP Address | Purpose |
|----------|------------|---------|
| proxmox01 | 192.168.1.10 | Proxmox host |
| k8s-api | 192.168.1.20 | Kubernetes API endpoint |
| k8s-cp1 | 192.168.1.21 | Control plane |
| k8s-cp2 | 192.168.1.22 | Control plane |
| k8s-cp3 | 192.168.1.23 | Control plane |
| k8s-w1 | 192.168.1.31 | Worker |
| k8s-w2 | 192.168.1.32 | Worker |
| k8s-w3 | 192.168.1.33 | Worker |
| ops1 | 192.168.1.41 | Management |
| svc1 | 192.168.1.42 | Utility / NFS |

## LoadBalancer Pool

Example MetalLB pool:
- `192.168.1.200-192.168.1.210`

## API Endpoint

The Kubernetes API should be reachable through a stable endpoint:
- DNS name: `k8s-api.lab.example`
- IP: reserved virtual IP

## DNS

Internal DNS records should be created for platform services such as:
- `argocd.lab.example`
- `grafana.lab.example`
- `prometheus.lab.example`

## Future Improvements

- dedicated VLANs
- private management network
- DNS automation with external-dns