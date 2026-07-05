# Metrics Server

## Purpose

Add the Kubernetes resource Metrics API to the cluster so node and pod CPU/memory usage can be queried and used by cluster features such as `kubectl top` and Horizontal Pod Autoscaler (HPA).

## What It Is

`metrics-server` is a lightweight cluster service that collects resource usage metrics from the kubelet running on each node and publishes them through the Kubernetes Metrics API.

It is used for:

- `kubectl top nodes`
- `kubectl top pods -A`
- autoscaling inputs such as HPA

It is not:

- a full monitoring stack
- long-term metrics storage
- a replacement for Prometheus or Grafana

## Why It Exists

A base Kubernetes cluster can be healthy and fully functional without exposing the Metrics API. `metrics-server` adds a core operational capability so the cluster can report near-real-time resource usage for nodes and pods.

This is one of the first platform services layered onto the base cluster after bootstrap.

## Current State

`metrics-server` is installed and working on the cluster.

Current validated outcomes:

- the Deployment is running in `kube-system`
- the Metrics APIService is registered
- `kubectl top nodes` works
- `kubectl top pods -A` works

The current install uses a temporary homelab workaround:

- `--kubelet-insecure-tls`

## Why a Workaround Was Needed

`metrics-server` connects to each node’s kubelet over HTTPS.

For strict TLS verification to work correctly, the kubelet serving certificate must:

1. be signed by a trusted internal signer or CA
2. include SANs matching the node address used by `metrics-server`

In this lab, kubelet serving certificate validation is not yet fully aligned for strict verification, so `metrics-server` required the insecure kubelet TLS workaround to function reliably.

This is an internal Kubernetes PKI issue, not an ingress or public certificate issue.

## Install Method

`metrics-server` was installed manually first from `ops1` using the upstream release manifest.

This manual-first approach was used intentionally to understand:

- what resources are created
- how the Deployment is configured
- how the Metrics API is registered
- how validation and troubleshooting work

Automation can be added later after the manual process is fully understood.

## Resources Created

The install creates resources including:

- ServiceAccount
- RBAC resources
- Service
- Deployment
- APIService for `metrics.k8s.io`

The key functional results are:

- a running `metrics-server` Deployment in `kube-system`
- an available APIService for `v1beta1.metrics.k8s.io`

## Manual Installation Summary

All commands were run from `ops1`.

A working directory was created:

```bash
mkdir -p ~/src/manual-installs/metrics-server
cd ~/src/manual-installs/metrics-server
```

The upstream manifest was downloaded:

```bash
curl -L -o components.yaml \
  https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

The manifest was reviewed before applying:

```bash
head -n 40 components.yaml
grep -n "kind: Deployment" components.yaml
grep -n "args:" -A20 components.yaml
```

The manifest was then applied:

```bash
kubectl apply -f components.yaml
```

The pod startup was observed:

```bash
kubectl get pods -n kube-system -l k8s-app=metrics-server -w
```

## Workaround Applied

After initial install, the Deployment was edited to add:

```yaml
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
```

This was applied by editing the Deployment directly:

```bash
kubectl edit deployment metrics-server -n kube-system
```

The rollout was then monitored:

```bash
kubectl rollout status deployment/metrics-server -n kube-system
```

## Validation

The installation was validated with the following commands:

```bash
kubectl get deploy,po,svc -n kube-system | grep metrics-server
kubectl get apiservice | grep metrics
kubectl describe apiservice v1beta1.metrics.k8s.io
kubectl top nodes
kubectl top pods -A
```

Optional direct Metrics API checks:

```bash
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```

A good result is:

- `metrics-server` pod is `Running`
- the Deployment is healthy
- APIService `v1beta1.metrics.k8s.io` is `Available=True`
- `kubectl top nodes` returns CPU and memory usage
- `kubectl top pods -A` returns pod metrics

## Operational Commands

Check the Deployment and pod:

```bash
kubectl get deploy,po -n kube-system | grep metrics-server
```

Check APIService status:

```bash
kubectl describe apiservice v1beta1.metrics.k8s.io
```

View logs:

```bash
kubectl logs -n kube-system deploy/metrics-server
```

Query node metrics:

```bash
kubectl top nodes
```

Query pod metrics:

```bash
kubectl top pods -A
```

Query the raw Metrics API:

```bash
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```

## Risks and Tradeoffs

Using `--kubelet-insecure-tls` is a homelab shortcut.

Benefits:

- allows progress without blocking on kubelet internal PKI work
- enables the Metrics API now
- supports continued learning on later platform services

Tradeoffs:

- kubelet serving certificate verification is weakened
- this is not the preferred production-style end state
- the workaround must be documented and revisited later

## Future Improvement

The preferred future state is to remove:

```text
--kubelet-insecure-tls
```

That will require:

- kubelet serving certificates issued by a trusted internal signer or CA
- SANs that match the node identities used by `metrics-server`
- node address selection aligned with certificate identity

This is separate from ingress TLS and separate from `cert-manager` use for applications.

## What Good Looks Like

The cluster is in a good current-state condition when:

- `metrics-server` is running successfully
- the Metrics APIService is available
- `kubectl top` works from `ops1`
- the insecure kubelet TLS workaround is documented
- a future cleanup item exists to remove the workaround
