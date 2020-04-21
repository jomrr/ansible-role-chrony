# ansible-role-chrony

![GitHub](https://img.shields.io/github/license/jam82/ansible-role-chrony) [![Build Status](https://travis-ci.org/jam82/ansible-role-chrony.svg?branch=master)](https://travis-ci.org/jam82/ansible-role-chrony)

**Ansible role for setting up chrony.**

Currently only client mode is supported, as I do not need an own ntp server.

But server functionality will follow... :)

> WARNING: This role uninstalls `ntp`.

## Supported Platforms

- Alpine 3.11
- Archlinux
- CentOS 8
- Debian 9, 10
- Fedora 31
- Ubuntu 18.04, 20.04

## Requirements

Ansible 2.9 or higher is recommended.

## Variables

Variables and defaults for this role.

### Cient defaults

```yaml
---
# role: ansible-role-chrony
# file: defaults/client.yml

# The role is disabled by default, so you do not get in trouble.
# Checked in tasks/main.yml which includes tasks.yml if enabled.
chrony_role_enabled: False

# Port 0 completely disables the ntp server functionality
chrony_port: 0

# Disable chronyc/command port.
# Only root and chrony user can access it via socket.
chrony_cmdport: 0

# Bind chronyc only to local interface by default.
# NOTE: This only has any effect if cmdport > 0.
chrony_bindcmdaddress: 127.0.0.1

# Location of the drift file
chrony_driftfile: /var/lib/chrony/chrony.drift

# This will reduce the time to catch up if chronyd is restarted,
# as it saves its state to a directory.
chrony_dumponexit: True
chrony_dumpdir: /var/lib/chrony

# List of network interfaces for hardware timestamping
# Capabilities SOF_TIMESTAMPING_TX_HARDWARE,
# SOF_TIMESTAMPING_TX_SOFTWARE, HWTSTAMP_FILTER_ALL are needed.
# Check with:
# test $(ethtool -T {{ item }} | grep 'SOF_TIMESTAMPING_TX_HARDWARE\|SOF_TIMESTAMPING_TX_SOFTWARE\|HWTSTAMP_FILTER_ALL' | wc -l) == 3 && echo OK || echo NOK
chrony_hwtimestamp_interfaces: []

# If the system timezone database is kept up to date and includes the
# right/UTC timezone, chronyd can use it to determine the current
# TAI-UTC offset and when will the next leap second occur.
chrony_leapsectz: right/UTC

# Allow stepping. Useful for VMs that can be suspended.
# This allows stepping during the first 3 updates if the clock
# is more than 1 second away from the ntp server.
# NOTE: This results in "makestep 1 3" in chrony.conf file.
chrony_makestep_secs: 1
chrony_makestep_nums: 3

# see https://chrony.tuxfamily.org/faq.html for options and faq.

# List of NTP Pools with option list.
chrony_ntp_pools:
  - address: 0.de.pool.ntp.org
    options:
      - iburst
  - address: 1.de.pool.ntp.org
    options:
      - iburst
  - address: 2.de.pool.ntp.org
    options:
      - iburst
  - address: 3.de.pool.ntp.org
    options:
      - iburst

# List of NTP Servers with option list.
chrony_ntp_servers: []
# following options are only for LAN, a public ntp could block your
# for sending too many requests.
#  - address: 192.168.1.1
#    options:
#      - "minpoll 2"
#      - "maxpoll 4"
#      - "polltarget 30"
#      - iburst
#      - xleave

# RealTime Clock in UTC
chrony_rtconutc: True
# Sync your RTC wit NTP
chrony_rtcsync: True
```

### Server defaults

Not implemented yet.

## Dependencies

None.

## Example Playbook

This will configure a client to use the first four *.de.pool.ntp.org servers.

```yaml
---
# role: ansible-role-chrony
# file: site.yml

- hosts: chrony_systems
  become: True
  vars:
    chrony_role_enabled: True
  roles:
    - role: ansible-role-chrony
```

## License and Author

- Author:: [jam82](https://github.com/jam82/)
- Copyright:: 2020, [jam82](https://github.com/jam82/)

Licensed under [MIT License](https://opensource.org/licenses/MIT).
See [LICENSE](https://github.com/jam82/ansible-role-chrony/blob/master/LICENSE) file in repository.

## References

- [Red Hat - Understanding chrony and Its Configuration](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-understanding_chrony_and-its_configuration)
- [Red Hat - Using the Chrony suite to configure NTP](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/using-chrony-to-configure-ntp)
- [TuxFamily - chrony](https://chrony.tuxfamily.org/faq.html#_must_i_specify_servers_by_ip_address_if_dns_is_not_available_on_chronyd_start)
