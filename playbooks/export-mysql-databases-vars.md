# MySQL Database Export - Semaphore Variables

This document lists all configurable variables for the `export-mysql-databases.yml` playbook when using Semaphore.

## Quick Start - Minimal Configuration

For a basic setup, you only need to configure these variables in Semaphore:

```yaml
mysql_username: backup_user
mysql_pass: your_secure_password
backup_directory: /mnt/share/backup/mysql
```

## All Available Variables

### Connection Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `target_hosts` | `all` | Which hosts to run the backup on |
| `mysql_hostname` | `localhost` | MySQL server hostname |
| `mysql_port_number` | `3306` | MySQL server port |
| `mysql_username` | `root` | MySQL username for authentication |
| `mysql_pass` | `` | MySQL password (use Semaphore secrets!) |
| `mysql_socket_path` | `` | Unix socket path (overrides hostname/port if set) |

### Privilege Escalation

| Variable | Default | Description |
|----------|---------|-------------|
| `become_enabled` | `false` | Enable privilege escalation (sudo) |
| `become_user_name` | `root` | User to become when using sudo |

### Backup Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `backup_directory` | `/mnt/share/backup/mysql` | Base directory for all backups |
| `custom_timestamp` | `ansible_date_time.iso8601_basic_short` | Custom timestamp format |
| `backup_directory_mode` | `0755` | Permissions for backup directories |
| `backup_file_permissions` | `0644` | Permissions for backup files |
| `backup_file_owner` | (none) | Owner for backup files |
| `backup_file_group` | (none) | Group for backup files |

### Database Selection

| Variable | Default | Description |
|----------|---------|-------------|
| `databases_to_backup` | `[]` | Specific databases to backup (empty = all) |
| `databases_to_exclude` | `[]` | Additional databases to exclude |

Example for specific databases:
```yaml
databases_to_backup: ['wordpress', 'nextcloud', 'ecommerce']
```

Example for excluding databases:
```yaml
databases_to_exclude: ['test_db', 'old_data']
```

### Export Options

| Variable | Default | Description |
|----------|---------|-------------|
| `use_single_transaction` | `true` | Use single transaction (recommended for InnoDB) |
| `include_routines` | `true` | Include stored procedures and functions |
| `include_triggers` | `true` | Include triggers |
| `include_events` | `true` | Include scheduled events |
| `use_lock_tables` | `false` | Lock tables during backup |
| `include_drop_table` | `true` | Add DROP TABLE before CREATE TABLE |
| `include_create_db` | `true` | Add CREATE DATABASE statements |
| `use_disable_keys` | `true` | Disable keys for faster imports |

### Compression Options

| Variable | Default | Description |
|----------|---------|-------------|
| `enable_compression` | `true` | Compress backups with gzip |
| `gzip_compression_level` | `6` | Compression level (1-9, 9=best) |

### Retention Options

| Variable | Default | Description |
|----------|---------|-------------|
| `backup_retention_days` | `0` | Delete backups older than N days (0=keep all) |
| `minimum_backups` | `3` | Minimum backups to keep regardless of age |

### Output Options

| Variable | Default | Description |
|----------|---------|-------------|
| `enable_verbose` | `true` | Show detailed output |
| `ignore_errors` | `false` | Continue on errors instead of failing |

## Semaphore Configuration Examples

### Example 1: Basic Backup (All Databases)

Create these variables in your Semaphore template:

```yaml
mysql_username: backup_user
mysql_pass: "{{ vault_mysql_password }}"  # Use Semaphore secrets
backup_directory: /mnt/share/backup/mysql
```

### Example 2: Specific Databases with Compression

```yaml
mysql_username: backup_user
mysql_pass: "{{ vault_mysql_password }}"
backup_directory: /mnt/share/backup/mysql
databases_to_backup: ['wordpress', 'nextcloud']
enable_compression: true
gzip_compression_level: 9
```

### Example 3: With Retention Policy

```yaml
mysql_username: backup_user
mysql_pass: "{{ vault_mysql_password }}"
backup_directory: /mnt/share/backup/mysql
backup_retention_days: 30
minimum_backups: 5
enable_compression: true
```

### Example 4: Remote MySQL Server

```yaml
mysql_hostname: db.example.com
mysql_port_number: 3306
mysql_username: backup_user
mysql_pass: "{{ vault_mysql_password }}"
backup_directory: /mnt/share/backup/mysql
target_hosts: backup_server
```

### Example 5: Multiple Environments

For different environments, create separate Semaphore templates:

**Production:**
```yaml
mysql_hostname: prod-db.example.com
mysql_username: prod_backup
mysql_pass: "{{ vault_prod_mysql_password }}"
backup_directory: /mnt/share/backup/mysql/production
databases_to_backup: ['prod_app', 'prod_users']
backup_retention_days: 90
```

**Staging:**
```yaml
mysql_hostname: staging-db.example.com
mysql_username: staging_backup
mysql_pass: "{{ vault_staging_mysql_password }}"
backup_directory: /mnt/share/backup/mysql/staging
backup_retention_days: 7
```

## Security Best Practices

1. **Always use Semaphore Secrets** for `mysql_pass`:
   - In Semaphore UI: Environment → Secrets → Add Secret
   - Reference as: `{{ vault_mysql_password }}`

2. **Create dedicated MySQL backup user**:
   ```sql
   CREATE USER 'backup_user'@'localhost' IDENTIFIED BY 'secure_password';
   GRANT SELECT, LOCK TABLES, SHOW VIEW, TRIGGER, EVENT ON *.* TO 'backup_user'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. **Set appropriate file permissions**:
   ```yaml
   backup_directory_mode: "0750"
   backup_file_permissions: "0640"
   backup_file_owner: backup
   backup_file_group: backup
   ```

## Scheduling in Semaphore

Create a schedule for automatic backups:

1. **Daily backups at 2 AM**:
   - Cron: `0 2 * * *`

2. **Weekly full backups**:
   - Cron: `0 3 * * 0`

3. **Hourly for critical databases**:
   - Cron: `0 * * * *`
   - Use `databases_to_backup: ['critical_db']`

## Troubleshooting

### Common Issues:

1. **Permission denied on /mnt/share/backup/mysql**:
   - Enable privilege escalation: `become_enabled: true`
   - Or ensure the Ansible user has write permissions

2. **MySQL access denied**:
   - Verify credentials in Semaphore secrets
   - Check MySQL user permissions
   - Test connection: `mysql -h <host> -u <user> -p<pass> -e "SHOW DATABASES;"`

3. **Out of space**:
   - Enable compression: `enable_compression: true`
   - Set retention policy: `backup_retention_days: 30`
   - Increase compression: `gzip_compression_level: 9`

## Output Directory Structure

```
/mnt/share/backup/mysql/
├── wordpress/
│   ├── wordpress_20251206T143000.sql.gz
│   ├── wordpress_20251207T143000.sql.gz
│   └── wordpress_20251208T143000.sql.gz
├── nextcloud/
│   ├── nextcloud_20251206T143000.sql.gz
│   └── nextcloud_20251207T143000.sql.gz
└── ecommerce/
    └── ecommerce_20251208T143000.sql.gz
```

## Restoring Backups

To restore a compressed backup:

```bash
# Decompress and restore
gunzip < /mnt/share/backup/mysql/wordpress/wordpress_20251206T143000.sql.gz | mysql -u root -p wordpress

# Or in one command
zcat /mnt/share/backup/mysql/wordpress/wordpress_20251206T143000.sql.gz | mysql -u root -p wordpress
```

To restore uncompressed:

```bash
mysql -u root -p wordpress < /mnt/share/backup/mysql/wordpress/wordpress_20251206T143000.sql
```
