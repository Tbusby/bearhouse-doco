# `ops1` Management Host

## Overview

`ops1` is the primary management and automation host for the homelab platform.

It acts as the central operational workstation for:

- Terraform infrastructure provisioning
- Ansible configuration management
- Git repository access
- Kubernetes cluster administration
- secret decryption and management with SOPS and age

`ops1` is intentionally separate from the Kubernetes cluster nodes. It is not a control plane node or worker node. Its purpose is to provide a stable administration environment for managing the rest of the platform.

---

## Purpose

The `ops1` host exists to provide a dedicated place to run:

- Terraform plans and applies
- Ansible playbooks
- Git operations against infrastructure and documentation repositories
- `kubectl`, `helm`, and other Kubernetes admin tooling
- SOPS-based secret access and decryption

This avoids overloading the Proxmox host with management tooling and keeps the control environment distinct from the Kubernetes runtime.

---

## Current Role in the Platform

`ops1` currently serves as:

- the normal Terraform execution host
- the normal Ansible control node
- the normal Kubernetes administration workstation
- the host used to manage encrypted secrets
- a reproducible management VM that can be rebuilt from code

The successful destroy-and-rebuild test confirmed that `ops1` is not a snowflake machine and can be recreated using Terraform and Ansible from an alternate control environment.

---

## Host Details

| Item | Value |
| ---- | ----- |
| Hostname | `ops1` |
| Role | Management and automation host |
| Operating System | Ubuntu Server 24.04 LTS |
| Provisioning | Terraform |
| Configuration | Ansible |
| Primary User | `labadmin` |
| Network Addressing | Static IP via cloud-init |
| Current IP Address | `192.168.1.91` |

---

## Responsibilities

`ops1` is responsible for the following operational functions:

### Infrastructure Management

- run Terraform against the Proxmox environment
- inspect and apply infrastructure changes
- act as the normal execution environment for the infrastructure repository

### Configuration Management

- run Ansible playbooks against the lab inventory
- manage baseline configuration for Ubuntu hosts
- manage Kubernetes node preparation
- manage management-host tooling and configuration

### Kubernetes Administration

- provide `kubectl` access to the cluster
- provide Helm access to the cluster
- provide Cilium CLI access where installed
- act as the main operator workstation for post-bootstrap cluster management

### Secret Management

- hold the local age private key used for SOPS decryption
- decrypt SOPS-encrypted files used by Ansible and other workflows
- provide a practical control-node secret management environment

### Source Control

- clone and manage Git repositories used by the lab
- interact with Git remotes using SSH keys
- store and update infrastructure, automation, and documentation repositories

---

## Control Node Assumptions

The current Ansible automation expects the control node to behave in a specific way.

### Ansible runtime assumptions

The Ansible configuration currently assumes:

- inventory path: `./inventory/lab/hosts.yml` [1]
- remote user: `labadmin` [1]
- SSH private key file: `~/.ssh/id_ed25519_ansible` [1]
- roles path: `./roles` [1]
- host key checking enabled [1]
- public-key SSH authentication preferred [1]
- `become = True` using `sudo` [1]
- no password prompt for privilege escalation [1]

### Operational implication

Any alternate machine used to replace or rebuild `ops1` must be able to satisfy these assumptions or temporarily override them.

---

## Required Tooling

`ops1` should contain the tools needed to operate the platform.

### Core operations tooling

- Git
- curl
- wget
- jq
- rsync
- Python 3
- pipx
- Ansible
- ansible-lint
- Terraform
- SOPS
- age

### Kubernetes administration tooling

- kubectl
- helm
- cilium CLI

These tools are installed and managed through Ansible roles where possible.

---

## Secrets Management

`ops1` is the primary host used for secret decryption in the lab.
[Secrets Management](../kubernetes/secrets-management.md)

### Current secret model

Secrets are stored in SOPS-encrypted files and encrypted using age recipients configured in the repository `.sops.yaml` [3].

### Required local secret state

For decryption to work on `ops1`, the host must have:

- `sops` installed
- `age` installed
- the correct age private key available locally

### Encrypted Ansible dependencies

The Ansible repository currently depends on the `community.sops` collection to work with SOPS-encrypted files [2].

### Important note

The age private key must never be committed to Git and must be backed up securely.

---

## Ansible Dependency Model

The Ansible repository currently depends on the following collections:

- `community.general`
- `community.sops` [2]

These must be installable on `ops1` and on any fallback control machine used for rebuild or disaster recovery workflows.

Example installation:

```bash
ansible-galaxy collection install -r requirements.yml
```

---

## Kubernetes Access

`ops1` is intended to be the main cluster administration workstation.

### Expected cluster admin capabilities

From `ops1`, the following should work:

```bash
kubectl get nodes
kubectl get pods -A
helm version
cilium status
```

### Kubeconfig location

The Kubernetes admin kubeconfig for the `labadmin` user should be present at:

```text
/home/labadmin/.kube/config
```

Recommended permissions:

- `.kube/` directory: `0700`
- `config` file: `0600`

---

## Git and SSH Access

`ops1` is expected to support Git operations over SSH.

### Expected behavior

- Git configuration is present for `labadmin`
- Git SSH private/public key material is deployed securely
- SSH configuration points Git operations at the correct key when needed
- GitHub or other Git remote access works non-interactively where expected

### Validation example

```bash
ssh -T git@github.com
```

---

## Rebuildability

A full rebuild of `ops1` has been tested successfully using:

- Terraform from an alternate control machine
- Ansible from an alternate control machine
- SOPS and age from the alternate control machine
- post-build validation of all expected tooling and access

This confirms that `ops1` is rebuildable and not dependent on itself to exist first.

---

## Fallback Control Machine Requirements

Because `ops1` cannot rebuild itself after destruction, a fallback control machine must be available for recovery.

A fallback machine should have:

- Git
- Terraform
- Ansible
- `sops`
- `age`
- the required Ansible collections [2]
- the Ansible SSH private key expected by the current config [1]
- the age private key used to decrypt the repository’s SOPS files [3]

This requirement should be considered part of the platform’s operational model.

---

## Validation Checklist

After provisioning or rebuilding `ops1`, validate the following:

### Tooling

- `terraform version`
- `ansible --version`
- `ansible-lint --version`
- `git --version`
- `sops --version`
- `age --version`
- `kubectl version --client`
- `helm version`
- `cilium version`

### Kubernetes access

- `kubectl get nodes`
- `kubectl get pods -A`
- `helm ls -A`
- `cilium status`

### Git access

- `git config --global --list`
- `ssh -T git@github.com`

### Secret access

- successful decryption of a SOPS-encrypted file
- age private key present locally
- `community.sops` available to Ansible [2]
[Secrets Management](../kubernetes/secrets-management.md)

### Ansible control behavior

- `ansible-inventory --graph`
- `ansible all -m ping`

### Terraform control behavior

- `terraform init`
- `terraform plan`

---

## Design Notes

- `ops1` is intentionally not part of the Kubernetes cluster
- `ops1` is designed to be rebuildable from a separate control machine
- `ops1` holds important control-plane-adjacent tools but is not itself a control plane node
- the lab currently depends on a fallback machine for disaster recovery of the management host
- the current model is appropriate for a single-host homelab and aligns with the goal of learning production-shaped workflows

---

## Future Enhancements

Potential future improvements for `ops1` include:

- stronger version pinning of admin tools
- more complete automation of kubeconfig deployment
- additional CLI tooling such as `k9s` or `yq`
- more formal validation playbooks
- documented shell profile standardization
- expanded secret-management workflows

---

## Related Documents

- [Configuration Management](../operations/configuration-management.md)
- [Node Bootstrap](../operations/node-bootstrap.md)
- [Run Ansible Configuration Management](../runbooks/run-ansible-configuration-management.md)
- [Rebuild `ops1` Management Host](../runbooks/rebuild-ops1-management-host.md)
- [Kubernetes Cluster Bootstrap Design](../kubernetes/cluster-bootstrap-design.md)
- [Secrets Management](../kubernetes/secrets-management.md)
