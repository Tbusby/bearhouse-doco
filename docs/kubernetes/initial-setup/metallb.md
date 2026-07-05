# MetalLB

## Purpose

Add a bare-metal `LoadBalancer` implementation to the Kubernetes cluster so Services of type `LoadBalancer` can receive usable LAN IP addresses.

## What It Is

MetalLB is a load balancer implementation for bare-metal Kubernetes environments.

In cloud environments, a cloud provider usually implements the `LoadBalancer` Service type. In a homelab or other bare-metal cluster, Kubernetes can create a `LoadBalancer` Service object, but nothing will assign or advertise an external IP unless an implementation is added.

MetalLB fills that gap.

## Why It Exists

The cluster was healthy and functional, but Services of type `LoadBalancer` were not usable on the LAN without additional platform support.

MetalLB adds the missing capability by:

- allocating IP addresses from a configured pool
- advertising those IPs on the network
- allowing clients on the LAN to reach Kubernetes Services through stable external IPs

This is an important platform layer because it enables later steps such as:

- ingress controller exposure
- cleaner service access than `NodePort`
- more production-shaped service publishing on bare metal

## Current State

MetalLB is installed and working on the cluster.

Current validated outcomes:

- MetalLB controller is running
- MetalLB speakers are running on cluster nodes
- an `IPAddressPool` is configured
- an `L2Advertisement` is configured
- a test `LoadBalancer` Service received an external IP
- the test nginx page was reachable from a laptop on the LAN

## How MetalLB Works

MetalLB has two main jobs:

1. allocate IP addresses to Kubernetes Services of type `LoadBalancer`
2. advertise those IP addresses on the network so traffic reaches the cluster

In this lab, MetalLB is operating in **Layer 2 mode**.

In Layer 2 mode:

- MetalLB assigns an IP from the configured pool to a `LoadBalancer` Service
- one node answers ARP requests for that IP on the LAN
- traffic arrives at the cluster
- Kubernetes forwards the traffic to the backend pods for the Service

## Why Layer 2 Mode Was Chosen

Layer 2 mode was chosen because it is the simplest and most appropriate option for the current environment:

- single LAN-backed subnet
- no BGP router integration required
- good learning value
- realistic for homelab and small bare-metal environments

This is a practical and production-shaped starting point for a bare-metal lab.

## IP Address Pool

The lab uses the following MetalLB address pool:

- `192.168.1.100-192.168.1.110`

This pool is intended for Kubernetes `LoadBalancer` Services.

Design notes:

- the pool must be reserved on the LAN
- the pool must not overlap DHCP allocations
- the pool must not overlap statically assigned host or VM IPs
- the Kubernetes API VIP, if used later, should remain conceptually separate from the MetalLB Service pool

## Why the Pool Was Kept Small

A smaller dedicated pool was chosen intentionally.

Although each `LoadBalancer` Service may consume an IP, most HTTP/HTTPS applications are expected to sit behind a shared ingress controller later. That means many applications can share a single external IP through ingress routing rather than requiring one IP per application.

Using a smaller pool:

- avoids unnecessary address reservation
- keeps the design intentional
- matches realistic platform usage
- still leaves room for ingress, testing, and a few dedicated services

## Components Installed

MetalLB deploys the following core components:

- **controller**
  - watches `LoadBalancer` Services
  - assigns IPs from configured pools

- **speaker**
  - runs on cluster nodes
  - advertises service IPs on the network

Together, these components allow Kubernetes Services of type `LoadBalancer` to function on the homelab LAN.

## Installation Method

MetalLB was installed manually first from `ops1` using the upstream native manifest.

This manual-first approach was used intentionally to understand:

- what components are installed
- how IP pools are defined
- how L2 advertisement works
- how to validate that service exposure works end to end

Automation can be added later after the manual process is fully understood.

## Manual Installation Summary

All commands were run from `ops1`.

A working directory was created:

```bash
mkdir -p ~/src/manual-installs/metallb
cd ~/src/manual-installs/metallb
```

MetalLB was installed from the upstream manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
```

The install was validated with:

```bash
kubectl get pods -n metallb-system -o wide
kubectl get crds | grep metallb
```

## Configuration Applied

An `IPAddressPool` was created for the lab address range:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: lab-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.1.100-192.168.1.110
```

An `L2Advertisement` was then created for that pool:

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: lab-l2
  namespace: metallb-system
spec:
  ipAddressPools:
    - lab-pool
```

These were applied with:

```bash
kubectl apply -f ipaddresspool.yaml
kubectl apply -f l2advertisement.yaml
```

## Test Validation

A simple nginx Deployment was used to validate MetalLB end to end.

A `LoadBalancer` Service was created for the test workload. MetalLB assigned an IP from the configured pool, and the service was successfully accessed from a laptop on the LAN.

This validated all major parts of the workflow:

- MetalLB installation
- IP allocation from the pool
- Layer 2 advertisement on the LAN
- Kubernetes Service routing to pods
- successful external access from another client on the network

## Validation Commands

Check MetalLB pods:

```bash
kubectl get pods -n metallb-system -o wide
```

Check MetalLB CRDs:

```bash
kubectl get crds | grep metallb
```

Check IP pools:

```bash
kubectl get ipaddresspool -n metallb-system
kubectl describe ipaddresspool lab-pool -n metallb-system
```

Check L2 advertisements:

```bash
kubectl get l2advertisement -n metallb-system
kubectl describe l2advertisement lab-l2 -n metallb-system
```

Check the test Service:

```bash
kubectl get svc
kubectl describe svc nginx-test-lb
kubectl get endpoints nginx-test-lb
```

Test access from `ops1` or another LAN client:

```bash
curl http://<assigned-loadbalancer-ip>
```

## What Good Looks Like

MetalLB is functioning correctly when:

- controller and speaker pods are healthy
- `IPAddressPool` and `L2Advertisement` resources exist
- a `LoadBalancer` Service receives an IP from the configured pool
- the Service is reachable from another machine on the LAN
- backend pod traffic works correctly through Kubernetes Service routing

## Risks and Tradeoffs

MetalLB improves service exposure, but introduces a few operational considerations.

Risks:

- IP conflicts if the pool overlaps DHCP or static assignments
- confusion if `LoadBalancer` Services are created without clear IP planning
- troubleshooting complexity if ingress is added before basic service exposure is validated

Tradeoffs:

- Layer 2 mode is simple and effective, but less advanced than BGP mode
- one node advertises a given service IP at a time in L2 mode
- this is still an appropriate and realistic design for the current lab stage

## MetalLB vs Ingress

MetalLB and ingress solve different problems.

MetalLB:

- provides external IP allocation for `LoadBalancer` Services
- makes Services reachable on the LAN

Ingress controller:

- handles HTTP/HTTPS routing
- routes by host or path
- terminates TLS
- exposes multiple web apps behind a smaller number of IPs

A common pattern is:

- MetalLB gives an ingress controller a `LoadBalancer` IP
- the ingress controller exposes many applications behind that one IP

## Future Improvement

Possible future enhancements include:

- automation of MetalLB installation and configuration
- separate address pools for different service categories
- dedicated ingress address pool
- BGP-based advertisement in a more advanced network design
- integration with GitOps for lifecycle management

## What Good Current-State Documentation Includes

The current-state documentation for MetalLB should clearly record:

- why MetalLB was installed
- that Layer 2 mode was chosen
- the exact address pool used
- that the pool is reserved and non-overlapping
- that a test `LoadBalancer` Service was successfully validated from another LAN client
