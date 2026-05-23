# ADR-0003: Use Cilium as CNI

- **Status:** Accepted
- **Date:** 2026-05-23

## Context

The platform needs a modern, production-relevant CNI with strong policy capabilities.

## Decision

Use Cilium as the Kubernetes CNI.

## Consequences

### Positive
- modern and widely adopted
- strong NetworkPolicy support
- useful for learning current Kubernetes networking patterns

### Negative
- somewhat more advanced than simpler alternatives
- may introduce additional concepts early

## Alternatives Considered

- Calico
- Flannel