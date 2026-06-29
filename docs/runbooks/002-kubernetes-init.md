# Runbook: Bootstrap Kubernetes Cluster with kubeadm

## Purpose

Bootstrap the homelab Kubernetes cluster manually using `kubeadm` on the prepared control plane and worker nodes.

This runbook captures the first manual cluster build and is intended to:

- document the working bootstrap process
- provide a repeatable procedure for future rebuilds
- serve as the basis for later Ansible automation

## Scope

Applies to the current homelab Kubernetes cluster built on:

- Ubuntu Server 24.04 LTS virtual machines
- containerd
- kubeadm
- Cilium CNI
- three control plane nodes
- three worker nodes

This runbook describes the initial successful manual bootstrap using `k8s-cp1` as the control plane endpoint.

## Prerequisites

- all six Kubernetes VMs exist and are reachable
- static IP addressing is configured and correct
- Ansible baseline configuration has been applied
- Kubernetes prerequisites are already installed
- containerd is installed and running
- kubeadm, kubelet, and kubectl are installed
- swap is disabled on all Kubernetes nodes
- time synchronization is healthy on all nodes
- SSH access to all nodes is working

## Current Node Inventory

| Hostname | Role | IP Address |
| -------- | ---- | ---------- |
| `k8s-cp1` | Control plane | `192.168.1.21` |
| `k8s-cp2` | Control plane | `192.168.1.22` |
| `k8s-cp3` | Control plane | `192.168.1.23` |
| `k8s-w1` | Worker | `192.168.1.31` |
| `k8s-w2` | Worker | `192.168.1.32` |
| `k8s-w3` | Worker | `192.168.1.33` |

## Design Notes

This first cluster bootstrap used:

- `k8s-cp1` directly as the Kubernetes API endpoint
- no virtual IP or load-balanced control plane endpoint yet
- `Cilium` as the CNI
- a manual bootstrap process to build understanding before automation

This means the first cluster build is educational and operationally useful, but the API endpoint design is not yet the final HA-style approach.

## Risks

- mistakes during `kubeadm init` can require reset and retry
- joining nodes with the wrong command or token will fail
- pod networking will not function until the CNI is installed
- using `k8s-cp1` directly as the API endpoint means API access depends on that node
- bootstrap retries may require cleanup of partial cluster state

## Step 1: Verify Node Readiness

Run the following checks on all Kubernetes nodes.

### Confirm swap is disabled

```bash
swapon --show
```

Expected result: no output.

### Confirm containerd is running

```bash
systemctl status containerd --no-pager
```

### Confirm kubeadm is installed

```bash
kubeadm version
```

### Confirm kubelet is installed

```bash
kubelet --version
```

### Confirm node hostname is correct

```bash
hostnamectl
```

### Confirm basic network reachability

Example from `k8s-cp1`:

```bash
ping -c 2 192.168.1.22
ping -c 2 192.168.1.23
ping -c 2 192.168.1.31
ping -c 2 192.168.1.32
ping -c 2 192.168.1.33
```

## Step 2: Prepare kubeadm Configuration on `k8s-cp1`

SSH to the first control plane node:

```bash
ssh labadmin@192.168.1.21
```

Create the kubeadm config file:

```bash
sudo nano /tmp/kubeadm-init.yaml
```

Use the following configuration:

```yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "192.168.1.21:6443"
networking:
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
```

## Step 3: Initialize the First Control Plane

Run:

```bash
sudo kubeadm init --config /tmp/kubeadm-init.yaml
```

Optional: capture the output for documentation:

```bash
sudo kubeadm init --config /tmp/kubeadm-init.yaml | tee ~/kubeadm-init.out
```

Expected result:

- cluster initialization completes successfully
- kubeadm prints post-init instructions
- join information is displayed

## Step 4: Configure kubectl on `k8s-cp1`

As the normal user on `k8s-cp1`, configure kubeconfig access:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown "$(id -u)":"$(id -g)" $HOME/.kube/config
```

Validate kubectl access:

```bash
kubectl get nodes
```

Expected result:

- `k8s-cp1` appears
- it may initially report `NotReady` until the CNI is installed

## Step 5: Install Cilium CLI on `k8s-cp1`

Install the Cilium CLI:

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all \
  https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
```

Validate installation:

```bash
cilium version
```

## Step 6: Install Cilium CNI

Install Cilium into the cluster:

```bash
cilium install
```

Wait for readiness:

```bash
cilium status --wait
```

Then validate node state again:

```bash
kubectl get nodes
```

Expected result:

- `k8s-cp1` becomes `Ready`

## Step 7: Generate Join Commands

### Generate the worker join command

```bash
kubeadm token create --print-join-command
```

Save the output.

### Generate the certificate key for joining additional control planes

```bash
sudo kubeadm init phase upload-certs --upload-certs
```

Save the certificate key shown in the output.

### Build the control plane join command

Run again if needed:

```bash
kubeadm token create --print-join-command
```

Append:

```text
--control-plane --certificate-key <certificate-key>
```

The final control plane join command should look similar to:

```bash
sudo kubeadm join 192.168.1.21:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <certificate-key>
```

## Step 8: Join Additional Control Plane Nodes

SSH to `k8s-cp2`:

```bash
ssh labadmin@192.168.1.22
```

Run the full control-plane join command with `sudo`.

Repeat the process on `k8s-cp3`:

```bash
ssh labadmin@192.168.1.23
```

Run the same control-plane join command with `sudo`.

After each join, validate from `k8s-cp1`:

```bash
kubectl get nodes
```

Expected result:

- additional control plane nodes appear
- they may take some time to become `Ready`

## Step 9: Join Worker Nodes

SSH to each worker node and run the worker join command with `sudo`.

Example for `k8s-w1`:

```bash
ssh labadmin@192.168.1.31
sudo kubeadm join 192.168.1.21:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Repeat for:

- `k8s-w2`
- `k8s-w3`

After joining each worker, validate from `k8s-cp1`:

```bash
kubectl get nodes
```

## Step 10: Validate Cluster Health

From `k8s-cp1`, validate the final cluster state.

### Check nodes

```bash
kubectl get nodes -o wide
```

Expected result:

- all 3 control planes present
- all 3 workers present
- all nodes `Ready`

### Check cluster pods

```bash
kubectl get pods -A
```

Expected result:

- control plane static pods healthy
- CoreDNS healthy
- Cilium components healthy

### Check Cilium status

```bash
cilium status
```

Expected result:

- Cilium reports healthy status

## Step 11: Confirm Successful Result

Successful result for this bootstrap was:

```text
NAME      STATUS   ROLES           AGE   VERSION
k8s-cp1   Ready    control-plane   54m   v1.36.1
k8s-cp2   Ready    control-plane   21m   v1.36.1
k8s-cp3   Ready    control-plane   17m   v1.36.1
k8s-w1    Ready    worker          19m   v1.36.1
k8s-w2    Ready    worker          18m   v1.36.1
k8s-w3    Ready    worker          16m   v1.36.1
```

## Step 12: Optional Post-Bootstrap Tasks

After the cluster is healthy, recommended next tasks include:

- configure `kubectl` access from `ops1`
- document the bootstrap commands and configuration used
- install `metrics-server`
- install `MetalLB`
- install `ingress-nginx`
- later automate the bootstrap process with Ansible

## Cleanup / Recovery Notes

If a node needs to be reset before retrying:

```bash
sudo kubeadm reset -f
sudo rm -rf /etc/cni/net.d
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

Use this carefully and only on nodes being rebuilt or rejoined.

## Validation Checklist

- [ ] `k8s-cp1` initialized successfully
- [ ] kubeconfig works on `k8s-cp1`
- [ ] Cilium installed successfully
- [ ] `k8s-cp2` joined as control plane
- [ ] `k8s-cp3` joined as control plane
- [ ] `k8s-w1` joined as worker
- [ ] `k8s-w2` joined as worker
- [ ] `k8s-w3` joined as worker
- [ ] all nodes report `Ready`
- [ ] `kubectl get pods -A` shows healthy cluster services

## Follow-Up

After this manual bootstrap:

- update cluster design documentation
- record the exact bootstrap values used
- create an automation plan for Ansible-based bootstrap
- decide whether to keep the direct `k8s-cp1` endpoint or later rebuild with a stable control plane virtual endpoint such as `kube-vip`

## References

- [Kubernetes Cluster Bootstrap Design](../kubernetes/cluster-bootstrap-design.md)
- [Node Bootstrap](../operations/node-bootstrap.md)
- [Configuration Management](../operations/configuration-management.md)
