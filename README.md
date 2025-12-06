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
│   ├── reboot-hosts.yml             # Server reboot management
│   ├── check-docker-containers.yml  # Docker container monitoring
│   ├── check-open-ports.yml         # Open port scanning
│   ├── export-mysql-databases.yml   # MySQL database backup
│   ├── export-mariadb-databases.yml # MariaDB database backup
│   ├── export-postgres-databases.yml # PostgreSQL database backup
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

### reboot-hosts.yml

Performs controlled server reboots with wait-for-reconnection logic.

```bash
ansible-playbook playbooks/reboot-hosts.yml
```

**Features:**
- Graceful system reboot
- Automatic reconnection verification
- Configurable reboot timeout
- Post-reboot connectivity checks

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

### export-mysql-databases.yml

Exports MySQL/MariaDB databases to backup files using mysqldump. **Fully Semaphore-compatible** with extensive variable configuration.

```bash
# Export all databases
ansible-playbook playbooks/export-mysql-databases.yml

# Export with authentication
ansible-playbook playbooks/export-mysql-databases.yml -e "mysql_username=backup mysql_pass=secret"

# Export specific databases
ansible-playbook playbooks/export-mysql-databases.yml -e "databases_to_backup=['wordpress','nextcloud']"

# With retention policy
ansible-playbook playbooks/export-mysql-databases.yml -e "backup_retention_days=30 minimum_backups=5"
```

**Features:**
- Exports all databases or specific ones
- Automatic compression with configurable levels
- Includes routines, triggers, and events
- Uses single-transaction for consistency
- Organized backups by database name with timestamps
- Saves to `/mnt/share/backup/mysql/<databaseName>/`
- Automatic cleanup with retention policies
- Fully variable-driven for Semaphore integration
- Socket or TCP connection support
- Comprehensive error handling

**Semaphore Setup:**
See [export-mysql-databases-vars.md](playbooks/export-mysql-databases-vars.md) for complete variable documentation and Semaphore configuration examples.

---

### export-mariadb-databases.yml

Exports MariaDB databases to backup files using mariadb-dump. **Fully Semaphore-compatible** with extensive variable configuration.

```bash
# Export all databases
ansible-playbook playbooks/export-mariadb-databases.yml

# Export with authentication
ansible-playbook playbooks/export-mariadb-databases.yml -e "mariadb_username=backup mariadb_pass=secret"

# Export specific databases
ansible-playbook playbooks/export-mariadb-databases.yml -e "databases_to_backup=['wordpress','nextcloud']"
```

**Features:**
- Native MariaDB support using mariadb-dump
- Exports all databases or specific ones
- Automatic compression with configurable levels
- Organized backups by database name with timestamps
- Retention policy with minimum backup guarantees
- Fully variable-driven for Semaphore integration

---

### export-postgres-databases.yml

Exports PostgreSQL databases to backup files using pg_dump. **Fully Semaphore-compatible** with extensive variable configuration.

```bash
# Export all databases
ansible-playbook playbooks/export-postgres-databases.yml

# Export with authentication
ansible-playbook playbooks/export-postgres-databases.yml -e "postgres_username=backup postgres_pass=secret"

# Export specific databases
ansible-playbook playbooks/export-postgres-databases.yml -e "databases_to_backup=['webapp','analytics']"

# With retention policy
ansible-playbook playbooks/export-postgres-databases.yml -e "backup_retention_days=30 minimum_backups=5"
```

**Features:**
- Full pg_dump support with configurable format (plain/custom)
- Automatic database discovery or explicit list
- Compression with configurable gzip levels (1-9)
- Include/exclude CREATE DATABASE and DROP statements
- Schema-only or data-only backup options
- Retention policy with minimum backup guarantees
- Saves to `/mnt/backup.fuhlig.de/databases/postgresql/<databaseName>/`
- Detailed backup summary with file sizes
- Socket or TCP connection support

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

Playbook reports are saved to `playbooks/output/`:
- `containers-<hostname>.json` - Docker container reports
- `packages-<hostname>.json` - Package inventory exports

Database backups are saved to:
- **MySQL:** `/mnt/share/backup/mysql/<databaseName>/`
- **MariaDB:** `/mnt/backup.fuhlig.de/databases/mariadb/<databaseName>/`
- **PostgreSQL:** `/mnt/backup.fuhlig.de/databases/postgresql/<databaseName>/`

Files are timestamped: `<databaseName>_<timestamp>.sql.gz`

## License

Private repository - All rights reserved.
