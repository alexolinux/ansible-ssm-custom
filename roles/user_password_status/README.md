# User Password Status

This role checks the password status and expiration details for a list of specified users.

## Requirements

None.

## Role Variables

- `users_to_check`: A list of users to check the password status for.

Defaults to:
```yaml
users_to_check:
  - root
  - ec2-user
  - ssm-user
```

## Dependencies

None.

## Example Playbook

```yaml
- hosts: servers
  roles:
     - user_password_status
```

## License

MIT

## Author Information

Alex M. Barbosa
