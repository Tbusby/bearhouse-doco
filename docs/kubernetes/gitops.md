# GitOps

## Tooling

- Argo CD for continuous delivery
- GitHub as the Git source of truth

## Principles

- desired cluster state is stored in Git
- Argo CD reconciles the cluster to match Git
- manual changes should be avoided
- application and platform changes should be traceable through commit history

## Scope

GitOps should manage:
- namespaces
- platform services
- application deployments
- ingress resources
- selected policies and configuration

## Benefits

- repeatability
- auditability
- reduced configuration drift
- easier rollback and review

## Future Notes

This document should later include:
- root app pattern
- app-of-apps or ApplicationSet strategy
- repo structure for manifests