# Kubernetes Cluster Design

## Cluster Type

- kubeadm-managed Kubernetes cluster
- three control plane nodes
- three worker nodes

## Runtime

- containerd

## Networking

- Cilium CNI
- NetworkPolicies enabled
- MetalLB for LoadBalancer services
- ingress-nginx for ingress

## Design Goals

- production-shaped architecture
- reproducible bootstrap
- GitOps-managed workloads
- clear separation between infrastructure and workloads

## Availability Model

This cluster is designed to teach control plane redundancy and scheduling across multiple nodes.

Because the environment runs on a single physical host, it is not physically HA.

## Bootstrap Approach

Cluster bootstrap should be:

- documented
- repeatable
- partially or fully automated using Ansible where practical

## Future Enhancements

- kube-vip for API endpoint management
- admission policies
- workload identity experiments
