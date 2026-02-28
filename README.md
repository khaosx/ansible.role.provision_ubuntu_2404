# Role: provision_ubuntu_2404

## Description

Baseline provisioning and hardening for Ubuntu hosts. The role applies base package lifecycle, time synchronization, SSH/sysctl/fail2ban hardening, optional Raspberry Pi radio controls, optional GPU driver workflow, and UFW baseline policy.

## Requirements

- Ansible >= 2.15
- Collections:
  - `community.general`
- Target platforms:
  - Ubuntu

## Privilege Escalation

requires_become: true

## Supported Platforms

- Ubuntu

## User-Facing Capabilities

- Hostname and `/etc/hosts` baseline management
- Base APT maintenance and package install set
- Optional QEMU guest agent installation
- Chrony-based NTP configuration
- Unattended upgrade and periodic APT policy rendering
- SSH hardening drop-in deployment
- Sysctl hardening drop-in deployment
- Fail2ban jail deployment
- UFW baseline defaults and optional profile catalog integration via `ufw_profiles`
- Optional Raspberry Pi radio disable workflow via firmware overlays
- Optional NVIDIA and Intel GPU tooling workflow
- Reboot marker detection and reboot signaling handler

## Role Variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `provision_ubuntu_2404_reboot_timeout` | `600` | No | Reboot timeout (seconds) for reboot tasks. |
| `provision_ubuntu_2404_site_timezone` | `"UTC"` | No | Timezone applied by `community.general.timezone`. |
| `provision_ubuntu_2404_apps_to_install` | See `defaults/main.yml` | No | Base package list installed by APT. |
| `provision_ubuntu_2404_unattended_mail` | `""` | No | Optional mail recipient for unattended-upgrades. |
| `provision_ubuntu_2404_unattended_mail_only_on_error` | `true` | No | Send unattended-upgrades mail only on errors. |
| `provision_ubuntu_2404_unattended_minimal_steps` | `true` | No | Toggle unattended-upgrades minimal-steps mode. |
| `provision_ubuntu_2404_unattended_reboot` | `true` | No | Allow unattended-upgrades automated reboot. |
| `provision_ubuntu_2404_unattended_reboot_time` | `"04:00"` | No | Time for unattended-upgrades auto reboot. |
| `provision_ubuntu_2404_apt_periodic_update` | `"1"` | No | APT periodic package list update cadence. |
| `provision_ubuntu_2404_apt_periodic_download` | `"1"` | No | APT periodic package download cadence. |
| `provision_ubuntu_2404_apt_periodic_upgrade` | `"1"` | No | APT periodic unattended-upgrade cadence. |
| `provision_ubuntu_2404_apt_periodic_autoclean` | `"7"` | No | APT periodic autoclean cadence. |
| `provision_ubuntu_2404_apt_periodic_random_sleep` | `"1800"` | No | APT periodic random sleep in seconds. |
| `provision_ubuntu_2404_ssh_allow_users` | `[]` | No | Optional SSH `AllowUsers` list. |
| `provision_ubuntu_2404_sysctl_params` | `{}` | No | Sysctl key/value map rendered to `/etc/sysctl.d/99-hardening.conf`. |
| `provision_ubuntu_2404_fail2ban_jail` | See `defaults/main.yml` | No | Fail2ban jail map rendered to `/etc/fail2ban/jail.local`. |
| `provision_ubuntu_2404_ufw_enable` | `true` | No | Enable and manage UFW baseline policies. |
| `provision_ubuntu_2404_ufw_reset` | `false` | No | Reset UFW before baseline application. |
| `provision_ubuntu_2404_ufw_ipv6` | `false` | No | Set `IPV6=` in `/etc/default/ufw`. |
| `provision_ubuntu_2404_ufw_logging` | `"on"` | No | UFW logging level. |
| `provision_ubuntu_2404_ufw_default_incoming` | `deny` | No | UFW default incoming policy. |
| `provision_ubuntu_2404_ufw_default_outgoing` | `allow` | No | UFW default outgoing policy. |
| `provision_ubuntu_2404_ufw_manage_forward_policy` | `false` | No | Manage `DEFAULT_FORWARD_POLICY` in `/etc/default/ufw`. |
| `provision_ubuntu_2404_ufw_default_forward_policy` | `"DROP"` | No | Value to set for `DEFAULT_FORWARD_POLICY` when managed. |
| `provision_ubuntu_2404_chrony_ntp_servers` | See `defaults/main.yml` | No | Chrony upstream server list. |
| `provision_ubuntu_2404_chrony_allow_network` | `[]` | No | Optional networks allowed by chrony `allow` directives. |
| `provision_ubuntu_2404_chrony_makestep_threshold` | `1.0` | No | Chrony `makestep` threshold value. |
| `provision_ubuntu_2404_chrony_makestep_limit` | `3` | No | Chrony `makestep` limit value. |
| `provision_ubuntu_2404_rpi_disable_radios` | `true` | No | Master toggle for Raspberry Pi Wi-Fi/Bluetooth disable defaults. |
| `provision_ubuntu_2404_rpi_disable_wifi` | `{{ provision_ubuntu_2404_rpi_disable_radios \| bool }}` | No | Disable Raspberry Pi Wi-Fi firmware overlay. |
| `provision_ubuntu_2404_rpi_disable_bluetooth` | `{{ provision_ubuntu_2404_rpi_disable_radios \| bool }}` | No | Disable Raspberry Pi Bluetooth firmware overlay. |
| `provision_ubuntu_2404_rpi_disable_bt_service` | `false` | No | Disable and mask `bluetooth.service`. |
| `provision_ubuntu_2404_rpi_config_txt_path` | `"/boot/firmware/config.txt"` | No | Raspberry Pi firmware config path. |
| `provision_ubuntu_2404_rpi_debug` | `false` | No | Enable Raspberry Pi detection debug output. |

## Inbound Variables

- Role-owned input variables are documented in the table above (`provision_ubuntu_2404_*`).
- Optional upstream variable for shared firewall profiles:
  - `ufw_application_catalog` (consumed only when present to import `ufw_profiles` catalog entries)

## Outbound Artifacts

This role modifies:
- `/etc/hosts`
- `/etc/sudoers`
- `/etc/update-motd.d/*` executable bits for selected scripts
- `/etc/chrony/chrony.conf`
- `/etc/apt/apt.conf.d/20auto-upgrades`
- `/etc/apt/apt.conf.d/50unattended-upgrades`
- `/etc/ssh/sshd_config.d/99-hardening.conf`
- `/etc/sysctl.d/99-hardening.conf`
- `/etc/fail2ban/jail.local`
- `/etc/default/ufw`
- Raspberry Pi firmware config file (default `/boot/firmware/config.txt`) when Raspberry Pi radio controls are requested

Role facts set:
- `provision_ubuntu_2404_reboot_needed`
- GPU and Raspberry Pi detection facts prefixed with `provision_ubuntu_2404_`

## Idempotency

Idempotent: True (with expected command-task exceptions explicitly marked via `changed_when`).

## Atomicity

Atomic: False (role performs multiple independent system configuration steps; no global transaction semantics).

## Rollback

No automatic rollback orchestration is implemented. Template and lineinfile tasks use `backup: true` where supported; manual rollback should restore generated backup files and re-run the role with corrected inputs.

## Dependencies

- Runtime dependency when UFW profile catalog integration is used:
  - `ufw_profiles` role (imported conditionally in `tasks/firewall.yml`)

## Example Playbook

```yaml
---
- name: Apply Ubuntu baseline
  hosts: ubuntu
  gather_facts: true
  become: true
  roles:
    - role: khaosx.homelab.provision_ubuntu_2404
```

## Check Mode Behavior

- Hardware probing command tasks run in check mode and report no change.
- Driver installation and package-management behavior depends on module support in check mode.
- Reboot handlers are notified only when marker-driven change conditions are met.

## License

MIT

## Author

Kris
