# Secrets Management with SOPS and age

## Overview

The homelab platform uses **SOPS** and **age** for secret management.

This approach allows secret files to be:

- stored safely in Git in encrypted form
- decrypted only on authorized control machines
- consumed by Ansible during runtime
- managed using a modern, Git-friendly workflow

The current implementation uses:

- **SOPS** for structured secret file encryption
- **age** as the encryption backend
- the `community.sops` Ansible collection for secret consumption in playbooks [2]
- repository-level encryption rules defined in `.sops.yaml` [3]

---

## Purpose

The SOPS and age workflow exists to provide a consistent way to manage secrets used by the homelab platform, including:

- Git SSH private/public key material
- Kubernetes admin kubeconfig content
- future API tokens or credentials
- other automation secrets stored in YAML files

The design goal is to keep secrets:

- encrypted at rest in Git
- easy to manage operationally
- compatible with Ansible workflows
- aligned with later GitOps and platform automation practices

---

## Tool Roles

## SOPS

SOPS is used to:

- encrypt and decrypt structured files such as YAML
- preserve the structure of configuration files while encrypting secret values
- allow encrypted files to remain Git-friendly
- support editing with `sops <file>`

SOPS does **not** manage long-term private keys itself. It uses an encryption backend.

## age

age is the encryption backend currently used by SOPS in this lab.

age provides:

- public-key encryption
- a simple and modern key format
- easy integration with SOPS
- a small operational footprint compared to more complex alternatives

---

## Current Repository Configuration

The repository currently contains a `.sops.yaml` file at the root with creation rules matching all files ending in:

- `.sops.yml`
- `.sops.yaml`

These files are encrypted using a configured age recipient [3].

Example rule:

```yaml
creation_rules:
  - path_regex: .*\.sops\.ya?ml$
    age: >-
      age1...
```

This means any matching file can be encrypted with the configured age public key by default.

---

## Current Ansible Integration

The Ansible repository currently depends on the following collections:

- `community.general`
- `community.sops` [2]

The `community.sops` collection is used to allow Ansible playbooks to decrypt SOPS-encrypted files at runtime.

This allows encrypted YAML files to remain in the repository while still being consumable by playbooks on the control node.

---

## Current Control Node Assumptions

Secret decryption occurs on the **Ansible control node**, not on the managed host.

In the current design, the control node is normally `ops1`, and the Ansible configuration assumes:

- inventory path: `./inventory/lab/hosts.yml`
- remote user: `labadmin`
- SSH private key file: `~/.ssh/id_ed25519_ansible`
- roles path: `./roles`
- `become = True` with `sudo` escalation [1]

Any alternate machine used to run Ansible must also have the required SOPS and age tooling and access to the correct age private key.

---

## Encryption Model

The current model is:

1. create or edit a plaintext YAML secret file
2. encrypt the file with SOPS using the configured age recipient
3. commit the encrypted file to Git
4. decrypt the file only on authorized control machines

The private age key is **not** stored in Git and must exist only on trusted control machines.

---

## File Naming Convention

The current recommended convention is to store encrypted secret files using one of these suffixes:

- `*.sops.yml`
- `*.sops.yaml`

Examples:

- `group_vars/ops/git-ssh-key.sops.yaml`
- `group_vars/ops/kubeconfig.sops.yml`

This matches the repository encryption rules defined in `.sops.yaml` [3].

---

## Secret File Locations

Secrets should be stored close to the scope in which they are used.

Examples:

- `group_vars/ops/` for secrets used by the `ops` group
- `group_vars/k8s_cluster/` for cluster-wide secrets
- more specific paths as the repo grows

This keeps secret scope narrow and easier to reason about.

---

## Editing Secrets

Secrets should not be edited directly with a normal text editor once encrypted.

### Correct workflow

To edit an encrypted secret file:

```bash
sops group_vars/ops/kubeconfig.sops.yml
```

SOPS will:

- decrypt the file temporarily
- open it in the configured editor
- re-encrypt it on save

### Decrypt for viewing only

```bash
sops --decrypt group_vars/ops/kubeconfig.sops.yml
```

### Encrypt a new plaintext file in place

```bash
sops --encrypt --in-place group_vars/ops/kubeconfig.sops.yml
```

---

## Ansible Secret Loading Pattern

The current preferred integration pattern is **explicit loading in the playbook**.

Example:

```yaml
- name: Configure management hosts
  hosts: ops
  become: true
  vars:
    ops_tools_git_vars: "{{ lookup('community.sops.sops', playbook_dir ~ '/../group_vars/ops/git-ssh-key.sops.yaml') | from_yaml }}"
    ops_kubectl_kubeconfig_vars: "{{ lookup('community.sops.sops', playbook_dir ~ '/../group_vars/ops/kubeconfig.sops.yml') | from_yaml }}"
  roles:
    - baseline
    - ops_tools
    - ops_terraform
    - ops_kubectl
```

This pattern is currently preferred because it is:

- explicit
- easy to understand
- easy to debug
- well suited for learning the toolchain

---

## Control Node Requirements for Secret Decryption

Any control machine running Ansible with SOPS-encrypted files must have:

- `sops` installed
- `age` installed
- the correct age private key available locally
- the `community.sops` collection installed [2]

Without these, Ansible cannot decrypt secret files during playbook execution.

---

## Age Key Handling

The age private key is one of the most important secrets in the environment.

### Requirements

- it must never be committed to Git
- it must be stored only on trusted control machines
- it must be backed up securely
- losing it may make existing encrypted files undecryptable

### Operational implication

A fallback control machine used to rebuild `ops1` must also have access to the age private key if it needs to run Ansible against encrypted secret files.

---

## Security Practices

The following practices should be followed:

- keep secrets encrypted in Git at all times
- use SOPS rather than plaintext secret files
- use `no_log: true` on tasks that write secrets to managed hosts
- avoid printing decrypted secret variables in debug output
- keep secret file scope as narrow as practical
- back up the age private key securely
- document which control machines are trusted to hold the age private key

---

## Validation

A healthy SOPS and age setup should allow the following from the control node:

### Confirm SOPS is installed

```bash
sops --version
```

### Confirm age is installed

```bash
age --version
```

### Confirm encrypted files can be decrypted

```bash
sops --decrypt group_vars/ops/kubeconfig.sops.yml > /dev/null
```

### Confirm Ansible can use SOPS-encrypted variables

Run the relevant playbook successfully and confirm no secret-loading errors occur.

Because the current repo relies on `community.sops`, this collection must also be available [2].

---

## Current Benefits of This Design

Using SOPS with age provides the following advantages:

- secrets can live in Git safely in encrypted form
- the same pattern can be reused across infrastructure and platform repos
- Ansible can consume encrypted secrets at runtime
- the secret workflow is more modern and Git-friendly than ad hoc plaintext files
- the design scales better than storing local-only secret files with no standard structure

---

## Current Limitations

- decryption depends on local access to the age private key
- a fallback control machine must be prepared for secret-aware recovery operations
- secret loading is currently explicit in playbooks rather than abstracted into more automatic workflows
- some secret-dependent tasks are still evolving as the platform matures

These limitations are acceptable for the current homelab stage and align with the educational goals of the platform.

---

## Future Enhancements

Potential future improvements include:

- more standardized secret file naming across the repo
- stronger separation of secret scopes by group or role
- documented age key backup and rotation procedures
- additional CI validation around encrypted file structure
- future integration with GitOps-managed secrets for Kubernetes workloads

---

## Related Documents

- [Configuration Management](../operations/configuration-management.md)
- [`ops1` Management Host](../infrastructure/ops1-management-host.md)
- [Run Ansible Configuration Management](../runbooks/run-ansible-configuration-management.md)
- [Rebuild `ops1` Management Host](../runbooks/rebuild-ops1-management-host.md)
