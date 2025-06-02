# sync_authorized_keys

This role validates and syncs SSH authorized keys for users across hosts.

## Features

1. **Validation**: Validates all authorized keys before applying them
   - Checks that required fields (`user` and `key`) are present
   - Validates SSH key format (supports ssh-rsa, ssh-dss, ssh-ed25519, ecdsa-sha2-*)
   - Ensures keys are non-empty strings

2. **Flexible Configuration**: Supports various options per key:
   - `state`: present (default) or absent
   - `exclusive`: whether to remove all other keys (default: false)
   - `manage_dir`: whether to create .ssh directory (default: true)
   - `path`: custom path for authorized_keys file
   - `comment`: comment for the key
   - `key_options`: SSH key options
   - `validate_certs`: certificate validation (default: true)

## Requirements

- `ansible.posix` collection (for the `authorized_key` module)

## Role Variables

Configure authorized keys in `group_vars/all.yml` under the `acl.authorized_keys` list:

```yaml
acl:
  authorized_keys:
  - user: "labuser"
    key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC... user@example.com"
    state: present
    comment: "Main workstation key"
  - user: "labuser"
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG... backup@example.com"
    state: present
```

## Example Playbook

```yaml
- hosts: all
  roles:
    - sync_authorized_keys
```

## Validation Process

The role performs the following validation steps:

1. Checks that `acl.authorized_keys` is defined and is a list
2. Validates each key entry has required fields (`user` and `key`)
3. Validates SSH key format using regex pattern matching
4. Only applies keys that pass all validation checks

If any validation fails, the playbook will stop with a descriptive error message.