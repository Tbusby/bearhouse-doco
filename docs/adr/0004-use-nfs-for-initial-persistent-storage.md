# ADR-0004: Use NFS for Initial Persistent Storage

- **Status:** Accepted
- **Date:** 2026-05-23

## Context

The platform needs a practical persistent storage solution appropriate for a single-host environment.

## Decision

Use an NFS server VM and `nfs-subdir-external-provisioner` for initial Kubernetes persistent storage.

## Consequences

### Positive

- simple and reliable
- easy to understand and back up
- good fit for single-host constraints

### Negative

- not cloud-native distributed storage
- no host-level resilience

## Alternatives Considered

- Longhorn
- Ceph
- local-path only
