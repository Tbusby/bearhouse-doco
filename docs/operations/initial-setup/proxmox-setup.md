# Proxmox Install and Post-Install Checklist

## Purpose

This checklist defines the required planning, installation, validation, and post-install tasks for deploying the Proxmox VE host that underpins the homelab platform.

The goal is to ensure the Proxmox host is installed deliberately, securely, and with enough forethought to support later automation, Kubernetes workloads, backups, and operational maintenance.

## Scope

Applies to the dedicated Proxmox VE installation on the homelab physical host:

- **Host Model:** Lenovo ThinkStation P520c
- **CPU:** Intel Xeon W-2245
- **RAM:** 64 GB
- **Storage:** 1 TB NVMe SSD
- **Role:** Dedicated virtualization platform for the homelab

## Design Intent

The Proxmox host is foundational infrastructure and should be treated as such.

It should:
- remain minimal
- avoid running general-purpose applications directly
- host virtual machines for the platform
- provide predictable networking and storage
- be documented and maintained carefully

It should not become a catch-all machine for unmanaged services.

---

# 1. Pre-Install Planning Checklist

## 1.1 Hardware and Firmware

- [ ] Confirm physical host is healthy and ready for dedicated use
- [ ] Confirm BIOS/UEFI firmware is up to date if practical
- [ ] Enable Intel virtualization support (VT-x)
- [ ] Enable VT-d / IOMMU if available
- [ ] Confirm system boot mode choice (prefer UEFI unless there is a specific reason not to)
- [ ] Set correct system date and time in BIOS
- [ ] Configure **power on after AC loss**
- [ ] Confirm NVMe drive is detected correctly
- [ ] Confirm installation media is bootable

## 1.2 Host Identity and Networking

- [ ] Select Proxmox hostname
- [ ] Recommended hostname: `proxmox01`
- [ ] Select static management IP address
- [ ] Select gateway
- [ ] Select DNS server(s)
- [ ] Confirm local subnet and CIDR
- [ ] Confirm Proxmox will initially live on the main LAN
- [ ] Confirm physical NIC to be used for bridge networking
- [ ] Confirm initial bridge design:
  - [ ] `vmbr0` bridged to physical NIC
  - [ ] host management and VMs on same LAN initially
- [ ] Record intended MetalLB address range for later Kubernetes use
- [ ] Record intended Kubernetes API virtual IP for later use

## 1.3 Storage Planning

- [ ] Decide storage model before install
- [ ] Choose one:
  - [ ] ext4/LVM
  - [ ] ZFS (single disk, no redundancy)
- [ ] Document why this storage model was chosen
- [ ] Plan to leave sufficient free space for:
  - [ ] VM growth
  - [ ] snapshots
  - [ ] templates
  - [ ] temporary backups
- [ ] Confirm no expectation of disk redundancy from a single NVMe device

## 1.4 Resource Planning

- [ ] Reserve host headroom for Proxmox
- [ ] Plan to leave at least 8–12 GB RAM uncommitted
- [ ] Plan to retain CPU headroom for host and burst workloads
- [ ] Plan to keep meaningful free disk space available after VM deployment

## 1.5 Naming and Standards

- [ ] Confirm host naming standard
- [ ] Confirm VM naming standard
- [ ] Confirm VM ID allocation strategy
- [ ] Example ID ranges documented:
  - [ ] 100–199 templates/base images
  - [ ] 200–299 Kubernetes control planes
  - [ ] 300–399 Kubernetes workers
  - [ ] 400–499 ops and utility VMs

## 1.6 Access and Security

- [ ] Confirm Proxmox management UI will not be exposed publicly
- [ ] Confirm only local LAN or VPN access will be allowed
- [ ] Prepare strong initial administrative password
- [ ] Prepare SSH public key for post-install access
- [ ] Confirm intention to use key-based SSH access after install

## 1.7 Documentation Readiness

- [ ] Create or update documentation page for Proxmox host details
- [ ] Record intended IP address, hostname, storage choice, and bridge layout
- [ ] Prepare post-install validation checklist
- [ ] Prepare template creation plan for Ubuntu Server cloud-init image

---

# 2. Installation Checklist

## 2.1 Installation Media and Boot

- [ ] Boot from Proxmox VE installer
- [ ] Confirm correct target disk before proceeding
- [ ] Confirm no unintended disks or removable media will be overwritten

## 2.2 Disk and Filesystem

- [ ] Select intended target NVMe device
- [ ] Apply chosen filesystem/storage model
- [ ] Verify storage choice matches planning decision

## 2.3 Region and Credentials

- [ ] Set correct country/region/timezone
- [ ] Set secure administrator password
- [ ] Set administrative email address if desired

## 2.4 Network Configuration

- [ ] Set hostname
- [ ] Set static management IP address
- [ ] Set gateway
- [ ] Set DNS server(s)
- [ ] Confirm selected management NIC is correct

## 2.5 Final Review

- [ ] Review summary carefully before install starts
- [ ] Confirm disk target
- [ ] Confirm network settings
- [ ] Confirm hostname
- [ ] Confirm storage settings
- [ ] Proceed with installation

---

# 3. Immediate Post-Install Validation Checklist

## 3.1 Host Access

- [ ] Confirm host boots successfully
- [ ] Confirm web UI is reachable
- [ ] Confirm SSH access works
- [ ] Verify hostname is correct
- [ ] Verify management IP is correct

## 3.2 Basic Host Health

- [ ] Confirm correct system time
- [ ] Confirm DNS resolution works
- [ ] Confirm gateway connectivity
- [ ] Confirm internet access works if expected
- [ ] Confirm storage is visible in Proxmox UI
- [ ] Confirm bridge networking exists and is functional

## 3.3 Initial Documentation

- [ ] Record final installed hostname
- [ ] Record final IP configuration
- [ ] Record final storage configuration
- [ ] Record Proxmox version installed
- [ ] Record NIC/bridge mapping
- [ ] Record any deviations from planned design

---

# 4. Post-Install Configuration Checklist

## 4.1 Repository and Updates

- [ ] Configure Proxmox repositories appropriately for the environment
- [ ] Disable or replace enterprise repository if not using subscription
- [ ] Update package lists
- [ ] Apply all current updates
- [ ] Reboot if required after updates
- [ ] Confirm host returns cleanly after reboot

## 4.2 SSH and Access Hardening

- [ ] Add administrator SSH public key
- [ ] Confirm SSH key login works
- [ ] Reduce reliance on password-based login where practical
- [ ] Confirm Proxmox UI access is limited to intended networks
- [ ] Document access requirements and admin methods

## 4.3 Time and System Services

- [ ] Confirm NTP/time synchronization is functioning
- [ ] Confirm core services are healthy
- [ ] Review boot-time warnings or service failures
- [ ] Confirm no failed systemd services requiring attention

## 4.4 Networking

- [ ] Validate `vmbr0` bridge configuration
- [ ] Confirm VMs will be able to use the bridge as intended
- [ ] Confirm no unexpected DNS or routing issues exist
- [ ] Record network settings in docs

## 4.5 Storage

- [ ] Confirm available free space
- [ ] Confirm datastore naming is sensible
- [ ] Confirm ISO/template storage location
- [ ] Confirm snapshot expectations are understood
- [ ] Document storage caveats of the single-disk design

---

# 5. Host Standards Checklist

## 5.1 Proxmox Host Role Boundaries

- [ ] Confirm host will remain dedicated to virtualization duties
- [ ] Do not install Docker workloads directly on Proxmox
- [ ] Do not run random application services directly on the host
- [ ] Do not use the host as the long-term home for operational tooling
- [ ] Reserve application and automation tooling for guest VMs such as `ops1`

## 5.2 Capacity Discipline

- [ ] Do not fully allocate all host RAM to VMs
- [ ] Do not consume nearly all storage with initial VM builds
- [ ] Reserve space for templates, snapshots, logs, and future growth
- [ ] Monitor storage usage from the start

---

# 6. VM Template Preparation Checklist

## 6.1 Base Image Strategy

- [ ] Decide on guest OS template standard
- [ ] Recommended guest OS: Ubuntu Server 24.04 LTS
- [ ] Download cloud image or installation ISO as needed
- [ ] Document template naming convention

## 6.2 Template Build

- [ ] Create initial Ubuntu VM template
- [ ] Enable cloud-init support
- [ ] Install `qemu-guest-agent`
- [ ] Configure default user strategy
- [ ] Configure SSH key injection
- [ ] Confirm template can clone successfully
- [ ] Convert tested VM into reusable template

## 6.3 Template Validation

- [ ] Clone a test VM from the template
- [ ] Confirm network comes up correctly
- [ ] Confirm hostname assignment works
- [ ] Confirm cloud-init settings apply
- [ ] Confirm SSH access works
- [ ] Confirm `qemu-guest-agent` reports correctly in Proxmox

---

# 7. Automation Readiness Checklist

## 7.1 Terraform Readiness

- [ ] Confirm Proxmox API access plan for Terraform
- [ ] Confirm token/user strategy for automation
- [ ] Record Proxmox endpoint details for Terraform provider config
- [ ] Document template IDs/names for Terraform use
- [ ] Confirm naming and VM ID strategy is compatible with IaC

## 7.2 Ansible Readiness

- [ ] Confirm Ansible control node strategy (`ops1` planned)
- [ ] Confirm inventory structure approach
- [ ] Confirm SSH access from management environment will work
- [ ] Document baseline bootstrap expectations for all VMs

---

# 8. Backup Planning Checklist

## 8.1 Proxmox-Level Backup Planning

- [ ] Decide which VMs should receive Proxmox backups
- [ ] Identify temporary local backup location if needed
- [ ] Record that Kubernetes worker/control-plane nodes are primarily rebuildable
- [ ] Identify critical VMs likely to warrant backup attention:
  - [ ] `ops1`
  - [ ] `svc1`

## 8.2 Offsite Backup Direction

- [ ] Confirm future AWS S3 backup plan
- [ ] Record that offsite backup will focus primarily on:
  - [ ] data
  - [ ] repositories
  - [ ] restic-managed backup sets
  - [ ] selected exports
- [ ] Do not assume Proxmox alone is the disaster recovery strategy

---

# 9. Final Validation Checklist Before Creating Real VMs

- [ ] Proxmox fully updated
- [ ] Web UI and SSH working
- [ ] Bridge networking validated
- [ ] Storage validated
- [ ] Documentation updated
- [ ] SSH keys configured
- [ ] Ubuntu template strategy prepared
- [ ] Naming and VM ID standards documented
- [ ] Automation approach understood
- [ ] Host remains clean and minimal

---

# 10. Recommended Initial Build Order After Proxmox Install

1. Update Proxmox and validate host health
2. Configure repositories and SSH access
3. Validate bridge networking and storage
4. Build and test Ubuntu cloud-init template
5. Document final host configuration
6. Prepare Terraform for VM creation
7. Create `ops1`
8. Create Kubernetes control plane and worker VMs
9. Begin Ansible-based bootstrap

---

# 11. Notes and Decisions

## Selected Hostname
- `proxmox01`

## Selected Management IP
- `<fill in>`

## Selected Gateway
- `<fill in>`

## Selected DNS Servers
- `<fill in>`

## Selected Storage Model
- `<fill in>`

## Selected Physical NIC
- `<fill in>`

## Selected Bridge
- `vmbr0`

## Notes
- `<fill in>`

---

# 12. Acceptance Criteria

The Proxmox installation is considered ready when:

- the host is reachable and stable
- storage and networking are correctly configured
- updates are applied
- access is secured appropriately
- documentation reflects the real installed state
- a reusable Ubuntu template can be built
- the host is ready for Terraform-driven VM provisioning