# Observability

## Monitoring Stack

- Prometheus
- Grafana
- Alertmanager
- metrics-server

## Logging Stack

- Loki
- Promtail or Grafana Alloy

## Goals

The platform should provide visibility into:

- node health
- pod health
- cluster resource usage
- storage usage
- ingress and service health
- backup success or failure

## Initial Alerts

- node unavailable
- disk pressure
- failed backup job
- high resource usage
- certificate expiry threshold
