# Architectural Principles

## Everything as Code

Where practical, all components should be defined as code:
- infrastructure in Terraform
- operating system configuration in Ansible
- Kubernetes resources in Git
- secrets encrypted and stored in Git
- documentation in Markdown

## Git as Source of Truth

Git repositories are the authoritative source for:
- infrastructure definitions
- configuration
- application deployment manifests
- architectural decisions
- operational documentation

Manual changes should be minimized and later captured in code.

## Production-Shaped, Not Overengineered

The lab should reflect realistic patterns without adding unnecessary complexity for a single-host environment.

## Rebuildability Over Snowflakes

Systems should be replaceable and reproducible. Rebuilding a node should be easier than manually repairing configuration drift.

## Secure by Default

Security should be part of the design:
- SSH key-based access
- no plaintext secrets in Git
- least privilege where practical
- patching and upgrade discipline
- network segmentation and policy where useful

## Observability First

Monitoring, dashboards, logs, and backup visibility are required platform capabilities, not optional extras.

## Document Decisions

Key technology and design decisions should be recorded using ADRs.