# Ansible Homelab Infrastructure

This repository contains Ansible playbooks and roles for managing a homelab infrastructure with a focus on Docker deployment, user management, filesystem synchronization, and NFS mount configuration.

## Project Structure

```
ansible/
├── .gitignore                     # Git ignore file
├── inventories/                   # Host inventories and group variables
│   └── example/                   # Example inventory demonstrating structure
│       ├── group_vars/            # Group-specific variables
│       │   ├── all.yml           # Variables for all hosts
│       │   ├── cloud.yml         # Variables for cloud hosts
│       │   ├── docker.yml        # Variables for Docker hosts
│       │   └── onprem.yml        # Variables for on-premise hosts
│       ├── hosts.yml             # Host definitions and groupings
│       └── README.md             # Example inventory documentation
└── playbooks/                     # Ansible playbooks and roles
    ├── docker.playbook.yml        # Main Docker deployment playbook
    └── roles/                     # Reusable Ansible roles
        ├── filesystem_sync/       # Creates and manages directories
        ├── install_docker/        # Installs Docker and Docker Compose
        ├── ping_hosts/           # Basic connectivity test
        ├── setup_nfs_mount/      # Configures NFS mounts
        ├── update_system_packages/# System package updates
        └── users_and_groups_sync/ # User and group management
```

## Available Roles

### filesystem_sync
Creates and synchronizes directories across hosts. Requires:
- `filesystem_sync_target_directories`: List of directories to create
- `filesystem_sync_target_directories_owner`: Owner for directories
- `filesystem_sync_target_directories_group`: Group for directories

### install_docker
Installs Docker and Docker Compose packages, ensures the Docker service is enabled and running.

### ping_hosts
Simple connectivity test role that pings hosts and displays results.

### setup_nfs_mount
Configures NFS mounts on target hosts. Supports both Debian and Arch Linux distributions. Requires:
- `setup_nfs_mount_mounts`: List of mount configurations with source and target paths
- `filesystem.nfs.server`: NFS server hostname/IP
- `filesystem.nfs.fstype`: Filesystem type (e.g., nfs4)
- `filesystem.nfs.options`: Mount options

### update_system_packages
Updates system packages on target hosts (implementation not shown in current code).

### users_and_groups_sync
Synchronizes users and groups across all hosts. Requires:
- `acl.users`: List of users with their properties (name, uid, groups)
- `acl.groups`: List of groups with their properties (name, gid)

## Usage

### Running Playbooks

Execute playbooks against a specific inventory:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/docker.playbook.yml
```

Target specific hosts or groups:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/docker.playbook.yml -l cloud
```

### Syntax Checking

Validate playbook syntax before running:

```bash
ansible-playbook --syntax-check -i inventories/example/hosts.yml playbooks/docker.playbook.yml
```

### Linting

Check for best practices and common issues:

```bash
ansible-lint playbooks/
```

## Configuration

### Inventory Structure

The inventory follows a hierarchical structure with:
- Host definitions in `hosts.yml`
- Group variables in `group_vars/` directory
- Variables cascade from `all.yml` to specific group files

### Variable Naming Convention

Variables use a namespace prefix to avoid conflicts:
- `lab_` for general lab variables
- `acl_` for access control list variables
- `filesystem_` for filesystem-related variables
- Use your own prefix for custom variables

## Requirements

- Python 3.11+
- Ansible
- ansible-lint (for linting)

## Security Considerations

- Use Ansible Vault for sensitive data
- Store private keys securely
- Follow principle of least privilege for user permissions
- Keep inventory files with sensitive data out of version control

## Contributing

1. Create feature branches for changes
2. Follow the established code style
3. Update documentation as needed
4. Run syntax checks and linting before committing
5. Create pull requests for review

## License

[Specify your license here]