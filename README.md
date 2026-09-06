# Ansible Role: chronyd

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-chronyd) ![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-chronyd) ![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-chronyd) [![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-chronyd/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-chronyd/actions/workflows/dev.yml?query=branch%3Adev) [![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-chronyd/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-chronyd/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for managing time synchronization with chronyd.

## Purpose

Install chrony, validate and manage its configuration, and enable and start the time service. The default configuration synchronizes from NTP pools without serving NTP or exposing the UDP command port.

## Scope

### Managed

- Chrony packages, configuration, service enablement, and running state.
- NTP sources, server bindings, access rules, RTC settings, and rate limits.
- Removal of conflicting ntp packages and masking of platform-specific time services.

### Not Managed

- Firewall rules, authentication keys, and Samba socket permissions.
- Creation and permissions of custom drift-file and measurement-history directories.

## Requirements

- Gather Ansible facts before applying the role and run with root privileges.
- Upstream NTP sources must be reachable for actual clock synchronization.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

The following variables are part of the public role interface.

| Name | Type | Required | Default | Description |
| ---- | ---- | -------- | ------- | ----------- |
| `chronyd_directives` | `list` | `false` | [] | Ordered access directives; explicit deny entries below are applied afterwards. |
| `chronyd_port` | `int` | `false` | `0` | NTP server port; zero disables serving NTP requests. |
| `chronyd_bindaddress` | `str` | `false` | `` | Local address for NTP requests; an empty string uses the daemon default. |
| `chronyd_bindcmdaddress` | `str` | `false` | `127.0.0.1` | Local address for the optional chronyc UDP command socket. |
| `chronyd_cmdport` | `int` | `false` | `0` | Command port; zero restricts chronyc access to the local Unix socket. |
| `chronyd_deny` | `list` | `false` | [] | Subnets denied NTP access after processing the ordered directives. |
| `chronyd_driftfile` | `str` | `false` | `/var/lib/chrony/chrony.drift` | Drift file path; an empty string disables saving the estimated clock drift. |
| `chronyd_dumponexit` | `bool` | `false` | `True` | Save measurement histories when the daemon exits. |
| `chronyd_dumpdir` | `str` | `false` | `/var/lib/chrony` | Measurement history directory; an empty string omits the dumpdir directive. |
| `chronyd_hwtimestamp_interfaces` | `list` | `false` | [] | Network interfaces supporting hardware timestamping. |
| `chronyd_leapsectz` | `str` | `false` | `right/UTC` | Timezone containing leap-second information; an empty string omits the directive. |
| `chronyd_makestep_secs` | `float` | `false` | `1.0` | Clock offset threshold in seconds above which an initial update may step time. |
| `chronyd_makestep_nums` | `int` | `false` | `3` | Number of initial updates allowed to step time; negative values remove the limit. |
| `chronyd_ntp_pools` | `list` | `false` | - address: 0.de.pool.ntp.org<br />  options:<br />    - iburst<br />- address: 1.de.pool.ntp.org<br />  options:<br />    - iburst<br />- address: 2.de.pool.ntp.org<br />  options:<br />    - iburst<br />- address: 3.de.pool.ntp.org<br />  options:<br />    - iburst | NTP pools with a required address and list of chrony source options. |
| `chronyd_ntp_servers` | `list` | `false` | [] | Individual NTP servers with a required address and list of source options. |
| `chronyd_ntpsigndsocket` | `str` | `false` | `` | Samba NTP signing socket directory; an empty string disables integration. |
| `chronyd_ratelimit` | `str` | `false` | `` | NTP response rate-limit options; an empty string omits the directive. |
| `chronyd_rtconutc` | `bool` | `false` | `True` | Interpret the hardware real-time clock as UTC. |
| `chronyd_rtcsync` | `bool` | `false` | `True` | Enable periodic kernel synchronization of the hardware real-time clock. |

## Managed Files

- `/etc/chrony/chrony.conf on Debian-family systems and Alpine.`
- `/etc/chrony.conf on Red Hat, SUSE, and Arch Linux systems.`

## Check Mode

Supports check mode on provisioned hosts through native module behavior.

- A first installation in check mode cannot validate configuration before chronyd is installed.

## Service Behavior

Every run enables and starts the service. Configuration changes notify a restart, applied before the role returns. Repeated runs with unchanged inputs and service state are idempotent.

### Handlers

- Restart the chrony service after a validated configuration change.

## Security Notes

- NTP serving and UDP command access are disabled by default.
- Configuration is owned by root with mode 0644 and uses module-provided backups.
- Explicit chronyd_deny entries are rendered after chronyd_directives; directive order is preserved.

## Operational Notes

- Rename legacy chrony_* inventory variables to chronyd_*; the role is now jomrr.chronyd.
- Pool and server entries require address and options; use an empty options list when no options are needed.
- Empty optional strings omit their corresponding configuration directives.
- Molecule runs chronyd with -x through a test-only service override and does not adjust the host clock.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### NTP client

Install the default client configuration.

```yaml
- name: Configure time synchronization
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: jomrr.chronyd
```
### NTP server

Serve one network while excluding a subnet.

```yaml
- name: Configure an NTP server
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: jomrr.chronyd
      chronyd_port: 123
      chronyd_bindaddress: 192.0.2.1
      chronyd_directives:
        - allow 192.0.2.0/24
      chronyd_deny:
        - 192.0.2.128/25
```

## References

- [chrony Documentation](https://chrony-project.org/documentation.html)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2020 Jonas Mauer.
