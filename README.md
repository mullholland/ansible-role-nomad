# [Ansible role nomad](#nomad)
# [Ansible role nomad](#nomad)

description

|GitHub|Downloads|Version|
|------|---------|-------|
|[![github](https://github.com/mullholland/ansible-role-nomad/actions/workflows/molecule.yml/badge.svg)](https://github.com/mullholland/ansible-role-nomad/actions/workflows/molecule.yml)|[![downloads](https://img.shields.io/ansible/role/d/mullholland/nomad)](https://galaxy.ansible.com/mullholland/nomad)|[![Version](https://img.shields.io/github/release/mullholland/ansible-role-nomad.svg)](https://github.com/mullholland/ansible-role-nomad/releases/)|
## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/mullholland/ansible-role-nomad/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  # vars:
  #   example_var: "value"
  roles:
    - role: "mullholland.nomad"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/mullholland/ansible-role-nomad/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Debian/Ubuntu | Install support packages for testing
      package:
        name:
          - "iproute2"  # for ansible_default_ipv4.address
          - "cron"  # for backup cron
          - "findutils"  # for backup cron
        state: latest
      when: ansible_os_family == "Debian"

    - name: RedHat/CentOS | Install support packages for testing
      package:
        name:
          - "iproute"  # for ansible_default_ipv4.address
          - "cronie"  # for backup cron
          - "findutils"  # for backup cron
        state: latest
      when: ansible_os_family == "RedHat" or ansible_os_family == "Rocky"

  roles:
    - role: mullholland.repository_hashicorp
```



|GitHub|Downloads|Version|
|------|---------|-------|
|[![github](https://github.com/mullholland/ansible-role-nomad/actions/workflows/molecule.yml/badge.svg)](https://github.com/mullholland/ansible-role-nomad/actions/workflows/molecule.yml)|[![downloads](https://img.shields.io/ansible/role/d/mullholland/nomad)](https://galaxy.ansible.com/mullholland/nomad)|[![Version](https://img.shields.io/github/release/mullholland/ansible-role-nomad.svg)](https://github.com/mullholland/ansible-role-nomad/releases/)|
## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/mullholland/ansible-role-nomad/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  # vars:
  #   example_var: "value"
  roles:
    - role: "mullholland.nomad"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/mullholland/ansible-role-nomad/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Debian/Ubuntu | Install support packages for testing
      package:
        name:
          - "iproute2"  # for ansible_default_ipv4.address
          - "cron"  # for backup cron
          - "findutils"  # for backup cron
        state: latest
      when: ansible_os_family == "Debian"

    - name: RedHat/CentOS | Install support packages for testing
      package:
        name:
          - "iproute"  # for ansible_default_ipv4.address
          - "cronie"  # for backup cron
          - "findutils"  # for backup cron
        state: latest
      when: ansible_os_family == "RedHat" or ansible_os_family == "Rocky"

  roles:
    - role: mullholland.repository_hashicorp
```



## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/mullholland/ansible-role-nomad/blob/master/defaults/main.yml):

The default values for the variables are set in [`defaults/main.yml`](https://github.com/mullholland/ansible-role-nomad/blob/master/defaults/main.yml):

```yaml
---
# Is this server a nomad server?
# agents need this as `false`
nomad_server: true

# Packages
nomad_packages:
  - nomad
  # - nomad-enterprise

# set nomad license variable
# nomad_license: "12345679987654321"

# bin to this address
nomad_bind_address: "0.0.0.0"

# System user and group (from package)
nomad_user: "nomad"
nomad_group: "nomad"

# # Paths
nomad_bin_path: "/usr/bin"
nomad_config_path: "/etc/nomad.d"
nomad_data_path: "/opt/nomad/data"
nomad_log_path: "/var/log/nomad"
nomad_run_path: "/var/run/nomad"

# Create nomad configuration
# Sections are only for your personal structure
# they have no effect on the configuration
nomad_config_sections:
  - name: base
    config: |
      bind_addr = "{{ nomad_bind_address }}"
      data_dir  = "{{ nomad_data_path }}"
      name      = "{{ inventory_hostname }}"
  - name: logging
    config: |
      log_level            = "INFO"
      log_json             = false
      log_file             = "{{ nomad_log_path }}/nomad.log"
      log_rotate_duration  = "24h" # rotate after 24h
      log_rotate_max_files = "7"   # keep 1 week worth of logs
  - name: server
    config: |
      server {
        enabled          = true
        bootstrap_expect = 1  # Self-elect, should be 3 or 5 for production
      }
  - name: agent
    config: |
      client {
        enabled = true
        servers = [ "{{ inventory_hostname }}:4647" ]
      }
      # Disable the dangling container cleanup to avoid interaction with other clients
      plugin "docker" {
        config {
          gc {
            dangling_containers {
              enabled = false
            }
          }
        }
      }
  - name: telementry
    config: |
      telemetry {
        publish_allocation_metrics = true
        publish_node_metrics       = true
        prometheus_metrics         = true
      }
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/mullholland/ansible-role-nomad/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[mullholland.repository_hashicorp](https://galaxy.ansible.com/mullholland/repository_hashicorp)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-repository_hashicorp/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-repository_hashicorp/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-repository_hashicorp/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-repository_hashicorp)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://mullholland.net) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/mullholland/ansible-role-nomad/png/requirements.png "Dependencies")
## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/mullholland/ansible-role-nomad/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[mullholland.repository_hashicorp](https://galaxy.ansible.com/mullholland/repository_hashicorp)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-repository_hashicorp/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-repository_hashicorp/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-repository_hashicorp/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-repository_hashicorp)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://mullholland.net) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/mullholland/ansible-role-nomad/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/r/mullholland/enterpriselinux)|all|
|[Amazon](https://hub.docker.com/r/mullholland/amazonlinux)|Candidate|
|[Fedora](https://hub.docker.com/r/mullholland/fedora/)|38, 39|
|[Ubuntu](https://hub.docker.com/r/mullholland/ubuntu)|all|
|[Debian](https://hub.docker.com/r/mullholland/debian)|all|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.
- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-nomad/issues).
If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-nomad/issues).

## [License](#license)

[MIT](https://github.com/mullholland/ansible-role-nomad/blob/master/LICENSE).
[MIT](https://github.com/mullholland/ansible-role-nomad/blob/master/LICENSE).

## [Author Information](#author-information)

[Mullholland](https://mullholland.net)
[Mullholland](https://mullholland.net)
