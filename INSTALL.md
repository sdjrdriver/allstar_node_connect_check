# Installation Instructions

## Prerequisites

- Linux/Unix system with Asterisk installed
- User account with permissions to execute asterisk CLI commands
- Writable directory for log files (usually `/var/log/allstar` or similar)
- Basic command-line familiarity

## Quick Installation

### 1. Copy Script to Binary Directory

Choose one of these locations:
```bash
# Option A: User local directory (recommended for single user)
cp autoconnect ~/.local/bin/
chmod 755 ~/.local/bin/autoconnect

# Option B: System directory (requires root or sudo)
sudo cp autoconnect /usr/local/bin/
sudo chmod 755 /usr/local/bin/autoconnect

# Option C: Asterisk directory (if asterisk has its own bin)
cp autoconnect /usr/sbin/
chmod 755 /usr/sbin/autoconnect
```

### 2. Set File Ownership and Permissions

```bash
# If installing to system directory (requires root)
sudo chown root:root /usr/local/bin/autoconnect
sudo chmod 755 /usr/local/bin/autoconnect

# If installing to user directory
chown $(whoami):$(whoami) ~/.local/bin/autoconnect
chmod 755 ~/.local/bin/autoconnect
```

### 3. Create Log Directory

```bash
# Create log directory (replace with your chosen path)
mkdir -p /var/log/allstar

# Set permissions (adjust user as needed)
sudo chown asterisk:asterisk /var/log/allstar
sudo chmod 755 /var/log/allstar
```

Note: The script will auto-create this directory if it doesn't exist (must have parent directory permissions).

### 4. Configure the Script (Choose One Method)

#### Method A: External Config File (Recommended)

Copy the example config:
```bash
cp autoconnect.conf.example ~/.autoconnect.conf
```

Edit the config file with your values:
```bash
nano ~/.autoconnect.conf
```

Set these values:
```ini
ORIGIN_NODE_ID=12345      # Your AllStar node ID
DEST_NODE_ID=54321        # Destination node to connect to
LOG_DIR=/var/log/allstar  # Log directory (will be created if missing)
```

Benefits:
- Easy to update without editing script
- Can use different configs per node
- Can distribute single script with multiple configs

#### Method B: Environment Variables

Export variables before running:
```bash
export ORIGIN_NODE_ID=12345
export DEST_NODE_ID=54321
export LOG_DIR=/var/log/allstar
./autoconnect
```

#### Method C: Edit Script Directly

Edit the script at lines 24-30:
```bash
nano /usr/local/bin/autoconnect
```

Change:
```bash
LOG_DIR="/var/log/allstar"
ORIGIN_NODE_ID="12345"
DEST_NODE_ID="54321"
```

### 5. Test the Script

Run manually to verify configuration:
```bash
# If using config file
~/.autoconnect.conf /usr/local/bin/autoconnect

# If using environment variables
export ORIGIN_NODE_ID=12345
export DEST_NODE_ID=54321
export LOG_DIR=/var/log/allstar
/usr/local/bin/autoconnect

# If using script variables
/usr/local/bin/autoconnect
```

Expected output:
- No errors displayed
- Exit code 0 if connected, 1 if disconnected after retries
- Log file created and populated: `cat /var/log/allstar/autoconnect.log`

### 6. Setup Cron Job

Create or edit crontab for the appropriate user:
```bash
# Edit cron for current user
crontab -e

# Or edit for specific user (requires root)
sudo crontab -e -u asterisk
```

Add entry for every 15 minutes:
```bash
# Basic setup
*/15 * * * * /usr/local/bin/autoconnect

# With error output captured
*/15 * * * * /usr/local/bin/autoconnect >> /var/log/allstar/cron.log 2>&1

# With email notifications
*/15 * * * * ENABLE_EMAIL=true EMAIL_ADDRESS=admin@example.com /usr/local/bin/autoconnect

# With custom log directory
*/15 * * * * LOG_DIR=/var/log/allstar /usr/local/bin/autoconnect
```

Notes:
- Frequency: Every 15 minutes is recommended (adjust as needed)
- User: Should be asterisk user or user with CLI access (NOT root unless necessary)
- Output: Use `> /dev/null 2>&1` if you prefer no output, or log to file to troubleshoot

### 7. Verify Installation

Check that script is installed:
```bash
which autoconnect
ls -l /usr/local/bin/autoconnect
```

Check that log directory exists:
```bash
ls -ld /var/log/allstar
```

Check log file is created:
```bash
ls -l /var/log/allstar/autoconnect.log
tail /var/log/allstar/autoconnect.log
```

Check cron job is scheduled:
```bash
crontab -l
```

## Optional Configuration

### Log Rotation

Add to `/etc/logrotate.d/autoconnect`:
```
/var/log/allstar/autoconnect*.log {
    weekly
    rotate 4
    compress
    delaycompress
    missingok
    notifempty
    create 0640 asterisk asterisk
}
```

Then enable:
```bash
sudo logrotate -f /etc/logrotate.d/autoconnect
```

### Email Notifications

Enable email alerts for persistent failures:

Option 1: In crontab
```bash
*/15 * * * * ENABLE_EMAIL=true EMAIL_ADDRESS=admin@example.com /usr/local/bin/autoconnect
```

Option 2: In config file
```ini
ENABLE_EMAIL=true
EMAIL_ADDRESS=admin@example.com
```

Option 3: In script
```bash
ENABLE_EMAIL="true"
EMAIL_ADDRESS="admin@example.com"
```

Requires `mail` command installed:
```bash
sudo apt-get install mailutils  # Debian/Ubuntu
sudo yum install mailx          # RHEL/CentOS
```

### Syslog Monitoring

View logs in system syslog:
```bash
# All autoconnect entries
grep autoconnect /var/log/syslog

# Real-time monitoring
tail -f /var/log/syslog | grep autoconnect

# systemd systems
journalctl -t autoconnect -f
```

## Troubleshooting Installation

### Problem: "Permission denied" when running script
**Solution:** Check file permissions
```bash
ls -l /usr/local/bin/autoconnect
chmod 755 /usr/local/bin/autoconnect
```

### Problem: "ERROR: LOG_DIR is not set"
**Solution:** Configure LOG_DIR via config file or environment
```bash
# Check config file
cat ~/.autoconnect.conf
# Or set environment variable
export LOG_DIR=/var/log/allstar
```

### Problem: "ERROR: Cannot create log directory"
**Solution:** Check parent directory permissions
```bash
ls -ld /var/log
# Parent must be writable
sudo chmod 755 /var/log
```

### Problem: Cron job not running
**Solution:** Check cron log
```bash
# View cron logs
sudo tail /var/log/cron       # RHEL/CentOS
sudo tail /var/log/syslog     # Debian/Ubuntu
grep CRON /var/log/syslog     # systemd systems

# Verify cron daemon is running
sudo systemctl status cron
```

### Problem: "ERROR: Asterisk binary not found"
**Solution:** Verify asterisk installation and path
```bash
which asterisk
/usr/bin/asterisk -rx "show version"

# If at different path, override in config or script
export ASTERISK_BIN=/usr/bin/asterisk
```

### Problem: "Another instance is already running"
**Solution:** Check for stale lock file
```bash
# List running instances
ps aux | grep autoconnect

# Check lock file
cat /var/log/allstar/autoconnect.lock

# Remove stale lock if no process running
rm /var/log/allstar/autoconnect.lock
```

## Upgrade from v1.x

Installation stays the same - just replace the script:

```bash
# Backup old script
cp /usr/local/bin/autoconnect /usr/local/bin/autoconnect.v1.bak

# Install new version
cp autoconnect /usr/local/bin/autoconnect
chmod 755 /usr/local/bin/autoconnect

# First run will create new log files
/usr/local/bin/autoconnect

# Verify it worked
tail /var/log/allstar/autoconnect.log
```

No configuration changes needed - fully backward compatible.

See [CHANGELOG.md](CHANGELOG.md) for what's new in v2.0.

## Next Steps

1. Read [README.md](README.md) for feature documentation
2. Check logs: `tail -f /var/log/allstar/autoconnect.log`
3. Monitor connections: `watch -n 5 grep CONNECTED /var/log/allstar/autoconnect.log`
4. Setup email alerts if needed
5. Configure log rotation (optional)

## Verification Checklist

- [ ] Script installed in `/usr/local/bin/` (or chosen directory)
- [ ] Script has executable permissions (755)
- [ ] Log directory exists: `/var/log/allstar/`
- [ ] Log directory is writable by executing user
- [ ] Configuration set (config file, env vars, or script)
- [ ] `ORIGIN_NODE_ID` configured
- [ ] `DEST_NODE_ID` configured
- [ ] `LOG_DIR` configured
- [ ] Script runs manually without errors
- [ ] Cron job added to crontab
- [ ] Log file created after first run
- [ ] Can view logs: `cat /var/log/allstar/autoconnect.log`

## Support

For issues or questions, refer to [README.md](README.md) troubleshooting section or project documentation.
