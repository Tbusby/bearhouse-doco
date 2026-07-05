# ADR-0007: Adopt MetalLB as the Bare-Metal LoadBalancer Implementation

- **Status:** Accepted
- **Date:** 2026-07-05

## Context

The homelab Kubernetes cluster runs on bare-metal-style infrastructure hosted on Proxmox and connected to a LAN-backed subnet. Unlike managed cloud Kubernetes environments, this cluster does not have a cloud provider integration that can automatically implement Kubernetes Services of type `LoadBalancer`.

Without an additional component, `LoadBalancer` Services would remain unusable or stay in a pending state. This would limit the platform to patterns such as:

- `NodePort` Services
- manually managed reverse proxies
- direct per-node exposure methods

Those approaches would work for basic access, but they are less aligned with the homelab’s platform goals:

- production-shaped design
- realistic operational workflows
- clean service exposure patterns
- reproducibility and documentation
- enabling later ingress and certificate automation

The platform therefore needs a bare-metal load balancer implementation that can:

- assign external IPs to `LoadBalancer` Services
- advertise those IPs on the LAN
- integrate cleanly with the current flat network design
- support future ingress controller exposure

## Decision

Adopt MetalLB as the bare-metal implementation for Kubernetes Services of type `LoadBalancer`.

MetalLB will be:

- installed manually first for learning
- configured using the CRD-based configuration model
- operated initially in Layer 2 mode
- assigned a dedicated service IP pool on the LAN

The initial MetalLB address pool is:

- `192.168.1.100-192.168.1.110`

This pool is reserved for Kubernetes `LoadBalancer` Services and is intentionally kept smaller than the originally considered larger pool to reflect realistic address management and expected ingress-based service sharing.

## Consequences

### Positive

- Enables `LoadBalancer` Services on the bare-metal Kubernetes cluster
- Provides a clean and Kubernetes-native service exposure pattern
- Integrates well with the current single-subnet LAN design
- Supports later deployment of `ingress-nginx`
- Allows most future HTTP/HTTPS applications to share a smaller number of external IPs through ingress
- Uses standard Kubernetes resources and a modern CRD-based configuration model
- Layer 2 mode keeps the initial networking model simple and understandable for learning

### Negative

- Introduces another cluster platform component that must be operated and documented
- Requires careful LAN IP management to avoid overlap with DHCP or static assignments
- Layer 2 mode is simpler but less advanced than a BGP-based routing model
- External service exposure now depends on both Kubernetes service configuration and MetalLB health
- Future growth may require additional pools or a more sophisticated network design

## Alternatives Considered

- **Rely on `NodePort` Services**
- **Expose applications through manually managed reverse proxies outside the cluster**
- **Use a different bare-metal load balancer solution such as kube-vip for service exposure**
- **Implement BGP-based service advertisement from the beginning**
- **Delay load balancer functionality and expose only internal cluster services**
