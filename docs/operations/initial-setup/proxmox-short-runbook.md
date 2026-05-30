# Runbook: Proxmox Install Day

## Purpose

Install and validate the Proxmox VE host for the homelab platform.

This runbook is intended for actual install day use and is shorter than the full planning checklist.

## Scope

Applies to the initial Proxmox VE installation on the dedicated homelab host.

## Prerequisites

- Proxmox installer USB prepared
- Keyboard, monitor, and network connected
- Planned hostname, IP, gateway, and DNS available
- Storage choice already decided
- SSH public key available
- Documentation repo ready for post-install notes

## Planned Values

| Item | Value |
| ---- | ----- |
| Hostname | `proxmox01` |
| Management IP | `<fill in>` |
| Gateway | `<fill in>` |
| DNS | `<fill in>` |
| Storage model | `<fill in>` |
| Bridge | `vmbr0` |

## Risks

- Installing to the wrong disk
- Incorrect network config preventing access
- Forgetting post-install update and repo configuration
- Drifting from documented design during setup

## Procedure

### 1. Pre-boot checks

- [ ] Confirm BIOS virtualization settings are enabled
- [ ] Confirm VT-d / IOMMU enabled if desired
- [ ] Confirm power-on after AC loss is enabled
- [ ] Confirm target NVMe disk is correct
- [ ] Confirm installer USB is inserted and bootable
- [ ] Confirm planned network settings are available

### 2. Install Proxmox

- [ ] Boot installer
- [ ] Select correct target disk
- [ ] Select intended storage/filesystem option
- [ ] Set timezone and locale
- [ ] Set secure admin password
- [ ] Set hostname to `proxmox01`
- [ ] Configure static IP, gateway, and DNS
- [ ] Review summary carefully
- [ ] Start installation

### 3. First boot validation

- [ ] Log into the console
- [ ] Confirm hostname is correct
- [ ] Confirm management IP is correct
- [ ] Confirm web UI is reachable from another machine
- [ ] Confirm SSH access works if enabled/available
- [ ] Confirm DNS resolution works
- [ ] Confirm default gateway/network connectivity works

### 4. Immediate post-install tasks

- [ ] Configure appropriate Proxmox repositories
- [ ] Update package lists
- [ ] Apply system updates
- [ ] Reboot if required
- [ ] Confirm system returns cleanly after reboot

### 5. Access and baseline config

- [ ] Add SSH public key
- [ ] Confirm key-based SSH login works
- [ ] Verify `vmbr0` bridge exists and looks correct
- [ ] Verify storage is present and usable in the UI
- [ ] Record final installed settings in docs

### 6. Ready for next phase

- [ ] Confirm host remains clean and minimal
- [ ] Do not install random apps on the host
- [ ] Plan next step: Ubuntu cloud-init template
- [ ] Plan following step: Terraform VM provisioning

## Validation

The install is successful when:

- Proxmox UI is reachable
- SSH access works
- networking is correct
- updates are applied
- storage is visible and healthy
- host configuration is documented

## Rollback / Fallback

- Reinstall if the wrong storage or network settings were chosen
- Correct bridge/network settings from the console if remote access fails
- Do not proceed to VM creation until host networking and storage are confirmed working

## Follow-Up

- Create/update the full Proxmox host documentation page
- Build Ubuntu template
- Prepare Terraform provider configuration
- Begin VM provisioning workflow

## Notes

- Avoid overcomplicating install day
- Keep the host dedicated to virtualization duties
- Save deeper automation for the next phase
