# Semaphore UI Complete Tutorial

A step-by-step guide to set up and run your first playbook in Semaphore UI.

---

## What is Semaphore?

Semaphore is a web UI for running Ansible playbooks. Instead of running `ansible-playbook` from the command line, you configure everything in a web interface and click "Run".

**Key concepts:**
- **Project** = A container for all your Ansible stuff
- **Key Store** = Where you store SSH keys and passwords
- **Inventory** = Your list of servers (like Ansible's inventory file)
- **Environment** = Variables passed to your playbook
- **Repository** = Where your playbook files live (Git repo or local path)
- **Task Template** = The actual "run this playbook" configuration

---

## Step 1: Create a Project

1. Log in to Semaphore
2. Click **"New Project"** (or the + button)
3. Enter a name like `My Ansible Project`
4. Click **Create**

You now have an empty project. Everything else goes inside this project.

---

## Step 2: Add Your SSH Key (Key Store)

Semaphore needs SSH access to your servers.

1. In your project, click **Key Store** in the left menu
2. Click **New Key**
3. Fill in:
   - **Key Name**: `SSH Key` (or any name you want)
   - **Type**: Select `SSH Key`
   - **Username**: The SSH user (e.g., `root` or `ansible`)
   - **Private Key**: Paste your private SSH key content (the one that starts with `-----BEGIN`)
4. Click **Create**

**To get your private key:**
```bash
# On Linux/Mac
cat ~/.ssh/id_rsa

# Or if using ed25519
cat ~/.ssh/id_ed25519
```

---

## Step 3: Add Password/Secret Keys (Optional)

If your playbook needs passwords (like MySQL password):

1. Go to **Key Store** → **New Key**
2. Fill in:
   - **Key Name**: `MySQL Password`
   - **Type**: Select `Login with password`
   - **Username**: `backup_user` (or whatever username)
   - **Password**: Your actual password
3. Click **Create**

---

## Step 4: Create an Inventory

The inventory tells Semaphore which servers to run playbooks on.

1. Click **Inventory** in the left menu
2. Click **New Inventory**
3. Fill in:
   - **Name**: `My Servers`
   - **User Credentials**: Select your SSH key from Step 2
   - **Type**: Select `Static` (simplest option)
4. In the **Inventory** text box, enter your servers:

**Simple format:**
```ini
[all]
192.168.1.100
192.168.1.101
server1.example.com
```

**With groups:**
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com

[all:children]
webservers
databases
```

**With SSH port or user override:**
```ini
[all]
192.168.1.100 ansible_port=2222
192.168.1.101 ansible_user=admin
```

5. Click **Create**

---

## Step 5: Add Your Repository

Semaphore needs to know where your playbook files are.

### Option A: Git Repository (Recommended)

1. Click **Repositories** in the left menu
2. Click **New Repository**
3. Fill in:
   - **Name**: `My Playbooks`
   - **URL**: Your Git URL, e.g.:
     - `https://github.com/username/ansible-playbooks.git`
     - `git@github.com:username/ansible-playbooks.git`
   - **Branch**: `main` (or `master`)
   - **Access Key**: Select `None` for public repos, or create an SSH key for private repos
4. Click **Create**

### Option B: Local Path

If playbooks are on the same server as Semaphore:

1. Click **Repositories** → **New Repository**
2. Fill in:
   - **Name**: `Local Playbooks`
   - **URL**: Local path like `/home/user/ansible-playbooks`
   - **Branch**: Leave empty
   - **Access Key**: Select `None`
3. Click **Create**

---

## Step 6: Create an Environment (Variables)

Environments hold variables that get passed to your playbook.

1. Click **Environment** in the left menu
2. Click **New Environment**
3. Fill in:
   - **Name**: `MySQL Backup Config`
   - **Extra Variables**: Enter your variables in JSON or YAML format:

**JSON format:**
```json
{
  "mysql_username": "backup_user",
  "mysql_pass": "your_password_here",
  "backup_directory": "/mnt/backup/mysql",
  "enable_compression": true
}
```

**YAML format:**
```yaml
mysql_username: backup_user
mysql_pass: "your_password_here"
backup_directory: /mnt/backup/mysql
enable_compression: true
```

4. Click **Create**

**Tip:** Create multiple environments for different scenarios:
- `Production Config`
- `Testing Config`
- `Minimal Config`

---

## Step 7: Create a Task Template

This is where you combine everything to run a playbook.

1. Click **Task Templates** in the left menu
2. Click **New Template**
3. Fill in the form:

| Field | Value | Explanation |
|-------|-------|-------------|
| **Name** | `MySQL Backup` | Display name for this task |
| **Playbook Filename** | `playbooks/export-mysql-databases.yml` | Path to playbook file (relative to repo root) |
| **Inventory** | Select `My Servers` | Which servers to run on |
| **Repository** | Select `My Playbooks` | Where the playbook file is |
| **Environment** | Select `MySQL Backup Config` | Variables to use |

4. Click **Create**

---

## Step 8: Run Your First Task

1. Go to **Task Templates**
2. Find your template and click **Run** (play button)
3. A popup appears - just click **Run** again
4. Watch the output in real-time!

**What you'll see:**
- Green = Success
- Red = Failed
- The full Ansible output appears in the log

---

## Step 9: Set Up a Schedule (Optional)

To run playbooks automatically:

1. Go to **Task Templates**
2. Click on your template name to edit it
3. Scroll down to **Schedule** section
4. Click **Add Schedule**
5. Enter a cron expression:

| Schedule | Cron Expression |
|----------|-----------------|
| Every day at 2 AM | `0 2 * * *` |
| Every hour | `0 * * * *` |
| Every Sunday at 3 AM | `0 3 * * 0` |
| Every 6 hours | `0 */6 * * *` |
| Monday-Friday at 6 PM | `0 18 * * 1-5` |

6. Click **Save**

---

## Complete Example: MySQL Backup Setup

Here's everything you need to set up the MySQL backup playbook:

### Key Store Entry
- **Name**: `Server SSH Key`
- **Type**: SSH Key
- **Username**: `root`
- **Private Key**: (your SSH private key)

### Inventory
**Name**: `Database Servers`
```ini
[databases]
192.168.1.50
```

### Repository
- **Name**: `Ansible Playbooks`
- **URL**: `https://github.com/yourusername/ansible.git`
- **Branch**: `main`

### Environment
**Name**: `MySQL Backup Vars`
```json
{
  "mysql_username": "backup_user",
  "mysql_pass": "SecurePassword123",
  "backup_directory": "/mnt/backup/mysql",
  "enable_compression": true,
  "backup_retention_days": 30
}
```

### Task Template
- **Name**: `Daily MySQL Backup`
- **Playbook Filename**: `playbooks/export-mysql-databases.yml`
- **Inventory**: `Database Servers`
- **Repository**: `Ansible Playbooks`
- **Environment**: `MySQL Backup Vars`
- **Schedule**: `0 2 * * *` (2 AM daily)

---

## Troubleshooting

### "Permission denied" when connecting to servers

**Cause**: SSH key issue

**Fix**:
1. Make sure your SSH key is correct in Key Store
2. Check that the username matches what's on the server
3. Test manually: `ssh -i ~/.ssh/id_rsa user@server`

### "Playbook not found"

**Cause**: Wrong path in Task Template

**Fix**:
1. The path is relative to your repository root
2. If your repo structure is:
   ```
   my-repo/
   ├── playbooks/
   │   └── backup.yml
   ```
   Then use: `playbooks/backup.yml`

### "Variable undefined"

**Cause**: Missing variable in Environment

**Fix**:
1. Check what variables the playbook needs
2. Add them to your Environment configuration
3. Make sure JSON/YAML syntax is valid

### Task runs but nothing happens

**Cause**: Inventory doesn't match playbook's `hosts:` value

**Fix**:
1. Check what `hosts:` is set to in your playbook
2. Make sure your inventory has a group or host matching that name
3. Example: If playbook says `hosts: databases`, inventory needs `[databases]` group

### "Host key verification failed"

**Cause**: Server not in known_hosts

**Fix**: Add this to your Environment variables:
```json
{
  "ansible_ssh_common_args": "-o StrictHostKeyChecking=no"
}
```

Or manually SSH to each server once to accept the key.

---

## Quick Reference: Semaphore Navigation

| Menu Item | What It Does |
|-----------|--------------|
| **Dashboard** | See recent task runs |
| **Task Templates** | Create/edit/run playbooks |
| **Inventory** | Manage server lists |
| **Environment** | Manage variables |
| **Key Store** | Manage SSH keys and passwords |
| **Repositories** | Manage Git repos |
| **History** | See all past task runs |

---

## Next Steps

- Read the [variable reference](export-mysql-databases-vars.md) for all MySQL backup options
- Create different environments for different backup strategies
- Set up notifications in Project Settings → Integrations
