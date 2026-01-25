# AllStar Node Connection Check Script

## Overview

**autoconnect** is a robust monitoring and recovery script for maintaining connections between AllStar nodes (amateur radio VoIP system).

### Purpose

- **Monitor** if your local AllStar node (Origin Node) is connected to a remote node (Destination Node)
- **Reconnect** automatically with retry logic (up to 3 attempts by default)
- **Verify** that reconnection succeeded before marking as successful
- **Log** all connection attempts with timestamps and detailed status information
- **Alert** on persistent failures via syslog or email

### Features

- **3-Attempt Retry System** - Automatically retries up to 3 times with state persistence
- **Reconnection Verification** - Verifies reconnection actually succeeded, not just issued
- **Timeout Protection** - Prevents hanging on asterisk socket issues (default 10 seconds)
- **Structured Logging** - Machine-parsable logs with severity levels and metadata
- **Lock File Protection** - Prevents concurrent execution
- **Auto-File Creation** - Creates log directories and files automatically
- **Configuration Management** - External config file support with environment variable overrides
- **Input Validation** - Clear error messages for configuration issues
- **Notification System** - Syslog (default) and optional email alerts
- **Backward Compatible** - Works with existing v1.x installations without changes

## Quick Start

### Prerequisites

- Linux/Unix system with Asterisk installed
- User account with permissions to run asterisk CLI commands
- Writable directory for log files

### Configuration

The script needs these settings configured:

| Variable | Description | Example |
|----------|-------------|---------|
| `LOG_DIR` | Directory for logs and state files | `/var/log/allstar` |
| `ORIGIN_NODE_ID` | Your local AllStar node ID | `12345` |
| `DEST_NODE_ID` | Remote node to connect to | `54321` |

### Configuration Methods (in priority order)

1. **Environment Variables** (highest priority)
   ```bash
   export LOG_DIR=/var/log/allstar
   export ORIGIN_NODE_ID=12345
   export DEST_NODE_ID=54321
   ./autoconnect
   ```

2. **External Config File** (recommended for cron)
   - User config: `~/.autoconnect.conf`
   - System config: `/etc/autoconnect.conf`
   - Copy and edit `autoconnect.conf.example` to either location

3. **Script Variables** (edit at top of script, not recommended)
   - Edit `LOG_DIR`, `ORIGIN_NODE_ID`, `DEST_NODE_ID` at lines 24-30

### Running the Script

**Manual execution:**
```bash
./autoconnect
```

**Via cron (recommended, every 15 minutes):**
```bash
*/15 * * * * asterisk /usr/local/bin/autoconnect > /dev/null 2>&1
```

## Configuration Options

All configuration variables support environment variable overrides:

### Required
- `LOG_DIR` - Directory for logs and state files (auto-created)
- `ORIGIN_NODE_ID` - Your local AllStar node ID (numeric)
- `DEST_NODE_ID` - Remote node to connect to (numeric)

### Optional
- `ASTERISK_BIN` - Path to asterisk binary (default: `/usr/sbin/asterisk`)
- `ASTERISK_TIMEOUT` - Command timeout in seconds (default: `10`)
- `VERIFY_DELAY` - Wait time after reconnect before verifying (default: `5` seconds)
- `MAX_RETRIES` - Maximum reconnection attempts (default: `3`)
- `ENABLE_SYSLOG` - Send to syslog (default: `true`)
- `ENABLE_EMAIL` - Send email on failure (default: `false`)
- `EMAIL_ADDRESS` - Email for notifications (if enabled)

See `autoconnect.conf.example` for external config file format.

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | Nodes connected or reconnected successfully |
| 1 | Persistent failure | Max retries exceeded, will try again next run |
| 2 | Configuration error | Check configuration and error message |
| 3 | Already running | Another instance is active (will retry next run) |
| 4 | Asterisk error | Cannot communicate with asterisk socket |

## Logging

### Log Files

The script creates these log files in `LOG_DIR`:

| File | Purpose |
|------|---------|
| `autoconnect.log` | Main activity log, structured format |
| `autoconnect_summary.log` | Daily summaries (framework for future use) |
| `cron_check.txt` | Legacy format (for backward compatibility) |
| `retry_state.txt` | Current retry state (managed by script) |
| `autoconnect.lock` | Lock file during execution |
| `PERSISTENT_FAILURE` | Created when max retries exceeded |

### Log Format

Structured format for easy parsing:
```
[2024-01-24T15:30:00-06:00] [INFO] Nodes are connected | origin=12345, dest=54321
[2024-01-24T15:45:00-06:00] [WARNING] Nodes are disconnected | origin=12345, dest=54321
[2024-01-24T15:45:00-06:00] [INFO] Attempting reconnection | attempt=1/3
[2024-01-24T15:45:05-06:00] [SUCCESS] Reconnection verified | attempt=1/3
```

Log levels: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `SUCCESS`

### Viewing Logs

```bash
# Real-time monitoring
tail -f /var/log/allstar/autoconnect.log

# Last 50 lines
tail -50 /var/log/allstar/autoconnect.log

# Filter by level
grep "\[ERROR\]" /var/log/allstar/autoconnect.log

# View retry state
cat /var/log/allstar/retry_state.txt
```

## Retry System

### How It Works

1. **Check Connection** - Script checks if nodes are connected
2. **If Connected** - Logs success, resets retry state, exits with code 0
3. **If Disconnected** - Checks retry state:
   - Attempts < 3: Increment counter, attempt reconnection, verify, exit
   - Attempts ≥ 3: Log failure, create alert flag, exit with code 1
4. **Next Cron Run** - Script runs again, sees disconnect, tries next attempt

### Retry State File

Script maintains `retry_state.txt` with format: `timestamp:attempt_count:status`

Example:
```
1706137200:2:RECONNECTING
```

This file is:
- Created on first disconnection
- Updated with each retry attempt
- Reset to zero on successful connection
- Automatically cleaned up

## Troubleshooting

### Script won't start: "ERROR: LOG_DIR is not set"
**Solution:** Set LOG_DIR environment variable or edit script config section
```bash
export LOG_DIR=/var/log/allstar
./autoconnect
```

### "ERROR: ORIGIN_NODE_ID or DEST_NODE_ID is not set"
**Solution:** Configure both node IDs
```bash
export ORIGIN_NODE_ID=12345
export DEST_NODE_ID=54321
./autoconnect
```

### "ERROR: Another instance is already running"
**Solution:** Previous instance is still running or stale lock exists
```bash
# Check for running instance
ps aux | grep autoconnect

# Remove stale lock if needed
rm -f /var/log/allstar/autoconnect.lock
```

### "ERROR: Asterisk binary not found"
**Solution:** Asterisk not installed or at different path
```bash
# Find asterisk
which asterisk
/usr/bin/asterisk -rx "show version"

# Override path if needed
export ASTERISK_BIN=/usr/bin/asterisk
./autoconnect
```

### Reconnection attempts not working
**Solution:** Check asterisk connectivity
```bash
# Test asterisk commands
asterisk -rx "rpt nodes 12345"
asterisk -rx "rpt cmd 12345 ilink 3 54321"
```

## Cron Job Setup

Recommended cron configuration:

```bash
# Every 15 minutes as asterisk user
*/15 * * * * asterisk /usr/local/bin/autoconnect > /dev/null 2>&1

# Or with logging enabled
*/15 * * * * asterisk /usr/local/bin/autoconnect

# With email on failure
*/15 * * * * asterisk EMAIL_ADDRESS=admin@example.com ENABLE_EMAIL=true /usr/local/bin/autoconnect
```

Note: Ensure cron output is being redirected appropriately or errors won't be visible.

## Notifications

### Syslog (Recommended)
Enabled by default. View with:
```bash
# View autoconnect logs in syslog
grep autoconnect /var/log/syslog
journalctl -t autoconnect  # systemd systems
```

### Email Alerts (Optional)
When persistent failure occurs:
```bash
ENABLE_EMAIL=true EMAIL_ADDRESS=admin@example.com ./autoconnect
```

### Failure Flag
A flag file (`PERSISTENT_FAILURE`) is created when max retries exceeded:
```bash
# External monitoring can watch for this file
test -f /var/log/allstar/PERSISTENT_FAILURE && echo "Node down!"
```

## Performance

- **Normal operation**: ~2 seconds (check connection)
- **Reconnection**: ~7 seconds (includes 5-second verification wait)
- **Concurrent run blocked**: <1 second
- **Impact on cron**: Negligible for 15-minute intervals

## Upgrading from v1.x

✓ **Fully backward compatible** - Simply replace the script file

Old config still works:
```bash
LOG_DIR="/path/to/logs"
ORIGIN_NODE_ID="12345"
DEST_NODE_ID="54321"
```

New features available but optional:
- External config file for easier management
- Structured logging (legacy format still supported)
- Email notifications
- Configuration validation

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

## Security Considerations

### Permissions
- Script should NOT run as root unless absolutely necessary
- Use dedicated asterisk user with minimal required permissions
- Log directory should be readable by monitoring user

### Lock File
- Prevents concurrent execution (security benefit)
- Automatically cleaned up on exit
- Stale locks detected and removed

### Input Validation
- All configuration values validated
- Node IDs must be numeric
- Paths checked for validity
- Clear error messages for invalid input

## Integration Examples

### Monitor with Nagios/Icinga
```bash
#!/bin/bash
# Check if PERSISTENT_FAILURE exists
if [ -f /var/log/allstar/PERSISTENT_FAILURE ]; then
  echo "CRITICAL: AllStar node disconnected"
  exit 2
else
  echo "OK: AllStar node connected"
  exit 0
fi
```

### Email on Failure
```bash
ENABLE_EMAIL=true EMAIL_ADDRESS=alerts@example.com ./autoconnect
```

### Custom Alerting
```bash
# Check exit code
./autoconnect
if [ $? -eq 1 ]; then
  # Max retries exceeded
  echo "Alert: Node failed to reconnect" | mail -s "AllStar Alert" admin@example.com
fi
```

## Support & Contributing

For issues, questions, or contributions, please refer to the project repository.

See [INSTALL.md](INSTALL.md) for detailed installation instructions.
See [CHANGELOG.md](CHANGELOG.md) for version history and upgrade notes.
