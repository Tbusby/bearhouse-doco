# Runbook: Install Argo CD

## Purpose

Install and validate Argo CD on the Kubernetes cluster so GitOps-managed applications can be reconciled from Git.

This runbook is intended for the initial manual installation and validation of Argo CD from `ops1`.

## Scope

Applies to the current kubeadm-based homelab Kubernetes cluster.

This runbook covers:

- installing Argo CD from the upstream manifest
- validating core control plane components
- retrieving bootstrap admin credentials
- validating initial access
- exposing Argo CD through ingress with TLS
- deploying and validating a first GitOps-managed demo application

This runbook does not cover:

- full platform-wide GitOps conversion
- private Git repository credential management
- app-of-apps patterns
- multi-cluster Argo CD usage

## Prerequisites

- Cluster is healthy and all nodes are `Ready`
- `ops1` has working cluster admin access
- `kubectl` works from `ops1`
- `ingress-nginx` is installed and working
- `cert-manager` is installed and working
- Internal CA-based TLS workflow already works
- A Git repository is available for the first demo application

## Planned Values

| Item | Value |
| ---- | ----- |
| Namespace | `argocd` |
| Install source | upstream Argo CD manifest |
| Ingress hostname | `argocd.lab.local` |
| TLS Secret | `argocd-tls` |
| Certificate issuer | `lab-ca` |
| First demo app | `whoami` |
| Demo app namespace | `whoami` |

## Risks

- trying to GitOps-convert too much at once
- exposing Argo CD through ingress before validating the service internally
- forgetting that pre-existing manual resources can conflict with GitOps-managed resources
- using a private repo without first configuring repo credentials
- not documenting the ownership transition from manual to GitOps-managed resources

## Procedure

### 1. Prepare

- [ ] SSH to `ops1`
- [ ] Create and enter a working directory:

  ```bash
  mkdir -p ~/src/manual-installs/argocd
  cd ~/src/manual-installs/argocd
  ```

### 2. Create the namespace

- [ ] Create the namespace:

  ```bash
  kubectl create namespace argocd
  ```

### 3. Install Argo CD

- [ ] Apply the upstream manifest:

  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```

### 4. Validate the installation

- [ ] Check Argo CD resources:

  ```bash
  kubectl get all -n argocd
  ```

- [ ] Watch pods until core components are running:

  ```bash
  kubectl get pods -n argocd -w
  ```

### 5. Retrieve bootstrap admin credentials

- [ ] Get the initial admin password:

  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret \
    -o jsonpath="{.data.password}" | base64 -d && echo
  ```

- [ ] Record the initial password securely for bootstrap use

### 6. Validate initial access with port-forward

- [ ] Port-forward the Argo CD server:

  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  ```

- [ ] Open:

  ```text
  https://localhost:8080
  ```

- [ ] Log in with:
  - username: `admin`
  - password from `argocd-initial-admin-secret`

### 7. Create a TLS certificate for ingress exposure

- [ ] Create `argocd-certificate.yaml`:

  ```yaml
  apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    name: argocd-tls
    namespace: argocd
  spec:
    secretName: argocd-tls
    commonName: argocd.lab.local
    dnsNames:
      - argocd.lab.local
    issuerRef:
      name: lab-ca
      kind: ClusterIssuer
      group: cert-manager.io
  ```

- [ ] Apply the certificate:

  ```bash
  kubectl apply -f argocd-certificate.yaml
  ```

- [ ] Validate the certificate and Secret:

  ```bash
  kubectl get certificate -n argocd
  kubectl describe certificate argocd-tls -n argocd
  kubectl get secret -n argocd argocd-tls
  ```

### 8. Create the Argo CD ingress

- [ ] Create `argocd-ingress.yaml`:

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: argocd
    namespace: argocd
    annotations:
      nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
  spec:
    ingressClassName: nginx
    tls:
      - hosts:
          - argocd.lab.local
        secretName: argocd-tls
    rules:
      - host: argocd.lab.local
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: argocd-server
                  port:
                    number: 443
  ```

- [ ] Apply the ingress:

  ```bash
  kubectl apply -f argocd-ingress.yaml
  ```

- [ ] Validate the ingress:

  ```bash
  kubectl get ingress -n argocd
  kubectl describe ingress argocd -n argocd
  ```

### 9. Configure client hostname resolution

- [ ] Find the ingress controller external IP:

  ```bash
  kubectl get svc -n ingress-nginx
  ```

- [ ] Map `argocd.lab.local` to that IP on a client using `/etc/hosts`

### 10. Validate Argo CD through ingress

- [ ] Open:

  ```text
  https://argocd.lab.local
  ```

- [ ] Confirm login works through ingress
- [ ] Optionally change the initial admin password after validation

### 11. Prepare a first GitOps-managed demo app

- [ ] Create a simple demo app manifest set in a Git repo
- [ ] Include a path such as:

  ```text
  apps/whoami
  ```

- [ ] Include at least:
  - Deployment
  - Service
  - Certificate
  - Ingress

### 12. Create the first Argo CD Application

- [ ] Create an `Application` resource pointing to the Git repo and path
- [ ] Use `CreateNamespace=true` if the destination namespace should be created automatically
- [ ] Apply the `Application` resource with `kubectl`

### 13. Validate the first GitOps-managed app

- [ ] Check Argo CD applications:

  ```bash
  kubectl get applications -n argocd
  kubectl describe application whoami -n argocd
  ```

- [ ] Check deployed workload resources:

  ```bash
  kubectl get all -n whoami
  kubectl get ingress -n whoami
  kubectl get certificate -n whoami
  ```

- [ ] Confirm the app is reachable through ingress

### 14. Resolve any ownership conflicts

- [ ] If a manually created ingress or workload already exists with the same hostname/path, remove or rename the manual resource
- [ ] Re-sync the Argo CD application after the conflict is removed
- [ ] Confirm the GitOps-managed resource becomes the canonical version

### 15. Record the final state

- [ ] Record the Argo CD ingress hostname
- [ ] Record the Git repo used for the first app
- [ ] Record the initial GitOps-managed namespace and app
- [ ] Record any ownership conflict encountered and how it was resolved
- [ ] Update architecture and Kubernetes documentation

## Validation

The installation is successful when:

- Argo CD core components are running
- the UI is reachable
- ingress and TLS work for Argo CD
- a demo application can be deployed from Git
- the application reaches `Healthy` and `Synced`
- Git and cluster state are visibly connected through Argo CD

## Rollback / Fallback

- Delete the demo `Application` resource if only test app cleanup is required
- Delete the Argo CD ingress and certificate if only external access cleanup is required
- Delete the Argo CD namespace and resources if the controller must be fully removed
- Do not try to GitOps-convert broader platform components until the first application workflow is stable and understood

## Follow-Up

- Decide whether to enable automated sync or self-heal on selected apps
- Define a clearer long-term GitOps repo structure
- Expand GitOps adoption gradually
- Document ownership rules for GitOps-managed resources
- Consider future patterns such as app-of-apps only after the basic model is fully understood

## Notes

- Argo CD manages Kubernetes application state; it does not replace Terraform or Ansible
- Pre-existing manual resources can conflict with GitOps-managed resources
- Internal CA trust behavior still applies to the Argo CD UI and app TLS endpoints
- Start with small GitOps scope and expand intentionally
