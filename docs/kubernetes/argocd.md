# Argo CD

## Purpose

Document the Kubernetes-specific implementation details of Argo CD in the cluster.

## Overview

Argo CD is the GitOps controller currently used by the cluster to deploy and reconcile Kubernetes application resources from Git repositories.

It allows the cluster to:

- define application desired state in Git
- create applications declaratively
- observe sync and health status
- detect drift between Git and cluster state

## Why It Was Added

The cluster needed a more production-shaped application delivery model after the foundational platform services were established.

Before Argo CD:

- applications and add-ons were installed manually for learning
- desired state lived primarily in operator knowledge, manifests, and documentation
- cluster reconciliation from Git was not yet present

After Argo CD:

- a Git repository can define the desired state of a Kubernetes application
- the cluster can reconcile toward that Git-defined state
- sync status and health are visible operationally

## Namespace

Argo CD is installed in:

- `argocd`

## Installation Method

Argo CD was installed manually first from the upstream manifest.

This was chosen so the following could be understood directly:

- which control plane components are installed
- how the Argo CD UI and API are exposed
- how an `Application` resource works
- how Git-driven reconciliation behaves

## Installed Components

The Argo CD installation includes core components such as:

- `argocd-server`
- `argocd-repo-server`
- `argocd-application-controller`
- Redis
- supporting services, secrets, configmaps, and RBAC

Exact component names may vary slightly by release, but these are the key operational pieces.

## Exposure Model

Argo CD is exposed through:

- `ingress-nginx`
- `cert-manager`
- an internal CA-backed certificate

The Argo CD server is accessed through an ingress hostname using HTTPS.

## Initial Access Model

Initial access was validated via port-forwarding before ingress exposure.

This followed the same validation pattern used elsewhere in the platform:

1. validate the service internally first
2. then expose it through ingress

## TLS Model

Argo CD currently uses a TLS certificate issued by the internal cluster CA via `cert-manager`.

This means:

- HTTPS works end to end
- clients may need to trust the internal CA to avoid browser warnings

## Initial Authentication

The initial admin password was retrieved from:

- `argocd-initial-admin-secret`

This is suitable for the initial installation workflow and should be treated as bootstrap access.

## First GitOps-Managed Application

A small `whoami` demo application was used as the first GitOps-managed application.

This application included:

- `Deployment`
- `Service`
- `Certificate`
- `Ingress`

An `Application` resource in the `argocd` namespace pointed Argo CD at the Git repository path containing those manifests.

## First Application Behavior

The initial `whoami` application demonstrated:

- Git-driven application deployment
- namespace creation through sync options
- application health and sync status reporting
- interaction between Git-managed resources and pre-existing manual resources

An ingress conflict occurred because a previously created manual ingress used the same hostname and path. Removing the old manual resource resolved the conflict and allowed the GitOps-managed app to become the canonical version.

## Example Application Resource

Example initial `Application` resource pattern:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: whoami
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR-USER/YOUR-REPO.git
    targetRevision: HEAD
    path: apps/whoami
  destination:
    server: https://kubernetes.default.svc
    namespace: whoami
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

## Current GitOps Scope

The current Argo CD scope is intentionally limited.

Argo CD is currently being used to learn and validate GitOps behavior with a small application rather than immediately taking ownership of all platform services.

This keeps the blast radius small and makes the reconciliation model easier to observe.

## Operational Checks

Check Argo CD pods:

```bash
kubectl get pods -n argocd
```

Check Argo CD services:

```bash
kubectl get svc -n argocd
```

Check Argo CD ingress:

```bash
kubectl get ingress -n argocd
kubectl describe ingress argocd -n argocd
```

Check Argo CD certificate:

```bash
kubectl get certificate -n argocd
kubectl describe certificate argocd-tls -n argocd
```

Check Argo CD applications:

```bash
kubectl get applications -n argocd
kubectl describe application <app-name> -n argocd
```

## Good State

Argo CD is considered healthy when:

- core Argo CD pods are running
- the UI is reachable
- the ingress and TLS resources are healthy
- applications can sync successfully from Git
- sync and health status are visible and correct

## Risks and Notes

Important notes for this cluster:

- Argo CD introduces a new ownership model for Kubernetes resources
- pre-existing manual resources can conflict with GitOps-managed resources
- ingress host/path conflicts must be avoided
- the current Argo CD adoption is intentionally incremental rather than platform-wide
- internal CA trust behavior still applies to browser access

## Next Step

Future GitOps-related improvements may include:

- enabling automated sync/self-heal on selected applications
- managing more workloads through Argo CD
- introducing stronger repo structure patterns
- adopting app-of-apps or similar higher-level GitOps patterns
