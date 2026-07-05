# Kubernetes: kube-prometheus-stack

## Purpose

Document the Kubernetes-specific implementation details of the cluster observability stack.

## Overview

The current observability stack is implemented using the `kube-prometheus-stack` Helm chart.

This provides a Kubernetes-native monitoring stack centered on the Prometheus Operator and includes:

- Prometheus
- Grafana
- Alertmanager
- kube-state-metrics
- node-exporter
- monitoring CRDs managed by the Prometheus Operator

## Why It Was Added

The cluster required a full monitoring stack beyond the limited resource metrics provided by `metrics-server`.

Before this stack:

- `kubectl top` worked
- short-window CPU and memory inspection was available
- basic troubleshooting relied on `kubectl get`, `describe`, and `logs`

After this stack:

- historical metrics are available
- dashboards are available
- scrape targets can be inspected
- the cluster has a proper observability baseline

## Namespace

The observability stack is installed in:

- `monitoring`

## Installation Method

The stack was installed manually first using Helm from `ops1`.

This approach was chosen because:

- the stack is large and highly configurable
- Helm is a realistic lifecycle tool for this type of platform component
- it still supports manual learning through explicit values inspection and validation

## Helm Release

Current release name:

- `kube-prometheus-stack`

Chart source:

- `prometheus-community/kube-prometheus-stack`

## Core Components Installed

The stack includes:

- Prometheus Operator
- Prometheus
- Alertmanager
- Grafana
- kube-state-metrics
- node-exporter

## Prometheus Operator Resource Model

The stack introduces Prometheus Operator CRDs, including:

- `prometheuses.monitoring.coreos.com`
- `alertmanagers.monitoring.coreos.com`
- `servicemonitors.monitoring.coreos.com`
- `podmonitors.monitoring.coreos.com`
- `prometheusrules.monitoring.coreos.com`

These resources enable Kubernetes-native monitoring configuration.

## Current Helm Values Summary

The initial install used a minimal custom values file with a deliberately simple configuration.

Key characteristics:

- Grafana admin password explicitly set for initial manual login
- Grafana service kept internal as `ClusterIP`
- Grafana ingress disabled in Helm initially
- Prometheus retention set to `7d`
- Prometheus persistence not enabled
- Alertmanager persistence not enabled

## Example Values Used

Example initial values:

```yaml
grafana:
  adminPassword: "change-me-after-login"

  service:
    type: ClusterIP

  ingress:
    enabled: false

prometheus:
  prometheusSpec:
    retention: 7d
    storageSpec: null

alertmanager:
  alertmanagerSpec:
    storage: null
```

## Current Exposure Model

Grafana is exposed through a separate ingress configuration rather than directly through Helm-managed ingress values.

This was chosen for the initial learning phase because it makes the ingress and certificate integration more explicit.

Current Grafana exposure path:

- Grafana Service in `monitoring`
- `Ingress` resource in `monitoring`
- TLS Secret issued by `cert-manager`
- ingress handled by `ingress-nginx`

## Current Grafana TLS Model

Grafana is currently exposed with HTTPS using the existing internal CA-based cert-manager workflow.

This means:

- TLS is automated
- certificates are issued by the lab internal CA
- clients may need to trust the internal CA to avoid browser warnings

## Validation Performed

The stack was validated by:

- confirming Helm release installation succeeded
- confirming monitoring pods were healthy
- confirming monitoring CRDs were installed
- accessing Grafana
- confirming Prometheus was configured as a Grafana data source
- confirming dashboards displayed Kubernetes cluster metrics
- exposing Grafana through ingress
- confirming HTTPS access to Grafana through ingress

## Operational Checks

Check Helm release:

```bash
helm list -n monitoring
helm status kube-prometheus-stack -n monitoring
```

Check monitoring pods:

```bash
kubectl get pods -n monitoring
```

Check monitoring services:

```bash
kubectl get svc -n monitoring
```

Check Prometheus Operator CRDs:

```bash
kubectl get crds | grep monitoring.coreos.com
```

Check operator-managed resources:

```bash
kubectl get prometheus,alertmanager -n monitoring
kubectl get servicemonitors,podmonitors,prometheusrules -A
```

Check Grafana ingress:

```bash
kubectl get ingress -n monitoring
kubectl describe ingress grafana -n monitoring
```

Check Grafana TLS certificate:

```bash
kubectl get certificate -n monitoring
kubectl describe certificate grafana-tls -n monitoring
```

## Good State

The stack is considered healthy when:

- Helm release is deployed successfully
- core monitoring pods are running
- Prometheus Operator CRDs exist
- Prometheus is scraping targets successfully
- Grafana dashboards show cluster data
- Grafana is reachable through ingress with TLS

## Risks and Notes

Important notes for this cluster:

- the current install uses no persistent Prometheus or Alertmanager storage
- the current Grafana admin credential handling is suitable for initial learning but not final-state secret design
- Grafana is intentionally the main exposed observability UI
- Prometheus and Alertmanager are intentionally kept internal for now

## Next Step

Future observability improvements may include:

- persistent storage for Prometheus
- persistent storage for Alertmanager
- custom dashboards
- alert routing configuration
- monitoring of additional workloads
- later log aggregation and tracing