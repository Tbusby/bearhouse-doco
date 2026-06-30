# Runbook: Rebuild `ops1` Management Host

## Purpose

Rebuild the `ops1` management host from infrastructure-as-code and configuration management after intentional destruction, failure, or replacement.

This runbook exists to ensure that `ops1` can be recreated in a controlled and repeatable way without depending on the original `ops1` instance.

`ops1` is a critical management host used for:

- Terraform execution
- Ansible execution
- Git operations
- cluster administration tooling
- secret management tooling

Because of that role, rebuilding it requires a temporary alternate control environment.

---

## Scope

Applies to the rebuild of the `ops1` VM in the homelab environment.

This runbook assumes:

- Proxmox is healthy and reachable
- Terraform is used to provision `ops1`
- Ansible is used to configure `ops1`
- SOPS is used for encrypted secrets
- Ansible is normally configured to use the `labadmin` user, a specific inventory file, a specific private key, and `sudo` privilege escalation [1]
- the Ansible repository uses the `community.general` and `community.sops` collections [2]

---

## Rebuild Strategy

`ops1` cannot rebuild itself once it has been destroyed.

Therefore, the rebuild must be performed from a **temporary alternate control machine**, such as:

- a laptop
- another Linux workstation
- a temporary utility VM

This temporary control machine must be able to run both Terraform and Ansible and must have access to the required SSH keys and SOPS age private key.

---

## Prerequisites

Before destroying or rebuilding `ops1`, confirm the following are available on the temporary control machine.

### Required software

- Git
- Terraform
- Ansible
- `sops`
- `age`

### Required repositories

- Terraform infrastructure repository
- Ansible configuration repository

### Required credentials and keys

- SSH private key used for Ansible connectivity
- Proxmox API credentials or token used by Terraform
- SOPS age private key required to decrypt encrypted secrets
- any Git SSH key material if the rebuild process depends on private repository access

### Required Ansible dependencies

Install the required collections defined in `requirements.yml`:

```bash
ansible-galaxy collection install -r requirements.yml
```

The current required collections are:

- `community.general`
- `community.sops` [2]

### Required Ansible behavior assumptions

The current Ansible configuration expects:

- inventory at `./inventory/lab/hosts.yml`
- remote user `labadmin`
- private key file `~/.ssh/id_ed25519_ansible`
- roles path `./roles`
- `become = True`
- `sudo` escalation without password prompts [1]

Make sure the temporary control machine is prepared accordingly.

---

## Risks

- destroying `ops1` removes the normal Terraform/Ansible execution environment
- secrets cannot be decrypted if the age private key is missing
- Ansible cannot connect if the expected SSH private key is unavailable
- rebuilding from an unprepared temporary machine may delay recovery
- if `ops1` is used for cluster administration, operational access may be reduced until it is rebuilt

---

## Required Temporary Control Environment Checklist

Before destroying `ops1`, verify all of the following on the temporary control machine:

- [ ] Git installed
- [ ] Terraform installed
- [ ] Ansible installed
- [ ] `sops` installed
- [ ] `age` installed
- [ ] age private key available locally
- [ ] Ansible SSH key available at the expected path or configuration adjusted
- [ ] Terraform repository cloned
- [ ] Ansible repository cloned
- [ ] Ansible collections installed from `requirements.yml`
- [ ] ability to reach Proxmox API
- [ ] ability to SSH to existing lab nodes if needed

---

## Step 1: Prepare Temporary Control Machine

Clone the required repositories.

### Clone Terraform repository

```bash
git clone <terraform-repo-url>
```

### Clone Ansible repository

```bash
git clone <ansible-repo-url>
```

If the repositories already exist locally, confirm they are current:

```bash
git pull
```

### Install Ansible collections

From the Ansible repository root:

```bash
ansible-galaxy collection install -r requirements.yml
```

### Confirm SOPS configuration exists

The Ansible repository currently uses a `.sops.yaml` file with age-based encryption rules for `*.sops.yml` and `*.sops.yaml` files [3].

Confirm the age private key is available on the temporary control machine and can decrypt the relevant secrets.

---

## Step 2: Verify Terraform Access

From the Terraform repository:

```bash
cd environments/lab
terraform init
terraform plan
```

Confirm Terraform can:

- initialize successfully
- authenticate to Proxmox
- read the existing environment state

If Terraform uses remote secrets or token files, verify those are available on the temporary control machine.

---

## Step 3: Verify Ansible Access

From the Ansible repository root, test inventory access:

```bash
ansible-inventory --graph
```

Test SSH connectivity to an existing host if appropriate:

```bash
ansible all -m ping
```

If `ops1` has already been destroyed, this may be limited to remaining hosts.

Confirm the expected SSH private key exists at the configured path or adjust your environment accordingly. The current Ansible config expects:

```text
~/.ssh/id_ed25519_ansible
```

as the private key file [1].

---

## Step 4: Destroy `ops1` if Performing an Intentional Rebuild

Only do this after the temporary control machine is confirmed working.

From the Terraform repository environment directory:

```bash
terraform destroy -target=<ops1-resource-address>
```

Or use the current module/resource naming convention in your Terraform configuration.

If you are rebuilding after an unexpected failure, skip this step and proceed directly to reprovisioning.

---

## Step 5: Recreate `ops1` with Terraform

From the Terraform environment directory:

```bash
terraform apply
```

Or, if targeting only `ops1`:

```bash
terraform apply -target=<ops1-resource-address>
```

Wait for:

- VM creation to complete
- cloud-init to finish
- network connectivity to come up

Confirm the VM exists in Proxmox and receives the expected IP address.

---

## Step 6: Confirm Basic Access to the Rebuilt `ops1`

Test SSH access:

```bash
ssh labadmin@<ops1-ip>
```

At this stage, `ops1` may only have the base image and cloud-init configuration applied.

Do not assume all tools are installed until Ansible configuration has completed.

---

## Step 7: Run Ansible Against `ops1`

From the Ansible repository, run the `ops1` playbook or the role-targeted playbook used to configure the management host.

Example pattern:

```bash
ansible-playbook playbooks/ops.yml
```

This should apply at least:

- baseline host configuration
- ops tooling
- Terraform installation/configuration if managed by Ansible
- Git configuration
- SOPS/age-related tooling if managed there
- SSH key deployment as needed

Because Ansible uses `community.sops`, the temporary control machine must be able to decrypt encrypted secret files during this step [2] [3].

---

## Step 8: Validate `ops1` Tooling

After the playbook completes, SSH into `ops1` and verify the expected tooling is available.

### Verify Git

```bash
git --version
ssh -T git@github.com
cd ~
git clone git@github.com:Tbusby/bearhouse-infra-ansible.git
git clone git@github.com:Tbusby/bearhouse-infra-proxmox.git
```

### Verify Terraform

```bash
cd /home/labadmin/bearhouse-infra-proxmox
terraform version
terraform plan
```

### Verify Ansible

```bash
cd /home/labadmin/bearhouse-infra-ansible
ansible --version
ansible all -m ping
ansible-inventory --graph
```

### Verify SOPS

Manually copy the sops-age key file from a backup to -
`$HOME/.conf/sops/age/keys.txt`

```bash
sops --version
sops /home/labadmin/bearhouse-infra-ansible/group_vars/ops/kubeconfig.sops.yaml
```

### Verify age

```bash
age --version
```

### Verify Kubernetes admin tooling

```bash
kubectl version --client
kubectl get nodes
kubectl get pods -A
helm version
helm ls -A
cilium status
```

---

## Step 9: Return Operational Control to `ops1`

Once `ops1` is fully rebuilt and validated, return normal Terraform and Ansible operations to it.

Recommended checks before switching back:

- [ ] SSH access works
- [ ] Git access works
- [ ] Terraform works
- [ ] Ansible works
- [ ] SOPS decryption works
- [ ] expected SSH keys are present
- [ ] required repositories are cloned and current
- [ ] kubeconfig and cluster admin tools are available as expected

---

## Validation

The rebuild is considered successful when:

- `ops1` exists in Proxmox
- `ops1` is reachable by SSH
- Terraform can run from `ops1`
- Ansible can run from `ops1`
- SOPS can decrypt the required files from `ops1`
- Git operations work from `ops1`
- Expected kubernetes admin tooling is present and functional

---

## Rollback / Fallback

If the rebuild fails:

- continue operating from the temporary control machine
- correct Terraform or Ansible issues there
- re-run Terraform and/or Ansible as needed
- do not destroy additional management capability until `ops1` is stable

If SOPS decryption fails:

- verify that the age private key exists on the temporary control machine
- verify that the encrypted files match the configured `.sops.yaml` rules [3]

If Ansible connectivity fails:

- verify the expected SSH key exists at the configured path
- verify the inventory path and host definitions are correct
- confirm the `labadmin` user exists and can `sudo` without password prompts as expected by current configuration [1]

---

## Best Practices

- never destroy `ops1` until the alternate control machine is confirmed ready
- keep the Terraform and Ansible repositories usable from more than one machine
- back up the age private key securely
- keep Ansible collections installable from `requirements.yml` [2]
- document any local machine assumptions that are not yet automated
- periodically test rebuilding `ops1` so the runbook remains valid

---

## Follow-Up

After a successful rebuild:

- update documentation if any steps changed
- record any issues encountered
- improve automation where friction was found
- consider whether more of the `ops1` setup should be codified or validated automatically

---

## References

- Ansible configuration assumptions [1]
- Required Ansible collections [2]
- SOPS age encryption rules [3]
