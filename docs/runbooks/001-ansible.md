# Runbook: Run Ansible Configuration Management

## Purpose

Run Ansible playbooks against the homelab environment to apply baseline configuration and Kubernetes node preparation in a controlled and repeatable way.

This runbook covers the normal workflow for:

- validating configuration
- confirming inventory targeting
- running selected playbooks
- applying full site configuration
- reviewing outcomes

## Scope

Applies to the Ansible configuration management repository used for:

- baseline Ubuntu configuration
- Kubernetes prerequisite setup
- Kubernetes package and containerd installation

This runbook assumes execution from the management environment.

## Prerequisites

- access to the Ansible repository
- SSH connectivity from the management host to all target nodes
- key-based SSH access working
- Python available on managed Ubuntu hosts
- inventory file present and up to date
- required Ansible dependencies installed
- no conflicting maintenance or provisioning activity in progress

## Expected Execution Context

This runbook assumes execution from the Ansible repository root on the management host.

Example repository layout includes:

- `ansible.cfg`
- `inventory/lab/hosts.yml`
- `playbooks/`
- `roles/`
- `group_vars/`

## Risks

- configuration changes may restart services
- Kubernetes prerequisite changes may alter kernel or swap behavior
- package installation may change host state significantly
- running broad playbooks against the wrong inventory can affect unintended hosts
- SSH hardening changes may lock out access if not validated carefully

## Pre-Run Checks

### 1. Change into the repository

On ops1

```bash
cd ~/bearhouse-infra-ansible
```

### 2. Confirm repository status

```bash
git status
```

Review any uncommitted changes before running playbooks.

### 3. Confirm Ansible version

```bash
ansible --version
```

### 4. Confirm inventory file

```bash
ansible-inventory -i inventory/lab/hosts.yml --graph
```

Verify the expected groups and hosts are present.

### 5. Confirm target reachability

Test all managed Kubernetes nodes:

```bash
ansible -i inventory/lab/hosts.yml k8s_cluster -m ping
```

If needed, test control plane and worker groups separately:

```bash
ansible -i inventory/lab/hosts.yml k8s_control -m ping
ansible -i inventory/lab/hosts.yml k8s_workers -m ping
```

## Common Playbook Runs

### Run baseline configuration only

Use this to apply common Ubuntu baseline settings:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/baseline.yml
```

### Run Kubernetes prerequisites only

Use this to prepare Kubernetes nodes:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/k8s_prereqs.yml
```

### Run Kubernetes package installation only

Use this to install Kubernetes packages and containerd configuration:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/k8s_install.yml
```

### Run bootstrap known_hosts workflow

Use this when new hosts have been provisioned and SSH trust needs to be pre-populated:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/bootstrap.yml
```

### Run full site configuration

Use this to apply the current full configuration set:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml
```

## Recommended Execution Order for New Nodes

For newly provisioned nodes, use the following order:

1. `bootstrap.yml`
2. `baseline.yml`
3. `k8s_prereqs.yml`
4. `k8s_install.yml`

Or, if the site playbook correctly orchestrates the full sequence:

1. `bootstrap.yml`
2. `site.yml`

## Dry Run / Change Preview

When appropriate, use check mode before applying changes:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml --check
```

To see detailed task differences where supported:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml --check --diff
```

Note that not all modules fully support check mode.

## Target a Single Host

To apply configuration to a single node:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml --limit k8s-cp1
```

## Target a Group

To apply configuration only to control plane nodes:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml --limit k8s_control
```

To apply configuration only to worker nodes:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml --limit k8s_workers
```

## Optional Verbose Mode

For troubleshooting, increase verbosity:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml -v
```

More detail:

```bash
ansible-playbook -i inventory/lab/hosts.yml playbooks/site.yml -vv
```

Use `-vvv` or `-vvvv` only when deeper debugging is required.

## Validation

After a successful run, validate:

### 1. Ansible completed without failed hosts

Review the final recap for:

- `failed=0`
- `unreachable=0`

### 2. Hosts remain reachable

```bash
ansible -i inventory/lab/hosts.yml k8s_cluster -m ping
```

### 3. Guest agent is running where expected

Example ad hoc check:

```bash
ansible -i inventory/lab/hosts.yml k8s_cluster -b -m service -a "name=qemu-guest-agent state=started enabled=true"
```

### 4. Swap is disabled on Kubernetes nodes

```bash
ansible -i inventory/lab/hosts.yml k8s_cluster -b -m command -a "swapon --show"
```

Expected result: no active swap entries.

### 5. Kubernetes packages are installed

Example:

```bash
ansible -i inventory/lab/hosts.yml k8s_cluster -b -m shell -a "kubeadm version && kubelet --version"
```

### 6. containerd is running

```bash
ansible -i inventory/lab/hosts.yml k8s_cluster -b -m service -a "name=containerd state=started enabled=true"
```

## Rollback / Fallback

If a playbook run causes issues:

- stop and identify the failing task
- rerun with `--limit <host>` to isolate impact
- use verbose output to diagnose the failure
- revert relevant repository changes if the issue came from a recent modification
- restore host access before attempting further hardening changes
- do not continue to broader host groups until the issue is understood

For high-risk changes:

- consider snapshots before major configuration changes
- apply first to one host only
- validate before applying to the full group

## Follow-Up

After a successful run:

- update documentation if behavior or structure changed
- commit any associated repository changes
- capture notable issues in runbooks or ADRs if they affected design
- confirm readiness for the next phase, such as Kubernetes bootstrap

## References

- [Configuration Management](../operations/configuration-management.md)
- [Node Bootstrap](../operations/node-bootstrap.md)
- [Ansible Repository Structure](../standards/ansible-repository-structure.md)
