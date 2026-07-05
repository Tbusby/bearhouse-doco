# Runbook: Install ingress-nginx

## Purpose

Install and validate `ingress-nginx` on the Kubernetes cluster so HTTP and HTTPS applications can be exposed through a shared ingress controller.

This runbook is intended for the initial manual installation and validation of `ingress-nginx` from `ops1`.

## Scope

Applies to the current kubeadm-based homelab Kubernetes cluster.

This runbook covers:

- installing `ingress-nginx` from the upstream manifest
- validating the ingress controller Service
- confirming MetalLB assigns an external IP
- deploying a test application
- creating a test `Ingress` resource
- validating hostname-based routing from a LAN client

This runbook does not cover:

- Helm-based installation
- GitOps or Ansible automation
- TLS or `cert-manager`
- production DNS integration

## Prerequisites

- Cluster is healthy and all nodes are `Ready`
- `ops1` has working cluster admin access
- `kubectl` works from `ops1`
- MetalLB is installed and functioning
- At least one free IP remains in the MetalLB pool
- A LAN client is available for hostname-based testing
- User understands `/etc/hosts` can be used temporarily for the initial test

## Planned Values

| Item | Value |
| ---- | ----- |
| Namespace | `ingress-nginx` |
| Install manifest | upstream static provider manifest |
| Ingress class | `nginx` |
| Test application | `whoami` |
| Test hostname | `whoami.lab.local` |

## Risks

- installing ingress before MetalLB is functioning
- confusing ingress controller behavior with Service exposure behavior
- testing without first confirming the ingress controller has an external IP
- forgetting that hostname-based routing depends on client name resolution
- trying to debug TLS before plain HTTP ingress is working

## Procedure

### 1. Prepare

- [ ] SSH to `ops1`
- [ ] Create and enter a working directory:

  ```bash
  mkdir -p ~/src/manual-installs/ingress-nginx
  cd ~/src/manual-installs/ingress-nginx
  ```

### 2. Install ingress-nginx

- [ ] Apply the upstream manifest:

  ```bash
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
  ```

### 3. Validate the installation

- [ ] Check namespace resources:

  ```bash
  kubectl get all -n ingress-nginx
  ```

- [ ] Watch the pods until the controller is running and admission jobs complete:

  ```bash
  kubectl get pods -n ingress-nginx -w
  ```

- [ ] Check the ingress controller Service:

  ```bash
  kubectl get svc -n ingress-nginx
  ```

- [ ] Confirm the controller Service is type `LoadBalancer`
- [ ] Confirm the controller Service receives an external IP from the MetalLB pool

### 4. Validate ingress class

- [ ] Check ingress classes:

  ```bash
  kubectl get ingressclass
  ```

- [ ] Confirm the `nginx` ingress class exists

### 5. Deploy a simple test application

- [ ] Create `whoami.yaml`:

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: whoami
    namespace: default
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: whoami
    template:
      metadata:
        labels:
          app: whoami
      spec:
        containers:
          - name: whoami
            image: traefik/whoami:latest
            ports:
              - containerPort: 80
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: whoami
    namespace: default
  spec:
    selector:
      app: whoami
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
  ```

- [ ] Apply the test app:

  ```bash
  kubectl apply -f whoami.yaml
  ```

- [ ] Confirm the Deployment, pods, and Service exist:

  ```bash
  kubectl get deploy,po,svc -l app=whoami -o wide
  ```

### 6. Create a test Ingress

- [ ] Create `whoami-ingress.yaml`:

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: whoami
    namespace: default
  spec:
    ingressClassName: nginx
    rules:
      - host: whoami.lab.local
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: whoami
                  port:
                    number: 80
  ```

- [ ] Apply the ingress:

  ```bash
  kubectl apply -f whoami-ingress.yaml
  ```

- [ ] Validate the ingress resource:

  ```bash
  kubectl get ingress -A
  kubectl describe ingress whoami -n default
  ```

### 7. Configure client hostname resolution

- [ ] Find the ingress controller external IP:

  ```bash
  kubectl get svc -n ingress-nginx
  ```

- [ ] On a LAN client, add an `/etc/hosts` entry mapping the ingress IP to:

  ```text
  whoami.lab.local
  ```

### 8. Validate external routing

- [ ] From the LAN client, test the hostname:

  ```bash
  curl http://whoami.lab.local
  ```

- [ ] Optionally open the URL in a browser:

  ```text
  http://whoami.lab.local
  ```

- [ ] Confirm the response comes from the `whoami` application

### 9. Record the final state

- [ ] Record the ingress controller external IP
- [ ] Record the test hostname used
- [ ] Record that `/etc/hosts` was used for initial manual validation
- [ ] Update architecture and Kubernetes documentation

## Validation

The installation is successful when:

- the ingress controller is running
- the ingress controller Service is type `LoadBalancer`
- the Service has an external IP from MetalLB
- the `nginx` ingress class exists
- the test application is reachable by hostname through the ingress controller from a LAN client

## Rollback / Fallback

- Delete the test Ingress if only routing cleanup is needed
- Delete the test application and Service if only workload cleanup is needed
- Delete the ingress-nginx manifest if the controller must be fully removed
- Do not proceed to TLS or `cert-manager` until plain HTTP ingress is working reliably

## Follow-Up

- Replace `/etc/hosts` with proper DNS later
- Document the ingress controller external IP strategy
- Add automation for ingress-nginx installation
- Proceed to `cert-manager`

## Notes

- `Ingress` routes to Services, not directly to pods
- MetalLB provides the ingress IP; `ingress-nginx` provides HTTP/HTTPS routing
- Most future web applications should use ingress rather than separate `LoadBalancer` Services
