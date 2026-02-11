# ansible.role.provision_ubuntu_2404

Baseline provisioning and hardening role for Ubuntu 24.04.

## Purpose

This role:
- Applies baseline package install/upgrade
- Configures timezone, unattended upgrades, SSH hardening, sysctl hardening, fail2ban
- Configures UFW defaults and optional profile installation
- Applies chrony configuration
- Handles Raspberry Pi radio overlay settings when applicable
- Performs optional GPU driver workflow (NVIDIA detection/install)

## Requirements

- Ubuntu host
- `become: true` at play/role inclusion level
- `community.general` collection (for UFW module)
- Role `ufw_profiles` available to the controlling playbook when using UFW profile catalog integration

## Role Execution Model

This role is intended to run subordinately from a controlling playbook.

The controlling playbook should own:
- Inventory structure and vault management
- Site/global policy decisions
- Role ordering and privilege model
- Shared variables consumed by this role

## Control Playbook Contract

Role defaults are in `defaults/main.yml`.
Controller inventory/vault should override environment-specific values.

### Expected upstream variables

| Variable | Required | Description |
|---|---|---|
| `ufw_application_catalog` | no | Shared UFW application profile catalog consumed by `ufw_profiles` role import |

### Variable naming convention

Role-owned variables use the `provision_ubuntu_2404_` prefix.

## Key Variables

- `provision_ubuntu_2404_apps_to_install`
- `provision_ubuntu_2404_site_timezone`
- `provision_ubuntu_2404_ufw_enable`
- `provision_ubuntu_2404_ufw_default_incoming`
- `provision_ubuntu_2404_ufw_default_outgoing`
- `provision_ubuntu_2404_fail2ban_jail`
- `provision_ubuntu_2404_sysctl_params`
- `provision_ubuntu_2404_chrony_ntp_servers`
- `provision_ubuntu_2404_rpi_disable_radios`
- `provision_ubuntu_2404_reboot_timeout`

## Example Controller Play

```yaml
---
- name: Baseline Ubuntu provisioning
  hosts: ubuntu
  become: true
  roles:
    - role: provision_ubuntu_2404
```

## Notes

- This role intentionally configures passwordless sudo for the `sudo` group.
- UFW policy values are validated before apply.

## License

MIT
