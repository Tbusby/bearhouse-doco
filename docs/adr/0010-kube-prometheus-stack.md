# ADR-0010: Adopt kube-prometheus-stack as the Initial Kubernetes Observability Stack

- **Status:** Accepted
- **Date:** 2026-07-05

## Context

The homelab Kubernetes cluster now has working foundational platform services including:

- `metrics-server`
- MetalLB
- `ingress-nginx`
- `cert-manager`

This provides cluster exposure, ingress routing, and certificate automation, but it does not yet provide a full observability layer.

`metrics-server` is useful for short-window resource visibility and Kubernetes features such as `kubectl top` and HPA inputs, but it does not provide:

- historical metric storage
- dashboarding
- broad scrape coverage
- a foundation for alerting

The platform therefore requires a more complete observability solution that can:

- collect and store time-series metrics
- monitor nodes, workloads, and Kubernetes objects
- provide dashboards for operators
- support future alerting workflows
- align with Kubernetes-native operational patterns

The solution should fit the platform goals of:

- production-shaped design
- reproducibility
- realistic operational workflows
- manual-first learning before automation
- clear documentation and later automation

## Decision

Adopt `kube-prometheus-stack` as the initial Kubernetes observability stack for the homelab platform.

The stack will be:

- installed manually first using Helm
- deployed into a dedicated `monitoring` namespace
- used to provide Prometheus, Grafana, Alertmanager, node-exporter, kube-state-metrics, and the Prometheus Operator
- initially configured with a simplified no-persistence model
- integrated with the existing ingress and cert-manager pattern for Grafana exposure

Grafana will be the primary user-facing observability interface exposed through ingress.
Prometheus and Alertmanager will remain internal by default.

## Consequences

### Positive

- Introduces a widely used Kubernetes-native monitoring stack
- Provides historical metrics and dashboards beyond `metrics-server`
- Uses the Prometheus Operator model, which is common in real Kubernetes environments
- Enables better troubleshooting and operational visibility for future platform layers
- Exposes Grafana cleanly through the existing ingress and certificate workflow
- Allows learning of ServiceMonitor, PodMonitor, and PrometheusRule concepts later
- Aligns with a realistic manual Helm-based learning workflow for a larger platform component

### Negative

- Adds a substantial new platform component with many moving parts
- Introduces operator-specific CRDs and resources that require learning
- The initial no-persistence deployment is useful for learning but not a final-state monitoring design
- Grafana admin credentials need stronger lifecycle handling later
- Alertmanager is installed but not yet fully used as part of a notification/alerting workflow

## Alternatives Considered

- **Rely only on `metrics-server` for monitoring**
- **Install Prometheus and Grafana manually as separate components**
- **Delay observability until after storage or GitOps**
- **Adopt a different monitoring stack outside the Prometheus Operator ecosystem**
