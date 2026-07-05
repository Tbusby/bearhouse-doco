# Runbook: Install cert-manager

## Purpose

Install and validate `cert-manager` on the Kubernetes cluster so certificates can be issued and renewed automatically for ingress and workloads.

This runbook is intended for the initial manual installation and validation of `cert-manager` from `ops1`.

## Scope

Applies to the current kubeadm-based homelab Kubernetes cluster.

This runbook covers:

- installing `cert-manager` from the upstream manifest
- creating a bootstrap self-signed `ClusterIssuer`
- creating an internal root CA certificate
- creating a CA-backed `ClusterIssuer`
- issuing a test application certificate
- validating ingress TLS using the generated Secret

This runbook does not cover:

- Helm-based installation
- GitOps or Ansible automation
- public domain integration
- ACME / Let’s Encrypt configuration

## Prerequisites

- Cluster is healthy and all nodes are `Ready`
- `ops1` has working cluster admin access
- `kubectl` works from `ops1`
- `ingress-nginx` is installed and functioning
- A test ingress hostname already exists
- User understands the current install is using an internal CA model rather than public trust

## Planned Values

| Item | Value |
| ---- | ----- |
| Namespace | `cert-manager` |
| Install manifest | upstream release manifest |
| Bootstrap issuer | `selfsigned-bootstrap` |
| Internal CA certificate | `lab-root-ca` |
| Internal CA Secret | `lab-root-ca-secret` |
| CA ClusterIssuer | `lab-ca` |
| Test certificate | `whoami-tls` |
| Test hostname | `whoami.lab.local` |

## Risk

- assuming internal CA certificates will be browser-trusted automatically
- confusing certificate issuance with ingress routing
- creating issuers without validating cert-manager health first
- using TLS before plain HTTP ingress has already been validated
- leaving public ACME plans undocumented even if intentionally deferred

## Procedure

### 1. Prepare

- [ ] SSH to `ops1`
- [ ] Create and enter a working directory:

  ```bash
  mkdir -p ~/src/manual-installs/cert-manager
  cd ~/src/manual-installs/cert-manager
  ```

### 2. Install cert-manager

- [ ] Apply the upstream release manifest:

  ```bash
  kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.1/cert-manager.yaml
  ```

### 3. Validate the installation

- [ ] Check pods:

  ```bash
  kubectl get pods -n cert-manager
  ```

- [ ] Confirm the controller, webhook, and cainjector pods are running
- [ ] Check CRDs:

  ```bash
  kubectl get crds | grep cert-manager
  ```

### 4. Create the bootstrap self-signed ClusterIssuer

- [ ] Create `selfsigned-clusterissuer.yaml`:

  ```yaml
  apiVersion: cert-manager.io/v1
  kind: ClusterIssuer
  metadata:
    name: selfsigned-bootstrap
  spec:
    selfSigned: {}
  ```

- [ ] Apply the issuer:

  ```bash
  kubectl apply -f selfsigned-clusterissuer.yaml
  ```

- [ ] Validate the issuer:

  ```bash
  kubectl get clusterissuer
  kubectl describe clusterissuer selfsigned-bootstrap
  ```

### 5. Create the internal root CA certificate

- [ ] Create `lab-root-ca.yaml`:

  ```yaml
  apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    name: lab-root-ca
    namespace: cert-manager
  spec:
    isCA: true
    commonName: lab-root-ca
    secretName: lab-root-ca-secret
    privateKey:
      algorithm: RSA
      size: 2048
    issuerRef:
      name: selfsigned-bootstrap
      kind: ClusterIssuer
      group: cert-manager.io
  ```

- [ ] Apply the certificate:

  ```bash
  kubectl apply -f lab-root-ca.yaml
  ```

- [ ] Validate the certificate and Secret:

  ```bash
  kubectl get certificate -n cert-manager
  kubectl describe certificate lab-root-ca -n cert-manager
  kubectl get secret -n cert-manager lab-root-ca-secret
  ```

### 6. Create the CA-backed ClusterIssuer

- [ ] Create `lab-ca-clusterissuer.yaml`:

  ```yaml
  apiVersion: cert-manager.io/v1
  kind: ClusterIssuer
  metadata:
    name: lab-ca
  spec:
    ca:
      secretName: lab-root-ca-secret
  ```

- [ ] Apply the issuer:

  ```bash
  kubectl apply -f lab-ca-clusterissuer.yaml
  ```

- [ ] Validate the issuer:

  ```bash
  kubectl get clusterissuer
  kubectl describe clusterissuer lab-ca
  ```

### 7. Issue a test application certificate

- [ ] Create `whoami-certificate.yaml`:

  ```yaml
  apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    name: whoami-tls
    namespace: default
  spec:
    secretName: whoami-tls
    commonName: whoami.lab.local
    dnsNames:
      - whoami.lab.local
    issuerRef:
      name: lab-ca
      kind: ClusterIssuer
      group: cert-manager.io
  ```

- [ ] Apply the certificate:

  ```bash
  kubectl apply -f whoami-certificate.yaml
  ```

- [ ] Validate the certificate and Secret:

  ```bash
  kubectl get certificate -n default
  kubectl describe certificate whoami-tls -n default
  kubectl get secret -n default whoami-tls
  ```

### 8. Update ingress to use TLS

- [ ] Edit the existing ingress resource:

  ```bash
  kubectl edit ingress whoami -n default
  ```

- [ ] Add a TLS section referencing the generated Secret:

  ```yaml
  tls:
    - hosts:
        - whoami.lab.local
      secretName: whoami-tls
  ```

- [ ] Validate the ingress:

  ```bash
  kubectl describe ingress whoami -n default
  ```

### 9. Validate HTTPS

- [ ] From a client that resolves `whoami.lab.local`, test HTTPS:

  ```bash
  curl -k https://whoami.lab.local
  ```

- [ ] Optionally inspect the served certificate:

  ```bash
  openssl s_client -connect whoami.lab.local:443 -servername whoami.lab.local
  ```

### 10. Record the final state

- [ ] Record the bootstrap issuer name
- [ ] Record the CA issuer name
- [ ] Record the TLS Secret used by ingress
- [ ] Record that the current trust model is internal CA only
- [ ] Update architecture and Kubernetes documentation

## Validation

The installation is successful when:

- cert-manager controller, webhook, and cainjector are running
- cert-manager CRDs exist
- `ClusterIssuer` resources are `Ready`
- `Certificate` resources are `Ready`
- the generated TLS Secret exists
- ingress successfully terminates HTTPS using the generated Secret

## Rollback / Fallback

- Delete the application certificate if only test certificate cleanup is required
- Delete the CA issuer and CA certificate if the internal CA workflow must be removed
- Delete the cert-manager manifest if the full installation must be removed
- Do not proceed to public ACME workflows until the internal CA model is fully understood and documented

## Follow-Up

- Decide whether to keep the internal CA for lab-only use cases
- Add automation for cert-manager installation and issuer creation
- Revisit public domain and ACME integration later if external trust becomes desirable
- Continue to the next platform layer

## Notes

- The current model provides automated TLS but not public browser trust
- `cert-manager` manages certificate lifecycle; ingress consumes the generated Secret
- Public domain and Let’s Encrypt integration are intentionally deferred
