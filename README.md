# ansible-ssm-custom

--------------------

Ansible SSM + Custom Requirements

Project containing a set of roles required for SSM service.

## Requirements

- `python3`
- `ansible`
- [session-manager-plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html) installed in the ansible controller

## Dependencies

EC2 instances require an IAM Role with the following policies attached to allow the roles to function correctly.

- **AmazonSSMManagedInstanceCore**: The standard AWS managed policy for SSM.
- **Custom policy for IMDS**: A custom IAM policy is required to allow the `imds_manager` role to modify EC2 instance metadata options.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:ModifyInstanceMetadataOptions",
            "Resource": "arn:aws:ec2:*:*:instance/*"
        }
    ]
}
```

Create and attach an IAM Role with these policies in the AWS ec2 instances.

## Roles

- cloudwatch_agent
- ntp_conf
- ssm_agent
- ssm_plugin
- ssm_user_access
- user_password_status
- aws_cli_installer
- imds_manager

### Roles Description

- **cloudwatch_agent**: Installs, configures, and ensures the Amazon CloudWatch Agent is running.
- **ntp_conf**: Manages NTP configuration. It detects `chrony` or `ntpd`, defaulting to installing `chrony` if neither is present. It configures the service to use Amazon Time Sync Program servers.
- **ssm_agent**: Installs and manages the AWS Systems Manager Agent (`amazon-ssm-agent`), ensuring the service is running and the `ssm-user` is present.
- **ssm_plugin**: Installs the Session Manager plugin, which is necessary for using SSM Session Manager.
- **ssm_user_access**: Configures system access for the `ssm-user` by adding an entry to `/etc/security/access.conf`.
- **user_password_status**: Checks the password status (expiration, lock status) for a list of specified users and can lock accounts if required.
- **aws_cli_installer**: Installs the AWS CLI v2 if it is not already present on the system. It automatically detects the architecture (x86_64 or ARM) to download the correct package.
- **imds_manager**: Manages the EC2 Instance Metadata Service (IMDS), with functionality to enforce IMDSv2.

### `imds_manager` Role

This role is designed to manage and enforce the use of Instance Metadata Service Version 2 (IMDSv2) on EC2 instances. IMDSv2 provides enhanced security by requiring session-oriented requests.

By default, the role only checks and reports whether IMDSv2 is already enforced on the instance. It does not make any changes.

**Enforcing IMDSv2**

To actively enforce IMDSv2, you must set the `imds_enforce` variable to `true` when running the playbook. This will modify the instance's metadata options to require the use of tokens for metadata access.

**Usage Examples**

**1. Checking IMDSv2 status (Default Behavior)**

To run the role without making any changes and only report the current status of IMDSv2, simply include the role in your playbook.

```yaml
- name: Check IMDSv2 Status
  hosts: ec2_hosts
  become: true
  roles:
    - role: imds_manager
```

This will produce a debug message indicating if IMDSv2 is enforced or not.

**2. Enforcing IMDSv2**

To enforce IMDSv2 on the target instances, add the `imds_enforce` variable to your playbook or run it with an extra variable.

```yaml
- name: Enforce IMDSv2
  hosts: ec2_hosts
  become: true
  vars:
    imds_enforce: true
  roles:
    - role: aws_cli_installer
    - role: imds_manager
```

Or using the command line:

```bash
ansible-playbook ssm.yml -e "imds_enforce=true"
```

## Example Playbook

```yml
---
# Main playbook
- name: Ensure base configuration for EC2 instances
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
    - role: aws_cli_installer
    - role: imds_manager
      vars:
        imds_enforce: true # To apply changes (force IMDSv2)
```

## Author

[Alex Mendes](https://pt.linkedin.com/in/mendesalex)

<https://alexolinux.com>

![MIT License](https://img.shields.io/badge/license-MIT-brightgreen.svg)