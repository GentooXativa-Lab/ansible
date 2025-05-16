# Example Ansible Inventory

This is an example inventory structure that demonstrates how to organize hosts and groups for an Ansible homelab environment.

## Structure

```
inventories/example/
├── hosts.yml               # Main inventory file with host definitions
├── group_vars/            # Variables specific to groups
│   ├── all.yml           # Variables that apply to all hosts
│   ├── cloud.yml         # Variables for cloud-hosted servers
│   ├── docker.yml        # Variables for Docker hosts
│   └── onprem.yml        # Variables for on-premise servers
└── README.md             # This file
```

## hosts.yml

The main inventory file organizes hosts into logical groups:

- **cloud**: Cloud-hosted servers (VPS)
- **docker**: Dedicated Docker container hosts
- **raspberrypi**: Raspberry Pi nodes (RPi 4 and 5)
- **servers**: On-premise physical or virtual servers
- **laptop**: Development laptops
- **workstation**: Development workstations
- **homelab**: Custom grouping that includes specific children groups

### Host Properties

Each host can have:
- `ansible_host`: The actual hostname or IP address to connect to
- `host_tags`: Custom tags for categorization (e.g., `vps`, `docker`, `vm`, `rpi4`)

## Group Variables

### all.yml
Contains variables that apply to all hosts:
- ACL configurations (users and groups)
- Filesystem paths and permissions
- NFS and CIFS mount configurations
- Default settings for the entire inventory

### cloud.yml
Specific to cloud-hosted servers:
- SSH connection settings
- Authentication method
- User credentials

### docker.yml
Specific to Docker hosts:
- Docker volume storage location

### onprem.yml
Specific to on-premise servers:
- SSH connection settings
- Authentication method

## Usage

1. Copy this example directory as a template for your own inventory:
   ```bash
   cp -r inventories/example inventories/mylab
   ```

2. Update the hosts and variables according to your environment:
   - Modify hostnames and IP addresses in `hosts.yml`
   - Adjust user credentials and paths in group variables
   - Add or remove groups as needed

3. Test your inventory:
   ```bash
   ansible-inventory -i inventories/mylab/hosts.yml --list
   ```

4. Run a playbook using this inventory:
   ```bash
   ansible-playbook -i inventories/mylab/hosts.yml playbooks/test.playbook.yml
   ```

## Best Practices

- Use descriptive host names that indicate purpose or location
- Group hosts logically based on their role or characteristics
- Keep sensitive data in Ansible Vault (not shown in this example)
- Use consistent naming conventions throughout
- Document any custom variables or configurations
- Test connectivity before running complex playbooks:
  ```bash
  ansible -i inventories/mylab/hosts.yml all -m ping
  ```

## Variable Precedence

Variables are applied in the following order (from lowest to highest precedence):
1. all.yml (applies to all hosts)
2. Group-specific variables (e.g., cloud.yml, docker.yml)
3. Host-specific variables (defined in hosts.yml)
4. Command-line variables

This means more specific variables override more general ones.