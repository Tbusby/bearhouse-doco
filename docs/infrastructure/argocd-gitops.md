# Argo CD and GitOps

## Purpose

Describe the role of Argo CD in the homelab platform architecture and explain why GitOps was introduced at this stage.

## Summary

Argo CD provides the cluster’s GitOps application delivery and reconciliation layer.

The platform now has mature foundational services for:

- ingress
- certificates
- observability
- persistent storage

Argo CD was introduced to improve how Kubernetes applications are managed over time by making Git the source of truth for application state.

## Why Argo CD Exists in This Platform

Before Argo CD, the cluster add-on and application workflow was primarily:

- manual installation for learning
- validation in-cluster
- documentation afterward
- future automation planned later

That was the correct pattern for understanding platform components, but it does not by itself provide a steady-state operational model for Kubernetes application delivery.

Argo CD was added to provide:

- Git-based application deployment
- drift detection
- declarative reconciliation
- a more production-shaped application management model

This supports the broader platform goals of:

- reproducibility
- documentation
- infrastructure as code
- realistic operational workflows

## Architectural Role

Argo CD sits above the base Kubernetes platform and acts as the cluster’s application reconciliation engine.

Its role is to continuously compare:

- desired state in Git
- actual state in the cluster

and then:

- report differences
- optionally synchronize the cluster back to the declared state

This creates a clean separation between:

- infrastructure provisioning
- host configuration
- Kubernetes application delivery

## Relationship to Terraform and Ansible

Argo CD does not replace Terraform or Ansible.

The current platform model is:

- **Terraform**
  - provisions infrastructure resources such as VMs

- **Ansible**
  - configures hosts and supports cluster/bootstrap workflows

- **Argo CD**
  - manages Kubernetes application state from Git

This is a strong and realistic separation of concerns.

## Why Argo CD Was Chosen

Argo CD was selected because it is:

- widely used
- well documented
- strongly aligned with GitOps workflows
- easy to visualize through its UI
- a very useful learning tool for cluster reconciliation concepts

It provides a more approachable first GitOps experience than some alternatives because it makes drift, sync status, and health visible directly in the UI.

## Current GitOps Adoption Model

GitOps is being introduced incrementally.

The platform is not immediately converting every existing platform component into Argo CD-managed state. Instead, the adoption model is:

1. install Argo CD manually
2. validate UI and ingress access
3. deploy a small demo application from Git
4. observe sync and drift behavior
5. later decide what broader platform or application components Argo CD should own

This is a deliberate learning-first adoption pattern.

## Current Exposure Model

Argo CD is exposed through the existing shared platform edge:

- `ingress-nginx`
- `cert-manager`
- internal CA-backed TLS

This makes Argo CD available as a normal cluster application served through ingress and HTTPS.

## Current Validated State

Argo CD is installed and functioning in the cluster.

Validated outcomes include:

- Argo CD control plane components are running
- the Argo CD UI is reachable
- Argo CD is exposed through ingress with TLS
- an initial GitOps-managed demo application is deployed successfully
- Argo CD detects and reports desired-vs-live state
- Git is now part of the cluster application delivery path

## GitOps Ownership Boundary

The current GitOps boundary is intentionally small.

A demo application was chosen as the first managed workload so that:

- Argo CD behavior could be learned safely
- sync and drift could be observed without affecting core platform services
- ownership boundaries between manual platform setup and GitOps-managed applications remain clear

This avoids trying to GitOps-convert the entire platform in one step.

## Lessons Reinforced by the First App

The first GitOps-managed application highlighted an important operational lesson: pre-existing manually created resources can conflict with GitOps-managed resources.

In this case, a manually created ingress with the same hostname/path as the GitOps app caused a conflict until the manual resource was removed.

This reinforces the importance of:

- clear ownership boundaries
- avoiding duplicate ingress definitions
- understanding transition paths from manual to GitOps-managed resources

## Architectural Benefits

Adding Argo CD gives the platform:

- a Git-based application source of truth
- drift detection
- a repeatable application deployment model
- a clear path toward broader GitOps adoption later
- a stronger operational story for day-2 cluster management

## Future Direction

Possible future Argo CD enhancements include:

- broader application management through GitOps
- automated sync and self-heal for selected apps
- app-of-apps patterns
- GitOps management of additional platform workloads
- integration with secrets patterns suitable for GitOps
