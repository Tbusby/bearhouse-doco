# ADR-0002: Use kubeadm for Kubernetes Bootstrap

- **Status:** Accepted
- **Date:** 2026-05-23

## Context

The platform should prioritize learning industry-standard Kubernetes concepts and operational patterns.

## Decision

Use kubeadm to bootstrap the Kubernetes cluster.

## Consequences

### Positive
- closer to upstream Kubernetes
- better learning value for control plane architecture
- teaches foundational Kubernetes setup concepts

### Negative
- more operational complexity than lightweight distributions
- requires more setup and maintenance effort

## Alternatives Considered

- k3s
- MicroK8s