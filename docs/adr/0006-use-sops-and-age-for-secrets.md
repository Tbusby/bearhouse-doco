# ADR-0006: Use SOPS and age for Secrets

- **Status:** Accepted
- **Date:** 2026-05-23

## Context

The platform needs a secure but practical secrets workflow compatible with GitOps.

## Decision

Use SOPS with age for encrypting secrets stored in Git.

## Consequences

### Positive
- simple and effective
- Git-friendly
- avoids introducing Vault too early

### Negative
- key management still requires discipline
- not a full centralized secrets platform

## Alternatives Considered

- HashiCorp Vault
- sealed-secrets
- plaintext local secret files