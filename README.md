# Ansible Homelab Infrastructure

- [Ansible Homelab Infrastructure](#ansible-homelab-infrastructure)
  - [Project Structure](#project-structure)
  - [Available Roles](#available-roles)
    - [filesystem\_sync](#filesystem_sync)
    - [install\_docker](#install_docker)
    - [ping\_hosts](#ping_hosts)
    - [setup\_nfs\_mount](#setup_nfs_mount)
    - [update\_system\_packages](#update_system_packages)
    - [users\_and\_groups\_sync](#users_and_groups_sync)
  - [Usage](#usage)
    - [Running Playbooks](#running-playbooks)
    - [Syntax Checking](#syntax-checking)
    - [Linting](#linting)
  - [Configuration](#configuration)
    - [Inventory Structure](#inventory-structure)
    - [Variable Naming Convention](#variable-naming-convention)
  - [Requirements](#requirements)
  - [Security Considerations](#security-considerations)
  - [Contributing](#contributing)
  - [License](#license)

This repository contains Ansible playbooks and roles for managing homelab infrastructure. It provides automated configuration management for servers, including user management, filesystem synchronization, Docker installation, and network storage mounting.

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
│       │   ├── external.yml      # Variables for external provider hosts
│       │   └── onprem.yml        # Variables for on-premise hosts
│       ├── hosts.yml             # Host definitions and groupings
│       └── README.md             # Example inventory documentation
└── playbooks/                     # Ansible playbooks and roles
    ├── docker.playbook.yml        # Main Docker deployment playbook
    ├── setup_initial_external.playbook.yml # Initial setup for external VPS servers
    ├── upgrade_hosts.playbook.yml # System upgrade playbook
    └── roles/                     # Reusable Ansible roles
        ├── filesystem_sync/       # Creates and manages directories
        ├── install_docker/        # Installs Docker and Docker Compose
        ├── ping_hosts/           # Basic connectivity test
        ├── setup_hostname/        # Configures system hostname
        ├── setup_locale_and_timezone/ # Configures locale and timezone
        ├── setup_nfs_mount/      # Configures NFS mounts
        ├── sync_authorized_keys/  # Manages SSH authorized keys
        ├── update_system_packages/# System package updates
        └── users_and_groups_sync/ # User and group management
```

## Available Playbooks

### docker.playbook.yml

Comprehensive Docker host setup that runs the following roles in order:
1. `update_system_packages` - Updates system packages
2. `users_and_groups_sync` - Synchronizes users and groups
3. `filesystem_sync` - Creates required directories
4. `install_docker` - Installs Docker and related tools
5. `setup_nfs_mount` - Configures network storage mounts

### setup_initial_external.playbook.yml

Initial configuration for external VPS servers. Currently implements:
- SSH authorized keys synchronization
- User migration from default cloud provider user
- (Additional roles can be added as needed)

### upgrade_hosts.playbook.yml

Simple playbook to update system packages across all hosts.

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

### setup_hostname

Configures the system hostname to match the inventory hostname. Includes a handler to reboot if the hostname changes.

### setup_locale_and_timezone

Configures system locale and timezone settings. Also installs and configures NTP service with OS-specific handling.

**Note**: Currently has hardcoded values that should be made configurable:
- Timezone: `Europe/Madrid` (should use a variable like `lab_timezone`)
- Locale: `es_ES.UTF-8` (should use a variable like `lab_locale`)

### sync_authorized_keys

Validates and manages SSH authorized keys for users. Features:
- Comprehensive SSH key format validation
- Support for multiple key types (ssh-rsa, ssh-ed25519, ecdsa-sha2-*, etc.)
- Flexible per-key configuration options

Requires `acl.authorized_keys` list with entries containing:
- `user`: Target username (required)
- `key`: SSH public key (required)
- `state`: present/absent (optional)
- `exclusive`: Remove other keys (optional)
- Additional options documented in the role

### setup_nfs_mount

Configures NFS mounts on target hosts. Supports both Debian and Arch Linux distributions. Requires:

- `setup_nfs_mount_mounts`: List of mount configurations with source and target paths
- `filesystem.nfs.server`: NFS server hostname/IP
- `filesystem.nfs.fstype`: Filesystem type (e.g., nfs4)
- `filesystem.nfs.options`: Mount options

### update_system_packages

Updates system packages using OS-appropriate package managers:
- Debian/Ubuntu: Uses `apt` to update cache and upgrade packages
- Arch Linux: Uses `pacman` to perform system upgrade

Can be skipped by setting `skip_update: true` on specific hosts.

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

### Key Variables

#### Global Settings (all.yml)
- `lab_debug`: Enable debug mode (default: false)
- `skip_update`: Skip system updates (default: false)
- `exposed_ssh_port`: SSH port for remote access (default: 22)

#### Access Control (acl)
- `acl.default_user`: Default username for file ownership
- `acl.default_group`: Default group for file ownership
- `acl.users`: List of users to create/manage
- `acl.groups`: List of groups to create/manage
- `acl.authorized_keys`: SSH keys to deploy

#### Filesystem Configuration
- `filesystem.root_folder`: Main directory for homelab files
- `filesystem.storage_folder`: Directory for storage mounts
- `filesystem.docker_volume_folder`: Docker volume storage location
- `filesystem.nfs.*`: NFS mount configuration
- `filesystem.cifs.*`: CIFS/SMB mount configuration

### Inventory Structure

The inventory follows a hierarchical structure with:

- Host definitions in `hosts.yml`
- Group variables in `group_vars/` directory
- Variables cascade from `all.yml` to specific group files

### Variable Naming Convention

Variables use a namespace prefix to avoid conflicts:

- `lab_*` for general lab variables
- `acl.*` for access control list variables
- `filesystem.*` for filesystem-related variables
- Role-specific variables use `<role_name>_*` prefix

## Requirements

- Python 3.11+
- Ansible 2.9+
- ansible-lint (for linting)
- `ansible.posix` collection (for authorized_key module)

### Supported Operating Systems

- Debian/Ubuntu
- Arch Linux
- Gentoo (partial support in some roles)

## Security Considerations

- Use Ansible Vault for sensitive data (credentials, API keys, etc.)
- Store private keys securely and never commit them
- Follow principle of least privilege for user permissions
- Keep inventory files with sensitive data out of version control
- Use custom SSH ports for external-facing servers (see `exposed_ssh_port`)
- Regularly update authorized keys and remove unused ones
- Consider using `exclusive: true` for authorized_keys on critical systems

## Best Practices

1. **Inventory Management**
   - Create separate inventories for different environments (dev, staging, production)
   - Use the example inventory as a template
   - Keep sensitive data in vault-encrypted files

2. **Variable Organization**
   - Define common variables in `group_vars/all.yml`
   - Override specific settings in group or host vars
   - Use consistent naming with proper prefixes

3. **Role Development**
   - Make roles idempotent - running them multiple times should be safe
   - Add proper tags for selective execution
   - Document required variables in role README files

4. **Testing**
   - Always run with `--check` first on production systems
   - Test on a subset of hosts before rolling out broadly
   - Use `--diff` to see what changes will be made

## Contributing

1. Create feature branches for changes
2. Follow the established code style
3. Update documentation as needed
4. Run syntax checks and linting before committing
5. Create pull requests for review

## Host Groups

The inventory supports various host groupings:

- **cloud**: Traditional cloud VPS hosts
- **external**: External provider hosts (e.g., VPS from various providers)
- **docker**: Hosts designated for Docker containers
- **servers**: On-premise physical or virtual servers
- **raspberrypi**: Raspberry Pi devices
- **laptop/workstation**: Personal devices (often with `skip_update: true`)
- **homelab**: Custom grouping for homelab infrastructure
- **onprem**: All on-premise hosts

## Common Use Cases

### Setting Up a New Docker Host

```bash
ansible-playbook -i inventories/homelab/hosts.yml playbooks/docker.playbook.yml -l new-docker-host
```

### Initial VPS Configuration

```bash
ansible-playbook -i inventories/homelab/hosts.yml playbooks/setup_initial_external.playbook.yml -l external
```

### System-wide Updates

```bash
ansible-playbook -i inventories/homelab/hosts.yml playbooks/upgrade_hosts.playbook.yml
```

### Testing Connectivity

```bash
ansible -i inventories/homelab/hosts.yml all -m ping
```

## License

[Specify your license here]
