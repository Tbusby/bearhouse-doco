# Platform Services

## Core Platform Services

The initial platform service set includes:

- Argo CD
- cert-manager
- MetalLB
- ingress-nginx
- metrics-server
- Prometheus
- Grafana
- Alertmanager
- Loki
- NFS dynamic provisioner

## Service Categories

### Delivery

- Argo CD

### Networking

- MetalLB
- ingress-nginx

### Security and Certificates

- cert-manager

### Observability

- metrics-server
- Prometheus
- Grafana
- Alertmanager
- Loki

### Storage

- NFS provisioner

## Deployment Model

Platform services should be deployed via GitOps rather than manual `kubectl apply` after initial bootstrap.
