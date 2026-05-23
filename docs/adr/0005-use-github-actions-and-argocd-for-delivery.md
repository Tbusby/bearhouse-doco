# ADR-0005: Use GitHub Actions and Argo CD for Delivery

- **Status:** Accepted
- **Date:** 2026-05-23

## Context

The platform requires modern CI/CD workflows with strong GitOps alignment.

## Decision

Use GitHub Actions for CI and Argo CD for continuous delivery.

## Consequences

### Positive
- industry-relevant toolchain
- clear separation between CI and CD responsibilities
- strong GitOps support

### Negative
- requires managing multiple tools
- introduces dependency on external Git hosting

## Alternatives Considered

- Jenkins
- GitLab CI/CD
- Flux instead of Argo CD