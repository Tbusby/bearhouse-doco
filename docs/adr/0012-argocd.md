# ADR-0012: Adopt Argo CD as the Initial GitOps Controller for Kubernetes Applications

- **Status:** Accepted
- **Date:** 2026-07-08

## Context

The homelab Kubernetes platform now includes core runtime and platform services such as ingress, certificate management, observability, and persistent storage. The next architectural need is a more production-shaped application delivery and reconciliation model.

Before GitOps, the operational model for Kubernetes applications and add-ons was primarily:

- manual installation for learning
- validation in-cluster
- documentation after the fact
- later planned automation

This is effective for learning individual components but does not by itself provide a strong steady-state model for:

- desired state reconciliation
- drift detection
- Git-based application ownership
- application delivery workflows inside the cluster

The platform therefore needs a Kubernetes-native GitOps controller that can:

- read desired state from Git
- apply application resources to the cluster
- detect drift between Git and live state
- support incremental adoption without requiring immediate platform-wide conversion

## Decision

Adopt Argo CD as the initial GitOps controller for Kubernetes applications in the homelab platform.

Argo CD will be:

- installed manually first for learning
- exposed through the existing ingress and internal TLS pattern
- used initially for a small demo application to validate the GitOps workflow
- adopted incrementally rather than taking immediate ownership of all platform components

The first GitOps-managed application will be a simple demo workload so that sync, health, and drift behavior can be understood safely.

## Consequences

### Positive

- Introduces a Git-based source of truth for Kubernetes application state
- Provides visible sync and health reporting
- Supports drift detection and future reconciliation workflows
- Fits well with the existing ingress and certificate platform patterns
- Allows incremental adoption rather than forcing immediate platform-wide migration
- Provides strong learning value for GitOps operating models
- Complements Terraform and Ansible rather than replacing them

### Negative

- Adds another platform control plane component that must be operated and documented
- Introduces new ownership boundaries between manually managed and GitOps-managed resources
- Can create conflicts with pre-existing manually created resources if ownership is not clarified
- Requires careful repository and application structure decisions over time
- Initial bootstrap credentials and ingress exposure must be handled securely

## Alternatives Considered

- **Continue managing Kubernetes applications manually without GitOps**
- **Adopt Flux instead of Argo CD**
- **Delay GitOps until more applications exist**
- **Attempt to GitOps-convert the entire platform immediately**
