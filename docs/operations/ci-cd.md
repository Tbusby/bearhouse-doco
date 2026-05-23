# CI/CD

## CI Tool

- GitHub Actions

## CD Tool

- Argo CD

## Delivery Model

CI is responsible for:
- validation
- linting
- testing
- image builds
- security scans
- documentation builds

CD is responsible for:
- reconciling Kubernetes state from Git

## Example CI Responsibilities

- Terraform format and validate
- Terraform plan
- Ansible lint
- YAML lint
- Helm lint
- ShellCheck
- Python tests
- Trivy scans
- MkDocs build

## Design Principle

CI pushes validated artifacts and Git changes.
Argo CD pulls desired state into the cluster.