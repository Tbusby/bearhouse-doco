# Ubuntu 24.04 Cloud-Init Template

## Purpose

This template provides the standard base image for Linux virtual machines deployed on the Proxmox homelab platform.

It is intended to support:

- repeatable VM provisioning
- cloud-init customization at clone time
- SSH key-based access
- guest agent integration with Proxmox
- future Terraform and Ansible workflows

This template is the default starting point for:

- Kubernetes control plane nodes
- Kubernetes worker nodes
- management VMs such as `ops1`
- supporting service VMs such as `svc1`

---

## Template Summary

| Item | Value |
|------|-------|
| Template Name | `ubuntu-2404-cloudinit-template` |
| Template VM ID | `100` |
| Operating System | Ubuntu Server 24.04 LTS |
| Image Type | Ubuntu cloud image |
| Primary Disk Storage | `vmdata` |
| Cloud-Init Storage | `vmdata` |
| Network Bridge | `vmbr0` |
| Default Cloud-Init User | `tbusby` |
| Network Configuration | DHCP by default |
| Guest Agent | Enabled |
| SSH Access | Public key injection via cloud-init |

---

## Design Goals

This template was created to provide a clean and automation-friendly VM base.

Key design goals:

- use a cloud-init compatible Ubuntu image rather than a manually installed ISO-based VM
- allow per-VM customization without modifying the base image
- support SSH key injection for secure access
- support guest introspection through `qemu-guest-agent`
- reduce manual setup effort when cloning new VMs
- establish a consistent base for future infrastructure-as-code workflows

---

## Template Configuration

### Compute

| Setting | Value |
|---------|-------|
| CPU Type | `host` |
| vCPU | 2 |
| Memory | 2048 MB |

### Storage

| Setting | Value |
|---------|-------|
| Main Disk Bus | `SCSI` |
| SCSI Controller | `VirtIO SCSI single` |
| Main Disk Size | 20 GB |
| Main Disk Storage | `vmdata` |
| Cloud-Init Disk | Attached |
| Cloud-Init Disk Storage | `vmdata` |

### Network

| Setting | Value |
|---------|-------|
| NIC Model | `VirtIO` |
| Bridge | `vmbr0` |
| Default IP Assignment | DHCP |

### Console and Agent

| Setting | Value |
|---------|-------|
| Serial Port | Enabled |
| Display | Serial terminal |
| QEMU Guest Agent | Enabled |

---

## Cloud-Init Configuration

The template uses Proxmox cloud-init integration to apply per-VM customization during clone deployment.

### Default Cloud-Init Settings

| Setting | Value |
|---------|-------|
| User | `tbusby` |
| Authentication | SSH public key |
| IP Config | DHCP |
| DNS | Inherited / optional per clone |
| Hostname | Defined per clone |

### SSH Key Injection Strategy

SSH access is provided using cloud-init public key injection.

The public key is configured in the Proxmox **Cloud-Init** settings for the template or per cloned VM.

Only public keys are injected. Private keys are never stored on the Proxmox host.

---

## Guest Configuration

The following package is installed inside the guest image after first boot and before conversion to template:

- `qemu-guest-agent`

### Guest Agent Purpose

`qemu-guest-agent` provides:

- improved VM state visibility in Proxmox
- IP address reporting
- cleaner shutdown and interaction support
- better integration between Proxmox and the guest OS

---

## Build Process Summary

The template build process followed these high-level steps:

1. Create a new VM shell in Proxmox
2. Import the Ubuntu 24.04 cloud image as the main disk
3. Attach the imported disk as `scsi0`
4. Add a cloud-init drive
5. Configure boot order, network, and cloud-init defaults
6. Inject SSH public key
7. Resize the primary disk to 20 GB
8. Boot the VM
9. Install and enable `qemu-guest-agent`
10. Shut down the VM cleanly
11. Convert the VM into a Proxmox template
12. Validate template behavior by cloning a test VM

---

## Validation Performed

The template was validated by creating a full clone test VM and confirming:

- successful clone creation
- successful boot
- network configuration via DHCP
- SSH access using injected public key
- presence of the configured cloud-init user
- successful guest agent operation
- suitability for reuse as a base image

Validation outcome:

- **Status:** Passed

---

## Approved Usage

This template is approved for creating the following VM types:

| VM Type | Supported |
|---------|-----------|
| Management VM | Yes |
| Kubernetes Control Plane Node | Yes |
| Kubernetes Worker Node | Yes |
| General Ubuntu Utility VM | Yes |

---

## Operational Guidance

- New Linux VMs should be cloned from this template unless a documented exception exists
- Hostnames, IP settings, and SSH keys may be customized per clone via cloud-init
- Base package hardening and additional software should be applied using Ansible after deployment
- The template should remain minimal and not become a snowflake image
- Significant template changes should trigger revalidation with a fresh test clone

---

## Future Improvements

Potential future enhancements include:

- pre-installing additional baseline packages
- adding a documented image versioning scheme
- adding CIS-style baseline hardening through Ansible rather than baking it into the template
- introducing Terraform-driven cloning workflows
- using image lifecycle/version control with template replacement procedures

---

## Notes

- The template is intentionally minimal
- Configuration beyond baseline access and guest integration is expected to be applied after deployment
- This approach keeps the template reusable, predictable, and compatible with infrastructure-as-code workflows