# ADR-0008: Adopt ingress-nginx as the Initial Kubernetes Ingress Controller

- **Status:** Accepted
- **Date:** 2026-07-05

## Context

The homelab Kubernetes cluster now has a working bare-metal `LoadBalancer` implementation through MetalLB. This enables the cluster to expose Services on the LAN using stable external IP addresses.

The next platform need is a shared HTTP/HTTPS entry point for applications running in the cluster. Exposing each application individually with its own `LoadBalancer` Service would work, but would not be the preferred long-term platform pattern because it would:

- consume more IP addresses than necessary
- create less consistent application exposure
- make host-based and path-based routing harder to manage
- complicate future TLS automation

A Kubernetes ingress controller is therefore needed to provide:

- shared L7 HTTP/HTTPS routing
- hostname-based routing
- path-based routing
- a clean future integration point for `cert-manager`

The ingress solution should fit the current goals of the homelab:

- production-shaped learning
- infrastructure as code
- reproducibility
- documentation
- realistic operational workflows
- manual-first learning before automation

## Decision

Adopt `ingress-nginx` as the initial Kubernetes ingress controller for the homelab platform.

`ingress-nginx` will be:

- installed manually first for learning
- exposed through a MetalLB-backed `LoadBalancer` Service
- used as the shared HTTP/HTTPS entry point for cluster applications
- configured using standard Kubernetes `Ingress` resources

This will be the default ingress solution for the current phase of the platform unless future requirements justify a change.

## Consequences

### Positive

- Provides a widely used and well-documented ingress controller
- Supports standard Kubernetes `Ingress` resources
- Enables multiple web applications to share a single external IP
- Integrates cleanly with MetalLB for bare-metal service exposure
- Creates a strong foundation for later `cert-manager` integration
- Aligns well with learning goals because the controller behavior and routing model are easy to observe directly
- Keeps application exposure closer to real-world Kubernetes platform patterns than relying on `NodePort` or one `LoadBalancer` per app

### Negative

- Adds another platform component that must be operated and documented
- Introduces ingress-specific concepts such as ingress classes, host-based routing, and controller behavior
- Requires hostname resolution for realistic testing, even if initially handled with `/etc/hosts`
- May later require migration or re-evaluation if more advanced ingress or gateway requirements emerge
- Using the upstream static manifest manually first is good for learning but is not the final automated lifecycle approach

## Alternatives Considered

- **Expose applications directly with `LoadBalancer` Services only**
- **Expose applications with `NodePort` Services**
- **Use a different ingress controller such as Traefik**
- **Delay ingress and wait until DNS/TLS is fully designed**
