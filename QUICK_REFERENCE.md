# Quick Reference Guide

## Getting Started - 30 Seconds

```bash
# 1. Copy script
sudo cp autoconnect /usr/local/bin/

# 2. Configure (set your node IDs)
export ORIGIN_NODE_ID=12345
export DEST_NODE_ID=54321
export LOG_DIR=/var/log/allstar

# 3. Test
/usr/local/bin/autoconnect

# 4. Add to cron
crontab -e
# Add: */15 * * * * /usr/local/bin/autoconnect
```

---

## Common Commands

### Check Status
```bash
# View recent logs
tail -f /var/log/allstar/autoconnect.log

# Check current retry state
cat /var/log/allstar/retry_state.txt

# Check for failures
cat /var/log/allstar/PERSISTENT_FAILURE
```

### Configuration
```bash
# Create config file
cp autoconnect.conf.example ~/.autoconnect.conf
nano ~/.autoconnect.conf  # Edit with your values

# Set environment variables
export ORIGIN_NODE_ID=12345
export DEST_NODE_ID=54321
export LOG_DIR=/var/log/allstar

# Run with specific config
AUTOCONNECT_CONFIG=/etc/autoconnect.conf /usr/local/bin/autoconnect
```

### Testing
```bash
# Run manually
/usr/local/bin/autoconnect

# Run with custom path
ASTERISK_BIN=/usr/bin/asterisk /usr/local/bin/autoconnect

# Run with logging
LOG_DIR=/tmp/test /usr/local/bin/autoconnect

# Check if it's in cron
crontab -l
```

### Troubleshooting
```bash
# View all errors
grep ERROR /var/log/allstar/autoconnect.log

# View warnings
grep WARNING /var/log/allstar/autoconnect.log

# Check for lock file
cat /var/log/allstar/autoconnect.lock

# Remove stale lock
rm -f /var/log/allstar/autoconnect.lock

# Check permissions
ls -ld /var/log/allstar
ls -l /usr/local/bin/autoconnect

# Check cron logs
grep autoconnect /var/log/syslog
journalctl -t autoconnect  # For systemd
```

---

## Configuration Options

| Option | Default | Purpose |
|--------|---------|---------|
| `LOG_DIR` | Required | Log file directory |
| `ORIGIN_NODE_ID` | Required | Your node ID |
| `DEST_NODE_ID` | Required | Destination node ID |
| `ASTERISK_BIN` | `/usr/sbin/asterisk` | Asterisk path |
| `ASTERISK_TIMEOUT` | `10` | Command timeout (seconds) |
| `VERIFY_DELAY` | `5` | Wait before verify (seconds) |
| `MAX_RETRIES` | `3` | Retry attempts |
| `ENABLE_SYSLOG` | `true` | Send to syslog |
| `ENABLE_EMAIL` | `false` | Send email alerts |
| `EMAIL_ADDRESS` | Empty | Email for alerts |

---

## Log Files

| File | Purpose |
|------|---------|
| `autoconnect.log` | Main activity log |
| `retry_state.txt` | Current retry count |
| `PERSISTENT_FAILURE` | Alert flag (max retries exceeded) |
| `autoconnect.lock` | Concurrent run prevention |
| `cron_check.txt` | Legacy format (backward compat) |

---

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Connected/Reconnected | Success - check logs |
| 1 | Persistent failure | Max retries exceeded - fix issue |
| 2 | Configuration error | Fix config and try again |
| 3 | Already running | Check for running process |
| 4 | Asterisk error | Check Asterisk logs |

---

## Cron Examples

```bash
# Every 15 minutes (recommended)
*/15 * * * * /usr/local/bin/autoconnect

# Every 5 minutes (aggressive)
*/5 * * * * /usr/local/bin/autoconnect

# Every hour
0 * * * * /usr/local/bin/autoconnect

# Every hour with logging
0 * * * * /usr/local/bin/autoconnect >> /var/log/allstar/cron.log 2>&1

# With email alerts
*/15 * * * * ENABLE_EMAIL=true EMAIL_ADDRESS=me@example.com /usr/local/bin/autoconnect

# With custom log directory
*/15 * * * * LOG_DIR=/var/log/custom /usr/local/bin/autoconnect
```

---

## Monitoring Integration

### Check in Nagios/Icinga
```bash
if [ -f /var/log/allstar/PERSISTENT_FAILURE ]; then
  echo "CRITICAL: AllStar node disconnected"
  exit 2
fi
echo "OK: AllStar node connected"
exit 0
```

### Setup Email Alerts
```bash
# In crontab
*/15 * * * * ENABLE_EMAIL=true EMAIL_ADDRESS=admin@example.com /usr/local/bin/autoconnect

# Or in config file
ENABLE_EMAIL=true
EMAIL_ADDRESS=admin@example.com
```

### View in Syslog
```bash
# All entries
grep autoconnect /var/log/syslog

# Errors only
grep autoconnect /var/log/syslog | grep ERROR

# Real-time
tail -f /var/log/syslog | grep autoconnect

# systemd systems
journalctl -t autoconnect
journalctl -t autoconnect -f  # Real-time
```

---

## Installation Checklist

- [ ] Copy script to `/usr/local/bin/`
- [ ] Make executable: `chmod 755 /usr/local/bin/autoconnect`
- [ ] Create log directory: `mkdir -p /var/log/allstar`
- [ ] Set log permissions: `chmod 755 /var/log/allstar`
- [ ] Create config file: `cp autoconnect.conf.example ~/.autoconnect.conf`
- [ ] Edit config with your node IDs
- [ ] Test script manually
- [ ] Verify logs created
- [ ] Add to crontab
- [ ] Verify cron is running the script

---

## Troubleshooting Flowchart

```
Script won't start?
├─ Check configuration: export ORIGIN_NODE_ID=12345
├─ Check LOG_DIR: mkdir -p /var/log/allstar
└─ Check permissions: chmod 755 /var/log/allstar

No logs being created?
├─ Run manually: /usr/local/bin/autoconnect
├─ Check permissions: ls -l /var/log/allstar/autoconnect.log
└─ Check Asterisk: asterisk -rx "show version"

Cron not running?
├─ Check cron job: crontab -l
├─ Check cron logs: grep CRON /var/log/syslog
└─ Check syntax: 0 * * * * /usr/local/bin/autoconnect

Still having issues?
├─ Check error logs: grep ERROR /var/log/allstar/autoconnect.log
├─ Check Asterisk: asterisk -rx "rpt nodes 12345"
└─ Review README.md troubleshooting section
```

---

## Performance Tips

### Reduce Log Growth
```bash
# Setup log rotation
sudo nano /etc/logrotate.d/autoconnect

# Add:
/var/log/allstar/autoconnect*.log {
    weekly
    rotate 4
    compress
}

# Test it
sudo logrotate -f /etc/logrotate.d/autoconnect
```

### Optimize Frequency
```bash
# Current: Every 15 minutes (default)
*/15 * * * * /usr/local/bin/autoconnect

# More aggressive (every 5 minutes)
*/5 * * * * /usr/local/bin/autoconnect

# Less aggressive (every hour)
0 * * * * /usr/local/bin/autoconnect
```

### Monitor Resources
```bash
# Script runtime (should be < 10 seconds)
{ time /usr/local/bin/autoconnect; } 2>&1

# Log size
du -h /var/log/allstar/

# Memory usage (while running)
top -p $(pidof autoconnect)
```

---

## Documentation Quick Links

| Need | File | Time |
|------|------|------|
| Quick overview | PROJECT.md | 5 min |
| Install it | INSTALL.md | 15 min |
| Use it | README.md | 20 min |
| Understand it | TECHNICAL_OVERVIEW.md | 30 min |
| Fix a problem | README.md#troubleshooting | 10 min |
| Configure it | autoconnect.conf.example | 5 min |
| Upgrade | CHANGELOG.md | 10 min |

---

## Essential Configuration

### Minimum Config File
```ini
# Create as ~/.autoconnect.conf
[nodes]
origin_node=12345        # YOUR NODE ID
dest_node=54321          # DESTINATION NODE ID

[paths]
log_dir=/var/log/allstar # LOG DIRECTORY
```

### Recommended Config File
```ini
[nodes]
origin_node=12345
dest_node=54321

[paths]
log_dir=/var/log/allstar
asterisk_bin=/usr/sbin/asterisk

[behavior]
max_retries=3
asterisk_timeout=10
verify_delay=5

[notifications]
enable_syslog=true
enable_email=false
```

---

## Email Notification Setup

### Prerequisites
```bash
# Install mail
sudo apt-get install mailutils  # Debian/Ubuntu
sudo yum install mailx          # RHEL/CentOS
```

### Enable in Config
```ini
ENABLE_EMAIL=true
EMAIL_ADDRESS=admin@example.com
```

### Or via Cron
```bash
*/15 * * * * ENABLE_EMAIL=true EMAIL_ADDRESS=admin@example.com /usr/local/bin/autoconnect
```

### Custom Email Script
```bash
if [ -f /var/log/allstar/PERSISTENT_FAILURE ]; then
  mail -s "AllStar Node Alert" admin@example.com < /var/log/allstar/PERSISTENT_FAILURE
fi
```

---

## Version Information

**Current Version:** 2.0.0

**Release Date:** 2024-01-24

**Features:**
- ✅ 3-attempt retry system
- ✅ Reconnection verification
- ✅ Structured logging
- ✅ Email alerts
- ✅ Configuration files
- ✅ Input validation

**Compatibility:** Asterisk 11+, AllStar nodes

---

## Common Issues & Quick Fixes

### "ERROR: LOG_DIR is not set"
**Fix:** `export LOG_DIR=/var/log/allstar`

### "ERROR: ORIGIN_NODE_ID is not set"
**Fix:** Set it before running: `export ORIGIN_NODE_ID=12345`

### "Permission denied"
**Fix:** `chmod 755 /usr/local/bin/autoconnect`

### "Another instance is already running"
**Fix:** `rm -f /var/log/allstar/autoconnect.lock`

### "Asterisk binary not found"
**Fix:** `export ASTERISK_BIN=/usr/bin/asterisk`

### "Cannot create log directory"
**Fix:** `sudo mkdir -p /var/log/allstar && sudo chmod 755 /var/log/allstar`

---

## Useful One-Liners

```bash
# Tail logs with timestamps highlighted
tail -f /var/log/allstar/autoconnect.log | sed 's/ERROR/\x1b[31mERROR\x1b[0m/g'

# Count disconnections per day
grep "DISCONNECTED" /var/log/allstar/autoconnect.log | cut -d'T' -f1 | uniq -c

# Find slow commands
grep -E 'TIMEOUT|exceeded' /var/log/allstar/autoconnect.log

# Get retry statistics
grep 'Attempting reconnection' /var/log/allstar/autoconnect.log | wc -l

# Check current retry state
cat /var/log/allstar/retry_state.txt 2>/dev/null || echo "Not retrying"

# Monitor in real-time with colors
watch -c 'tail -20 /var/log/allstar/autoconnect.log | sed "s/ERROR/\x1b[31mERROR\x1b[0m/"'
```

---

## File Locations Summary

| Item | Location |
|------|----------|
| Script | `/usr/local/bin/autoconnect` |
| Logs | `/var/log/allstar/autoconnect.log` |
| Config | `~/.autoconnect.conf` or `/etc/autoconnect.conf` |
| Retry State | `/var/log/allstar/retry_state.txt` |
| Failure Flag | `/var/log/allstar/PERSISTENT_FAILURE` |
| Lock File | `/var/log/allstar/autoconnect.lock` |

---

## Key Concepts

**Retry State:** How many times the script has tried to reconnect
- Resets to 0 when connection is successful
- Increments each time connection fails
- After 3 attempts, gives up and alerts

**Reconnection Verification:** Confirms reconnect actually worked
- Issues command
- Waits 5 seconds
- Checks if connected
- Only counts as success if verified

**Lock File:** Prevents two instances running simultaneously
- Created when script starts
- Deleted when script exits
- Stale locks automatically removed

---

**Last Updated:** 2024-01-24
**Status:** Current & Tested
