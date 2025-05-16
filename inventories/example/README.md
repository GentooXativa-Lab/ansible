# Example Inventory

This is an example inventory structure that demonstrates:

- How to organize hosts into logical groups
- How to set variables at different levels (all hosts, group-specific)
- How to define connection methods for different types of hosts

## Structure

- `hosts.yml` - Main inventory file defining hosts and groups
- `group_vars/` - Group-specific variables
  - `all.yml` - Variables applied to all hosts
  - `cloud.yml` - Variables for cloud-hosted servers
  - `onprem.yml` - Variables for on-premise servers

## Usage

Run commands against this inventory with:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/your-playbook.yml
```

To target a specific group:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/your-playbook.yml -l cloud
```