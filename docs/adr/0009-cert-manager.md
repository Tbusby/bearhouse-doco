# ADR-0009: Adopt cert-manager for Kubernetes Certificate Lifecycle Management

- **Status:** Accepted
- **Date:** 2026-07-05

## Context

The homelab Kubernetes cluster now has a working service exposure path through MetalLB and `ingress-nginx`. Applications can be reached through a shared ingress entry point, but the platform still requires a repeatable and Kubernetes-native way to manage TLS certificates.

Without a certificate lifecycle controller, certificate handling would require:

- manual certificate generation
- manual TLS Secret creation
- manual renewal and replacement
- additional operational drift risk

This would be inconsistent with the platform goals of:

- infrastructure as code
- reproducibility
- realistic operational workflows
- secure secret handling
- documentation-driven operations

The platform therefore needs a certificate management solution that can:

- issue certificates declaratively
- manage TLS Secrets automatically
- renew certificates automatically
- integrate cleanly with ingress-based application exposure

At the current phase of the lab, public external exposure is not yet required, so full public-domain ACME integration is not immediately necessary.

## Decision

Adopt `cert-manager` as the Kubernetes certificate lifecycle controller for the homelab platform.

`cert-manager` will be:

- installed manually first for learning
- used to manage certificate issuance and renewal inside the cluster
- integrated with `ingress-nginx` through cert-manager-managed TLS Secrets
- initially configured with an internal CA-based trust model

The initial certificate workflow is:

1. bootstrap with a self-signed `ClusterIssuer`
2. generate an internal root CA certificate
3. create a CA-backed `ClusterIssuer`
4. issue application certificates from that internal CA

Public domain and ACME-based certificate issuance are intentionally deferred until they are operationally relevant.

## Consequences

### Positive

- Introduces a Kubernetes-native certificate lifecycle management solution
- Eliminates manual TLS Secret creation for managed certificates
- Enables automated certificate issuance and renewal
- Integrates cleanly with existing ingress-based application exposure
- Supports realistic platform workflows and declarative operations
- Provides immediate learning value without requiring public DNS or external exposure
- Creates a clean future path to Let’s Encrypt or other ACME issuers later

### Negative

- Adds another cluster platform component that must be operated and documented
- Introduces new CRDs and controller behavior that must be understood
- The initial internal CA model does not provide public browser trust
- Clients must trust the internal CA explicitly if trust warnings are to be avoided
- Future public-domain integration will require additional design, credentials, and operational decisions

## Alternatives Considered

- **Manage TLS certificates and Secrets manually**
- **Defer TLS automation entirely until a public domain is introduced**
- **Use a different certificate management approach outside Kubernetes**
- **Adopt public ACME / Let’s Encrypt immediately instead of starting with an internal CA**
