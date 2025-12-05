# Ansible Infrastructure Management

A collection of Ansible playbooks for managing and automating server infrastructure.

## Overview

This repository contains production-ready Ansible playbooks for system administration tasks including package management, Docker container monitoring, and server maintenance.

## Requirements

- Ansible 2.9+
- Python 3.x on target hosts
- SSH access to target servers

## Project Structure

```
.
├── ansible.cfg              # Ansible configuration
├── inventory.ini            # Host inventory
├── playbooks/
│   ├── ping-hosts.yml               # Host connectivity testing
│   ├── check-docker-containers.yml  # Docker container monitoring
│   ├── check-open-ports.yml         # Open port scanning
│   ├── export-packages-inventory.yml # Package inventory export
│   ├── update-packages.yml          # System package updates
│   └── delete-test-folder.yml       # Test folder cleanup
└── playbooks/output/        # Generated reports (JSON)
```

## Playbooks

### ping-hosts.yml

Tests connectivity to all hosts in the inventory and gathers basic system information.

```bash
ansible-playbook playbooks/ping-hosts.yml
```

**Features:**
- Verifies SSH connectivity to all hosts
- Gathers minimal host facts
- Displays hostname, IP, OS version, and kernel information

---

### check-docker-containers.yml

Monitors Docker containers across hosts and checks for available image updates.

```bash
ansible-playbook playbooks/check-docker-containers.yml
```

**Features:**
- Lists all running containers
- Checks for newer image versions
- Exports detailed JSON reports

---

### check-open-ports.yml

Scans all servers for open TCP and UDP ports and generates detailed reports.

```bash
ansible-playbook playbooks/check-open-ports.yml
```

**Features:**
- Scans TCP and UDP listening ports
- Identifies processes bound to each port
- Supports both `ss` and `netstat` (automatic fallback)
- Exports detailed JSON reports per host

---

### update-packages.yml

Updates and upgrades system packages on Debian/Ubuntu hosts.

```bash
ansible-playbook playbooks/update-packages.yml
```

**Features:**
- Updates apt cache
- Performs dist-upgrade
- Cleans up unused packages

---

### export-packages-inventory.yml

Exports a complete inventory of installed packages to JSON format.

```bash
ansible-playbook playbooks/export-packages-inventory.yml
```

**Features:**
- Gathers package facts from all hosts
- Exports to JSON with metadata
- Includes OS and version information

---

### delete-test-folder.yml

Safely removes contents from a specified test directory.

```bash
ansible-playbook playbooks/delete-test-folder.yml -e "target_folder=/path/to/folder"
```

## Configuration

### Inventory

Hosts are organized in `inventory.ini`:

| Group | Description |
|-------|-------------|
| `webservers` | Web server hosts |
| `backup_servers` | Backup infrastructure |
| `production` | All production hosts |

### Connection Settings

- **SSH Port:** 2244
- **Default User:** fuhlig
- **Python Interpreter:** /usr/bin/python3

## Quick Start

1. Clone the repository:
   ```bash
   git clone git@git.fuhlig.de:fuh/Ansible.git
   cd Ansible
   ```

2. Verify connectivity:
   ```bash
   ansible all -m ping
   ```

3. Run a playbook:
   ```bash
   ansible-playbook playbooks/update-packages.yml
   ```

## Output

Playbook reports are saved to `playbooks/output/` in JSON format:
- `containers-<hostname>.json` - Docker container reports
- `packages-<hostname>.json` - Package inventory exports

## License

Private repository - All rights reserved.
