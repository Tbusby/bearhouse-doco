# Security Baseline

## Access Control

- SSH keys only
- password authentication disabled
- root SSH login disabled where practical
- least privilege administrative access

## Host Security

- minimal installed packages
- routine patching
- firewall enabled where practical
- time synchronization configured
- logs retained appropriately

## Kubernetes Security

- RBAC enabled
- namespace separation by function
- NetworkPolicies implemented
- privileged workloads minimized
- TLS used for ingress

## Secrets

- secrets stored encrypted with SOPS
- age keys protected and backed up securely
- plaintext secrets prohibited in repositories

## CI Security Controls

CI should include:
- Trivy scanning
- linting
- manifest validation
- shell and Python quality checks