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
│   ├── backup-docker-volumes.yml    # Docker volume backups
│   ├── cleanup-old-backups.yml      # Backup retention cleanup
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
- Saves to `/mnt/storage/databases/postgresql/<databaseName>/`
- Detailed backup summary with file sizes
- Socket or TCP connection support

---

### backup-docker-volumes.yml

Backs up all Docker volumes to compressed tar archives.

```bash
# Backup all volumes
ansible-playbook playbooks/backup-docker-volumes.yml

# Custom backup directory
ansible-playbook playbooks/backup-docker-volumes.yml -e "backup_dir=/custom/path"

# Exclude specific volumes
ansible-playbook playbooks/backup-docker-volumes.yml -e "exclude_volumes=['cache_vol','temp_data']"
```

**Features:**
- Backs up all Docker volumes to compressed `.tar.gz` archives
- Each volume gets its own folder with timestamped backups
- Uses Alpine container for portable, permission-safe backups
- Supports excluding specific volumes
- Skips hosts without Docker installed
- Saves to `/mnt/storage/docker_volumes/<volume_name>/`

**Backup Structure:**
```
/mnt/storage/docker_volumes/
├── myvolume1/
│   ├── myvolume1-2025-12-06_0451.tar.gz
│   └── myvolume1-2025-12-07_0300.tar.gz
├── myvolume2/
│   └── myvolume2-2025-12-06_0451.tar.gz
└── ...
```

---

### cleanup-old-backups.yml

Cleans up old backup files based on a grandfather-father-son retention policy. **Fully Semaphore-compatible** with all options configurable via variables.

```bash
# Preview what would be deleted (dry run)
ansible-playbook playbooks/cleanup-old-backups.yml -e "dry_run=true"

# Run with default retention (7 daily, 4 weekly, 12 monthly)
ansible-playbook playbooks/cleanup-old-backups.yml

# Custom retention policy
ansible-playbook playbooks/cleanup-old-backups.yml -e "keep_daily_days=14 keep_weekly_weeks=8 keep_monthly_months=24"

# Different backup directory
ansible-playbook playbooks/cleanup-old-backups.yml -e "backup_directory=/custom/backup/path"
```

**Features:**
- Grandfather-Father-Son retention policy:
  - **Daily:** Keep all backups from the last 7 days (configurable)
  - **Weekly:** Keep one backup per week for the last 4 weeks (configurable)
  - **Monthly:** Keep one backup per month for the last 12 months (configurable)
- Applies to all backup types:
  - MariaDB database exports
  - MySQL database exports
  - PostgreSQL database exports
  - Docker volume backups
- Dry run mode to preview deletions without removing files
- Minimum backup guarantee (always keeps at least 1 backup per item)
- Detailed summary of files kept and deleted
- Runs locally on the Ansible controller

**Semaphore Variables:**
| Variable | Default | Description |
|----------|---------|-------------|
| `keep_daily_days` 	| 7 		| Days to keep daily backups |
| `keep_weekly_weeks` 	| 4 		| Weeks to keep weekly backups |
| `keep_monthly_months` | 12 		| Months to keep monthly backups |
| `minimum_backups` 	| 1 		| Minimum backups to always keep |
| `dry_run_mode` 	| false 	| Preview mode (no deletions) |
| `backup_directory` 	| /mnt/storage 	| Base backup directory |
| `enable_verbose` 	| true 		| Show detailed output |

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
## Output

Playbook reports are saved to `playbooks/output/`:
- `containers-<hostname>.json` - Docker container reports
- `packages-<hostname>.json` - Package inventory exports

Database backups are saved to:
- **MySQL:** `/mnt/storage/databases/mysql/<databaseName>/`
- **MariaDB:** `/mnt/storage/databases/mariadb/<databaseName>/`
- **PostgreSQL:** `/mnt/storage/databases/postgresql/<databaseName>/`

Docker volume backups are saved to:
- **Docker Volumes:** `/mnt/storage/docker_volumes/<volumeName>/`

Files are timestamped: `<name>-<timestamp>.tar.gz` or `<name>_<timestamp>.sql.gz`

## License

Private repository - All rights reserved.
