# Secrets Management

## Approach

Secrets are stored in Git in encrypted form using:
- SOPS
- age

## Goals

- prevent plaintext secrets in repositories
- allow GitOps workflows to remain declarative
- simplify secret management without introducing Vault initially

## Standards

- all Kubernetes secrets committed to Git must be encrypted
- age keys must be backed up securely
- key rotation procedures should be documented

## Future Options

Potential future enhancements:
- External Secrets Operator
- Vault
- cloud secret backends