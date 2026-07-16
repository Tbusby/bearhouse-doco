# Runbook: Install node_exporter on Proxmox Host

## Purpose

Install and validate `node_exporter` manually on the Proxmox host so Prometheus can collect host-level metrics from the hypervisor.

This runbook is intended for the Proxmox host only, since the Proxmox host is not currently managed by Ansible.

## Scope

Applies to the Proxmox VE host used by the homelab platform.

This runbook covers:

- downloading and installing `node_exporter`
- creating a dedicated service user
- installing a systemd service
- starting and validating the exporter
- confirming metrics are available on port `9100`

This runbook does not cover:

- Prometheus scrape configuration
- Grafana dashboards
- Proxmox API-specific exporters
- Ansible automation for non-Proxmox hosts

## Prerequisites

- Proxmox host is healthy and reachable
- SSH access to the Proxmox host is available
- Internet access is available from the Proxmox host
- A current `node_exporter` release version has been selected and pinned
- User understands this is one of the few acceptable additional host agents on the Proxmox host

## Planned Values

| Item | Value |
| ---- | ----- |
| Service user | `node_exporter` |
| Binary path | `/usr/local/bin/node_exporter` |
| Config directory | `/etc/node_exporter` |
| Textfile collector directory | `/var/lib/node_exporter/textfile_collector` |
| Systemd unit | `/etc/systemd/system/node_exporter.service` |
| Listen address | `:9100` |
| Version | `1.12.1` |
| Architecture | `linux-amd64` |

## Risks

- downloading an unpinned or incorrect upstream release
- installing additional software on the Proxmox host without documenting it
- forgetting to validate the exporter locally before adding it to Prometheus
- leaving the service unmanaged or undocumented
- exposing port `9100` more broadly than intended

## Procedure

### 1. Prepare

- [ ] SSH to the Proxmox host
- [ ] Choose and record the exact pinned `node_exporter` version
- [ ] Set shell variables for convenience:

  ```bash
  VERSION="1.12.1"
  ARCH="linux-amd64"
  ARCHIVE="node_exporter-${VERSION}.${ARCH}.tar.gz"
  URL="https://github.com/prometheus/node_exporter/releases/download/v${VERSION}/${ARCHIVE}"
  ```

### 2. Create the service account

- [ ] Create the `node_exporter` system user:

  ```bash
  sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter
  ```

- [ ] Confirm the user exists:

  ```bash
  id node_exporter
  ```

### 3. Create directories

- [ ] Create required directories:

  ```bash
  sudo mkdir -p /etc/node_exporter
  sudo mkdir -p /var/lib/node_exporter/textfile_collector
  ```

- [ ] Set ownership for the textfile collector directory:

  ```bash
  sudo chown -R node_exporter:node_exporter /var/lib/node_exporter
  ```

### 4. Download and install the binary

- [ ] Download the release archive:

  ```bash
  cd /tmp
  curl -LO "$URL"
  ```

- [ ] Extract the archive:

  ```bash
  tar -xzf "$ARCHIVE"
  ```

- [ ] Install the binary:

  ```bash
  sudo install -o root -g root -m 0755 "/tmp/node_exporter-${VERSION}.${ARCH}/node_exporter" /usr/local/bin/node_exporter
  ```

- [ ] Confirm the binary is installed:

  ```bash
  /usr/local/bin/node_exporter --version
  ```

### 5. Create the environment file

- [ ] Create `/etc/node_exporter/node_exporter.env`:

  ```bash
  sudo tee /etc/node_exporter/node_exporter.env > /dev/null <<'EOF'
  NODE_EXPORTER_ARGS="--web.listen-address=:9100 --collector.systemd --collector.processes --collector.textfile.directory=/var/lib/node_exporter/textfile_collector"
  EOF
  ```

- [ ] Confirm the file exists:

  ```bash
  sudo cat /etc/node_exporter/node_exporter.env
  ```

### 6. Create the systemd service

- [ ] Create `/etc/systemd/system/node_exporter.service`:

  ```bash
  sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<'EOF'
  [Unit]
  Description=Prometheus Node Exporter
  Wants=network-online.target
  After=network-online.target

  [Service]
  User=node_exporter
  Group=node_exporter
  EnvironmentFile=/etc/node_exporter/node_exporter.env
  ExecStart=/usr/local/bin/node_exporter $NODE_EXPORTER_ARGS
  Restart=on-failure
  RestartSec=5s
  NoNewPrivileges=true
  ProtectSystem=strict
  ProtectHome=true
  PrivateTmp=true
  ProtectControlGroups=true
  ProtectKernelModules=true
  ProtectKernelTunables=true
  LockPersonality=true
  MemoryDenyWriteExecute=true
  RestrictRealtime=true
  RestrictSUIDSGID=true
  SystemCallArchitectures=native

  [Install]
  WantedBy=multi-user.target
  EOF
  ```

### 7. Start and enable the service

- [ ] Reload systemd:

  ```bash
  sudo systemctl daemon-reload
  ```

- [ ] Enable and start the service:

  ```bash
  sudo systemctl enable --now node_exporter
  ```

- [ ] Check service status:

  ```bash
  sudo systemctl status node_exporter
  ```

### 8. Validate exporter availability

- [ ] Confirm the process is listening on port `9100`:

  ```bash
  sudo ss -tulpn | grep 9100
  ```

- [ ] Query the local metrics endpoint:

  ```bash
  curl http://127.0.0.1:9100/metrics | head
  ```

- [ ] Optionally test from another trusted host on the LAN:

  ```bash
  curl http://<proxmox-host-ip>:9100/metrics | head
  ```

### 9. Record the final state

- [ ] Record the pinned version installed
- [ ] Record the service file path
- [ ] Record the listen port
- [ ] Record that Prometheus should scrape the Proxmox host on port `9100`
- [ ] Update observability documentation

## Validation

The install is successful when:

- the `node_exporter` system user exists
- the binary is installed at `/usr/local/bin/node_exporter`
- the systemd service is enabled and running
- the exporter listens on port `9100`
- `curl http://127.0.0.1:9100/metrics` returns metrics output

## Rollback / Fallback

- Stop and disable the service:

  ```bash
  sudo systemctl disable --now node_exporter
  ```

- Remove the systemd unit:

  ```bash
  sudo rm -f /etc/systemd/system/node_exporter.service
  sudo systemctl daemon-reload
  ```

- Remove the environment file and binary if the install must be fully removed:

  ```bash
  sudo rm -f /etc/node_exporter/node_exporter.env
  sudo rm -f /usr/local/bin/node_exporter
  ```

- Remove the service user only if no longer needed:

  ```bash
  sudo userdel node_exporter
  ```

- Do not add the host to Prometheus scrape configuration unless local validation succeeds

## Follow-Up

- Add the Proxmox host to Prometheus scrape targets
- Validate target health in Prometheus
- Add Grafana dashboards for host metrics
- Keep the installed version documented for future upgrades

## Notes

- `node_exporter` is one of the few justified additional agents on the Proxmox host because it improves platform observability directly
- Keep the Proxmox host otherwise minimal
- Prefer version pinning and deliberate upgrades rather than floating “latest” installs
