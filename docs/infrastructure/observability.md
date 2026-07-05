# Architecture: Observability Stack

## Purpose

Describe the role of the Kubernetes observability stack in the homelab platform architecture and explain why it was added after ingress and certificate management were established.

## Summary

The observability stack provides cluster monitoring, metrics collection, and dashboard-based visualization for the homelab platform.

The initial implementation uses `kube-prometheus-stack`, which includes Prometheus, Grafana, Alertmanager, kube-state-metrics, node-exporter, and the Prometheus Operator.

This adds a production-shaped monitoring layer on top of the cluster.

## Why the Observability Stack Exists in This Platform

The base cluster already had:

- working Kubernetes control plane and workers
- `metrics-server` for short-window resource metrics
- MetalLB for `LoadBalancer` Services
- `ingress-nginx` for HTTP/HTTPS routing
- `cert-manager` for certificate lifecycle management

However, the platform still lacked a full observability layer.

`metrics-server` is useful for:

- `kubectl top`
- basic CPU and memory checks
- autoscaling-related metrics

It is not a full monitoring stack and does not provide:

- historical metrics
- rich dashboards
- broad scrape coverage of cluster components
- a basis for alerting

The observability stack was added to provide those capabilities.

## Architectural Role

The observability stack sits alongside the platform and continuously collects, stores, and visualizes metrics about:

- nodes
- Kubernetes objects
- cluster components
- application workloads
- platform services

Its role is to improve operational visibility and make the platform easier to operate, troubleshoot, and learn from.

## Why kube-prometheus-stack Was Chosen

`kube-prometheus-stack` was chosen because it provides a widely used, operator-based monitoring stack for Kubernetes.

It was selected because it is:

- common in real-world Kubernetes environments
- well documented
- reasonably production-shaped
- suitable for learning Prometheus Operator patterns
- much faster to adopt than assembling each monitoring component manually

It also gives immediate value while still exposing the underlying Kubernetes resources clearly enough for learning.

## Core Components

The current observability stack includes:

- **Prometheus**
  - scrapes and stores time-series metrics

- **Grafana**
  - provides dashboards and visualization

- **Alertmanager**
  - manages alert routing and grouping

- **kube-state-metrics**
  - exports Kubernetes object state as metrics

- **node-exporter**
  - exports host and node-level OS metrics

- **Prometheus Operator**
  - manages Prometheus-related custom resources and stack lifecycle

## How It Works

The high-level model is:

1. exporters and Kubernetes components expose metrics
2. Prometheus discovers and scrapes them
3. metrics are stored in Prometheus
4. Grafana reads those metrics from Prometheus
5. dashboards present cluster and workload health visually

The Prometheus Operator adds Kubernetes-native management through CRDs such as:

- `Prometheus`
- `Alertmanager`
- `ServiceMonitor`
- `PodMonitor`
- `PrometheusRule`

This is more production-shaped than a single hand-managed Prometheus configuration file.

## Current Storage Model

The initial install uses an intentionally simplified storage model:

- Prometheus persistence is not yet enabled
- Alertmanager persistence is not yet enabled

This was a deliberate staging decision because the lab has not yet introduced its longer-term Kubernetes storage layer.

This makes the current monitoring stack suitable for:

- learning
- cluster visibility
- short-term operational use

It is not yet the intended final-state storage design.

## Access Model

Grafana is currently exposed through the existing platform edge:

- `ingress-nginx`
- `cert-manager`
- internal CA-backed TLS

Grafana is therefore accessible through a stable HTTPS ingress hostname.

Prometheus and Alertmanager remain internal by default unless separately exposed later.

This is an intentional design choice because Grafana is the primary user-facing monitoring interface.

## Current Validated State

The observability stack is installed and functioning in the cluster.

Validated outcomes include:

- Prometheus Operator is running
- Prometheus is running
- Grafana is running
- Alertmanager is running
- kube-state-metrics is running
- node-exporter is running
- Grafana is reachable through ingress with TLS
- Grafana dashboards show Kubernetes cluster monitoring data

## Architectural Benefits

Adding the observability stack gives the platform:

- historical metrics beyond `metrics-server`
- richer operational insight into cluster behavior
- dashboard-based monitoring
- a base for future alerting and SRE-style workflows
- improved troubleshooting capability for later platform services and workloads

## Future Direction

The observability stack prepares the platform for future enhancements such as:

- persistent Prometheus storage
- persistent Alertmanager storage
- alert routing and notification workflows
- custom dashboards
- monitoring of additional workloads
- log aggregation and tracing in later phases
