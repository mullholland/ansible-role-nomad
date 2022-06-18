# [nomad](#nomad)

|GitHub|GitLab|
|------|------|
|[![github](https://github.com/mullholland/ansible-role-nomad/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-nomad/actions)|[![gitlab](https://gitlab.com/mullholland/ansible-role-nomad/badges/master/pipeline.svg)](https://gitlab.com/mullholland/ansible-role-nomad)|[![quality](https://img.shields.io/ansible/quality/unset)](https://galaxy.ansible.com/mullholland/nomad)|

description

## [Role Variables](#role-variables)

These variables are set in `defaults/main.yml`:
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


## [Example Playbook](#example-playbook)

This example is taken from `molecule/default/converge.yml` and is tested on each push, pull request and release.
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

The machine needs to be prepared in CI this is done using `molecule/default/prepare.yml`:
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





## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

-   [debian9](https://hub.docker.com/r/mullholland/docker-molecule-debian9)
-   [debian10](https://hub.docker.com/r/mullholland/docker-molecule-debian10)
-   [debian11](https://hub.docker.com/r/mullholland/docker-molecule-debian11)
-   [ubuntu1804](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu1804)
-   [ubuntu2004](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu2004)
-   [ubuntu2204](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu2204)
-   [centos7](https://hub.docker.com/r/mullholland/docker-molecule-centos7)
-   [centos-stream8](https://hub.docker.com/r/mullholland/docker-molecule-centos-stream8)
-   [ubi8](https://hub.docker.com/r/mullholland/docker-molecule-ubi8)
-   [fedora35](https://hub.docker.com/r/mullholland/docker-molecule-fedora35)
-   [fedora36](https://hub.docker.com/r/mullholland/docker-molecule-fedora36)
-   [amazonlinux](https://hub.docker.com/r/mullholland/docker-molecule-amazonlinux)
-   [rockylinux8](https://hub.docker.com/r/mullholland/docker-molecule-rockylinux8)
-   [almalinux8](https://hub.docker.com/r/mullholland/docker-molecule-almalinux8)

The minimum version of Ansible required is 2.10, tests have been done to:

-   The previous versions.
-   The current version.
-   The [devel](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-devel-from-github-with-pip) version.

This Role has the following additional molecule test scenarios:
-   cluster

Details can be found in ```molecule/```


## [Exceptions](#exceptions)

Some variations of the build matrix do not work. These are the variations and reasons why the build won't work:

| variation                 | reason                 |
|---------------------------|------------------------|
| centos-stream9 | No hashicorp Repository |


If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-nomad/issues)

## [License](#license)

MIT


## [Author Information](#author-information)

[Mullholland](https://github.com/mullholland)

## [Special Thanks](#special-thanks)

Template inspired by [Robert de Bock](https://github.com/robertdebock)
