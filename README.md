# Ansible Playbooks Collection

Collection of Ansible playbooks for automated Debian system updates and LXC container management, designed for use with Semaphore.

## Playbooks

### init-lxc.yaml
Initial setup playbook for LXC containers:
- Detects operating system (Ubuntu/Debian)
- Configures system repositories
- Sets up basic system utilities
- Configures security settings

### update-lxc.yaml
General Debian update playbook with automatic GPG key management:
- Updates package lists
- Handles GPG key management automatically
- Performs system updates
- Manages automatic updates configuration

## Usage with Semaphore

1. Configure environment variables in Semaphore:
```bash
ANSIBLE_USER=your_user
```

2. Run playbooks through Semaphore:
- Use `init-lxc.yaml` for initial container setup
- Use `update-lxc.yaml` for regular system updates

## Requirements

- Ansible 2.9 or higher
- Python 3.6 or higher
- Target systems: Debian/Ubuntu
- Semaphore for orchestration

## Structure

- `playbooks/`: Contains all Ansible playbooks
  - `init-lxc.yaml`: Initial container setup
  - `update-lxc.yaml`: System updates
- `collections/`: Ansible collections dependencies

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.