# ansible-ssm-custom TBD

--------------------

Ansible SSM + Custom Requirements

Project containing a set of roles required for SSM service. 

## Requirements

- `python3`
- `ansible`
- [session-manager-plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

## Roles

- cloudwatch_agent
- ntp_conf
- ssm_agent
- ssm_plugin
- ssm_user_access
- user_password_status

## Dependencies

ec2-instances must have the [System Manager IAM Role](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-permissions.html)

Example Playbook

```yml
---
# Update packages
- name: Ensure base configuration for SSM User Access
  hosts: ec2_hosts
  become: true

  tasks:
    - name: Update package manager cache
      ansible.builtin.package:
        name: "*"
        state: latest
        update_cache: true

      tags: [ update_package ]

  roles:
    # The order of the roles is important. They are executed sequentially
    - role: user_password_status
    - role: ntp_conf
    - role: ssm_agent
    - role: cloudwatch_agent
    - role: ssm_plugin
    - role: ssm_user_access

```

## Author

[Alex Mendes](https://pt.linkedin.com/in/mendesalex)

<https://alexolinux.com>

![MIT License](https://img.shields.io/badge/license-MIT-brightgreen.svg)
