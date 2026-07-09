# Platform Backlog

## Purpose

Track follow-up work, fixes, improvements, hardening tasks, and future enhancements across the homelab platform.

This page is intended to capture items discovered during build-out and operation so they are not lost or left as undocumented tribal knowledge.

## How to Use This Page

Use this backlog to record work that is:

- not urgent enough to do immediately
- intentionally deferred
- discovered during manual installs or validation
- part of future hardening or automation work
- related to security, operations, reproducibility, or cleanup

This backlog is not limited to one category of work. It can include:

- technical debt
- documentation improvements
- automation follow-up
- security cleanup
- storage improvements
- GitOps migration tasks
- observability enhancements
- backup and recovery work

## Suggested Status Values

Use consistent status values so the backlog remains readable.

Recommended values:

- `Todo`
- `Planned`
- `In Progress`
- `Blocked`
- `Done`
- `Deferred`

## Suggested Priority Values

Use a simple priority model.

Recommended values:

- `High`
- `Medium`
- `Low`

## Backlog Template

| ID | Area | Item | Reason / Context | Type | Priority | Status | Notes |
| -- | ---- | ---- | ---------------- | ---- | -------- | ------ | ----- |
| BL-000 | Example | Example item | Why this exists or was deferred | Hardening / Automation / Docs / Ops / Security / Cleanup | Medium | Todo | Optional extra detail |

## Active Backlog

| ID | Area | Item | Reason / Context | Type | Priority | Status | Notes |
| -- | ---- | ---- | ---------------- | ---- | -------- | ------ | ----- |
| BL-001 | Kubernetes / metrics-server | Revisit kubelet TLS validation for metrics-server | `metrics-server` currently uses `--kubelet-insecure-tls` as a documented homelab workaround; preferred future state is proper kubelet serving certificate validation | Security / Hardening | High | Todo | Remove insecure flag only after proper validation path is working |
| BL-002 | Storage / NFS | Tighten NFS export permissions and options | Current NFS export uses lab-friendly settings such as permissive directory permissions and `no_root_squash`; revisit for a stricter model later | Security / Hardening | Medium | Todo | Keep current state documented as intentional homelab simplification |
| BL-003 | DNS / Access | Replace `/etc/hosts`-based name resolution with a more formal internal DNS approach | Several internal ingress hostnames currently depend on client-side `/etc/hosts` entries; this is acceptable for learning but not ideal long-term | Platform / Networking | Medium | Todo | Could later align with internal DNS or real domain strategy |
| BL-004 | Observability | Review persistent storage sizing and retention for Prometheus | Prometheus persistence is now enabled on NFS-backed storage; future tuning may be needed for retention, storage size, and performance tradeoffs | Operations / Capacity | Medium | Todo | Current config is appropriate for initial lab use |
| BL-005 | GitOps / Argo CD | Define long-term GitOps repo structure and ownership boundaries | Argo CD is installed and validated with a first demo app; future structure should define what belongs in GitOps, what remains Helm-managed, and what remains Ansible-managed | Platform / GitOps | High | Todo | Important before broader GitOps adoption |
| BL-006 | Backups / Recovery | Define and implement backup strategy for Kubernetes resources and NFS-backed data | The platform now has persistent state worth protecting; recovery design should be intentional rather than ad hoc | Operations / Recovery | High | Todo | Likely include Kubernetes-aware backups and NFS data backup strategy |
| BL-007 | Security / Access | Review and rotate bootstrap or initial admin credentials used during platform installs | Several components may have used initial passwords or bootstrap secrets during manual-first installs; verify steady-state credential posture | Security / Cleanup | High | Todo | Includes Grafana, Argo CD, and any other manually bootstrapped services |
| BL-008 | Automation | Convert stable manual platform install steps into declared managed state | Several components were installed manually first for learning; once stable, their installation and configuration should be reflected in Git/automation cleanly | Automation / Platform | High | Planned | Good candidates include MetalLB, ingress-nginx, cert-manager, monitoring, and storage add-ons |

## Component Review Checklist

Use this section when reviewing an existing platform component for hardening or automation follow-up.

### Questions

- Is the current install method documented?
- Is there a clear source of truth for the component?
- Is any part of the configuration still only present as live cluster state?
- Were any security shortcuts used?
- Were any lab-only simplifications used?
- Is the component reproducible today?
- Should it be moved under GitOps, Helm values management, Ansible, or another managed pattern?
- Does the component need backup or restore planning?
- Does the current implementation still match the intended architecture?

### Optional Review Template

| Component | Current Source of Truth | Known Shortcut / Debt | Desired End State | Priority | Notes |
| --------- | ----------------------- | --------------------- | ----------------- | -------- | ----- |
| Example component | Manual + docs | Example shortcut | Git-managed + automated | Medium | Example note |

## Notes

- This backlog is intended to evolve continuously as the platform matures.
- It is acceptable to add small items here during day-to-day work rather than trying to fully classify them immediately.
- If an item becomes large enough to need design discussion, consider creating an ADR or a dedicated documentation page in addition to the backlog entry.
- Closed or completed items can either remain here marked `Done` or be moved to a changelog/improvements section later if desired.
