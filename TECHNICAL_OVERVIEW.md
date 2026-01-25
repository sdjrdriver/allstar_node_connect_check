# AllStar Auto-Connect Script - Technical Overview

## Executive Summary

The AllStar Auto-Connect script is an automated monitoring and recovery solution for maintaining reliable connections between amateur radio nodes in the AllStar network. It continuously monitors connection status and automatically attempts to restore connections when they drop, with built-in safeguards to prevent aggressive reconnection attempts.

**Target Users:** Amateur radio operators, network administrators, and system operators managing AllStar VoIP nodes.

---

## What This Script Does (Non-Technical)

### The Problem

When two AllStar radio nodes (like remote repeaters or linking systems) are supposed to be connected to each other, the connection can sometimes drop unexpectedly due to:
- Network issues
- Asterisk software glitches
- System reboots
- Configuration changes

When this happens, someone needs to manually reconnect them, which can take hours to notice and fix.

### The Solution

This script acts like an automated watchdog that:
1. **Checks regularly** - Runs every 15 minutes to verify the connection is still active
2. **Alerts immediately** - Logs what it finds each time
3. **Fixes automatically** - If disconnected, attempts to reconnect without manual intervention
4. **Stops before causing problems** - Gives up after 3 failed attempts to prevent network flooding
5. **Reports back** - Creates alerts and logs for monitoring systems

### Business Value

✅ **Higher Uptime** - Connections are restored within 15 minutes instead of hours
✅ **Less Manual Work** - No need for someone to manually reconnect nodes
✅ **Better Visibility** - Detailed logs show what happened and when
✅ **Prevents Spam** - Smart retry logic prevents the system from getting stuck trying to reconnect
✅ **Easy Integration** - Works with existing monitoring systems (Nagios, Zabbix, etc.)

---

## How It Works (Step-by-Step)

```
┌─────────────────────────────────────────────────────────────┐
│ Every 15 minutes, the script wakes up and:                 │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
           ┌──────────────────────────┐
           │  Step 1: Check Connection │
           │ (Query Asterisk system)   │
           └──────────────────────────┘
                          │
                ┌─────────┴─────────┐
                │                   │
                ▼                   ▼
        ┌────────────────┐  ┌────────────────┐
        │  Connected? ✓  │  │ Disconnected? ✗ │
        └────────────────┘  └────────────────┘
                │                   │
                │                   ▼
                │          ┌──────────────────┐
                │          │ Check Retry Count │
                │          └──────────────────┘
                │                   │
                │        ┌──────────┴──────────┐
                │        │                     │
                │        ▼                     ▼
                │  Attempts < 3?        Attempts ≥ 3?
                │        │                     │
                │        ▼                     ▼
                │   ┌─────────────────┐  ┌──────────────┐
                │   │ Attempt Reconnect│  │ Give Up      │
                │   │ + Verify Success  │  │ Alert Admin  │
                │   └─────────────────┘  └──────────────┘
                │        │
                ▼        ▼
            ┌─────────────────┐
            │ Log Status      │
            │ Clean Up Files  │
            │ Exit Gracefully │
            └─────────────────┘
```

### Connection States

The script tracks four possible states:

| State | Meaning | What Happens |
|-------|---------|--------------|
| **Connected** | Nodes are linked together | Script logs success, resets counters, exits happy |
| **Disconnected (1st time)** | Nodes lost connection | Script attempts first reconnection and waits |
| **Disconnected (2-3 times)** | Still disconnected after retries | Script attempts again, counting down attempts |
| **Failed** | Cannot reconnect after 3 attempts | Script alerts operators and stops trying |

### Key Features Explained

#### 1. **Automatic Reconnection with Verification**

Instead of just issuing a reconnect command and hoping it works, the script:
1. Issues the reconnect command
2. Waits 5 seconds for the connection to establish
3. Checks if it actually worked
4. Only marks as successful if verification confirms it

This prevents false "successes" where the command ran but didn't actually connect the nodes.

#### 2. **Smart Retry System**

The script doesn't spam reconnection attempts. Instead:
- **Attempt 1-3:** Tries to reconnect (happens once per 15 minutes)
- **After Attempt 3:** Stops trying and alerts operators

Without this, the system could hammer the network with constant reconnection attempts.

**Example Timeline:**
```
3:00 PM  - Connection drops. Script detects it. Attempt 1/3. Fails.
3:15 PM  - Still disconnected. Attempt 2/3. Fails.
3:30 PM  - Still disconnected. Attempt 3/3. Fails.
3:45 PM  - Still disconnected. Gives up. Alerts admin.
4:00 PM  - Won't try again until manually reset or admin intervenes.
```

#### 3. **Timeout Protection**

If the Asterisk system becomes unresponsive, the script doesn't hang forever. It waits 10 seconds, then gives up on that attempt.

This prevents the script from consuming system resources indefinitely.

#### 4. **Detailed Logging**

Every action is logged with:
- **Timestamp** - Exactly when it happened
- **Status** - What was attempted (check, reconnect, success, failure)
- **Details** - Which nodes, which attempt number, any errors

Example log entry:
```
[2024-01-24T15:30:00-06:00] [SUCCESS] Reconnection verified | attempt=1/3
```

This makes it easy to see what happened and when, useful for troubleshooting.

#### 5. **Prevents Simultaneous Execution**

If one script instance is still running, the next one (triggered by cron) won't start. This prevents conflicts.

---

## File Structure

```
allstar_node_connect_check/
├── autoconnect              ← Main script (executable)
├── README.md                ← User-friendly guide
├── INSTALL.md               ← Installation instructions
├── CHANGELOG.md             ← What's new / version history
├── TECHNICAL_OVERVIEW.md    ← This document
├── autoconnect.conf.example ← Template for configuration
└── .gitignore              ← (Recommended) Prevents committing logs
```

---

## Installation Overview

### For System Administrators

**The short version:**

1. Copy the script to `/usr/local/bin/`
2. Make it executable (`chmod 755`)
3. Configure with node IDs and log directory
4. Add to crontab to run every 15 minutes
5. Verify it's working by checking logs

**Estimated time:** 10-15 minutes

See [INSTALL.md](INSTALL.md) for detailed step-by-step instructions.

### System Requirements

- Linux/Unix-based system (Ubuntu, CentOS, Debian, etc.)
- Asterisk PBX software installed and running
- User account with permission to run Asterisk CLI commands
- Writable directory for log files (typically `/var/log/allstar/`)
- Cron scheduler (standard on all Linux systems)

---

## Configuration

### Required Settings

You must set three pieces of information:

| Setting | What It Is | Example |
|---------|-----------|---------|
| **ORIGIN_NODE_ID** | Your local AllStar node number | 12345 |
| **DEST_NODE_ID** | The node you want to connect to | 54321 |
| **LOG_DIR** | Where to store logs | /var/log/allstar |

### Configuration Methods (Pick One)

**Option 1: External Config File** (Recommended)
- Create a file with your settings
- Script reads it automatically
- Easy to update without editing the script

**Option 2: Environment Variables**
- Set before running the script
- Good for temporary overrides
- Doesn't persist between runs

**Option 3: Edit the Script**
- Modify the script file directly
- Not recommended (harder to update)
- Works but less flexible

---

## Logging & Monitoring

### Log Files Created

The script creates these files in your log directory:

| File | Purpose |
|------|---------|
| `autoconnect.log` | Main activity log (what you usually read) |
| `autoconnect.log` | Contains all status updates and errors |
| `retry_state.txt` | Tracks current retry attempts |
| `PERSISTENT_FAILURE` | Created if max retries exceeded (for alerts) |
| `autoconnect.lock` | Temporary file (prevents running twice at once) |

### Reading the Logs

**View recent activity:**
```bash
tail -20 /var/log/allstar/autoconnect.log
```

**Follow logs in real-time:**
```bash
tail -f /var/log/allstar/autoconnect.log
```

**Search for problems:**
```bash
grep ERROR /var/log/allstar/autoconnect.log
```

### Log Format Example

```
[2024-01-24T15:30:00-06:00] [INFO] Nodes are connected | origin=12345, dest=54321
[2024-01-24T15:45:00-06:00] [WARNING] Nodes are disconnected | origin=12345, dest=54321
[2024-01-24T15:45:05-06:00] [SUCCESS] Reconnection verified | attempt=1/3
```

**What each part means:**
- `[2024-01-24T15:45:00-06:00]` = Timestamp (date and time)
- `[SUCCESS]` = Log level (INFO, WARNING, ERROR, SUCCESS)
- `Reconnection verified` = What happened
- `attempt=1/3` = Which attempt this was

---

## Integration with Monitoring Systems

### For System Administrators: Monitoring Integration

The script creates files that external monitoring systems can watch:

**Failure Detection:**
- If file `PERSISTENT_FAILURE` exists → Node failed to reconnect
- If exit code is 1 → Max retries exceeded

**Examples:**

**Nagios/Icinga Check:**
```bash
# Alert if persistent failure flag exists
if [ -f /var/log/allstar/PERSISTENT_FAILURE ]; then
  echo "CRITICAL: AllStar node disconnected"
  exit 2
fi
echo "OK: AllStar node connected"
exit 0
```

**Email on Failure:**
```bash
# Check exit code from last run
if [ -f /var/log/allstar/PERSISTENT_FAILURE ]; then
  mail -s "Alert: Node Down" admin@example.com < /var/log/allstar/PERSISTENT_FAILURE
fi
```

---

## Performance Impact

### Resource Usage

The script is lightweight:

| Metric | Value |
|--------|-------|
| Memory used | ~2-5 MB while running |
| CPU usage | <1% during execution |
| Execution time | 2-7 seconds (depending on action) |
| Disk space | ~1-5 MB per month of logs |

### Impact on System

Running every 15 minutes causes:
- **Negligible CPU impact** - Script is idle for 14 minutes, active for ~5 seconds
- **Minimal I/O** - One read from Asterisk, one log write
- **No network flooding** - Smart retry logic prevents spam

---

## Reliability & Safety

### Built-In Safeguards

1. **Retry Limit** - Won't spam reconnection attempts
2. **Timeout Protection** - Won't hang if Asterisk is unresponsive
3. **Lock File** - Prevents two instances running simultaneously
4. **Input Validation** - Checks configuration before starting
5. **Graceful Failure** - Handles errors without crashing

### What Could Go Wrong?

**Scenario:** Asterisk system becomes unresponsive
- **Impact:** Script will timeout and try again next cycle (15 minutes later)
- **Result:** Acceptable - no impact on other services

**Scenario:** Network is completely down
- **Impact:** Script will fail to reconnect (expected)
- **Result:** Acceptable - human intervention needed anyway

**Scenario:** Configuration is incorrect
- **Impact:** Script logs error and exits cleanly
- **Result:** Acceptable - operator fixes config and tries again

**Scenario:** Disk full (can't write logs)
- **Impact:** Script still attempts to reconnect
- **Result:** Acceptable - reconnection attempted even if logging fails

---

## Upgrading & Version History

### Current Version: 2.0.0

**New in Version 2.0:**
- ✅ 3-attempt retry system (was manual before)
- ✅ Reconnection verification (wasn't verified before)
- ✅ Structured logging (was unformatted text before)
- ✅ Configuration validation (no validation before)
- ✅ Email alerts (new feature)
- ✅ Better error messages (were cryptic before)

**Backward Compatibility:**
- ✅ Works with old configurations
- ✅ Old log files still readable
- ✅ No breaking changes

**Upgrading from 1.x:**
- Simply replace the script file
- No configuration changes needed
- First run will create new log files

See [CHANGELOG.md](CHANGELOG.md) for complete change history.

---

## Troubleshooting Guide

### Common Issues

#### Issue: "Script won't start"
**Symptom:** Error about LOG_DIR not set
**Cause:** Configuration not provided
**Solution:** Set up configuration file or environment variables (see INSTALL.md)

#### Issue: "Connection keeps dropping"
**Symptom:** Logs show repeated disconnections
**Cause:**
- Network instability
- Asterisk configuration issue
- Remote node is offline
**Solution:** Check network connectivity, verify Asterisk settings, contact remote operator

#### Issue: "Script isn't running from cron"
**Symptom:** Cron job is set but nothing happens
**Cause:** Permission issues, wrong user, cron not running
**Solution:** Check permissions, verify cron daemon running, check cron logs

#### Issue: "Logs are growing too large"
**Symptom:** Disk space consumed rapidly
**Cause:** Log rotation not configured
**Solution:** Configure logrotate (see INSTALL.md)

See [README.md](README.md#troubleshooting) for more detailed troubleshooting.

---

## Security Considerations

### User Permissions

The script should **NOT** run as root. Instead:
- Run as dedicated `asterisk` user
- Or run as regular user with Asterisk CLI access
- Running as root unnecessarily increases security risk

### Network Security

The script:
- Only communicates with local Asterisk socket (not over network)
- Doesn't open any ports
- Doesn't access the internet
- Doesn't transmit data outside the local system

### Data Security

Log files contain:
- Node IDs (not sensitive)
- Timestamps (not sensitive)
- Connection status (not sensitive)

**Not logged:**
- Passwords
- Credentials
- Personal information

---

## Getting Help

### Documentation

1. **Quick Start** → Read [README.md](README.md)
2. **Installation** → Read [INSTALL.md](INSTALL.md)
3. **What's New** → Read [CHANGELOG.md](CHANGELOG.md)
4. **Configuration** → See `autoconnect.conf.example`

### Troubleshooting

1. **Check the logs** - Most information is there
   ```bash
   tail -50 /var/log/allstar/autoconnect.log | grep ERROR
   ```

2. **Test manually** - Run the script by hand to see output
   ```bash
   /usr/local/bin/autoconnect
   ```

3. **Verify configuration** - Make sure settings are correct
   ```bash
   cat ~/.autoconnect.conf
   ```

### Reporting Issues

If something isn't working:
1. Gather logs from `/var/log/allstar/autoconnect.log`
2. Note the exact error message
3. Describe what you expected vs. what happened
4. Provide your configuration (without sensitive data)

---

## Technical Architecture

### Script Design Philosophy

The script follows these principles:

1. **Simple** - Easy to understand and modify
2. **Reliable** - Handles edge cases and errors gracefully
3. **Observable** - Detailed logs for troubleshooting
4. **Defensive** - Validates input, checks assumptions
5. **Non-Intrusive** - Minimal system impact

### How It's Implemented

**Language:** Bash shell script
- Portable (works on any Linux system)
- Simple (no complex dependencies)
- Familiar (most sysadmins know shell scripting)

**Execution Model:**
- Triggered by cron scheduler every 15 minutes
- Runs independently (no daemon)
- Creates lock file to prevent concurrent runs
- Exits cleanly after each execution

**Key Functions:**

1. `check_connection()` - Queries Asterisk for node status
2. `attempt_reconnection()` - Issues reconnect command
3. `log_entry()` - Records all actions with timestamps
4. `validate_configuration()` - Checks setup before running
5. `read_retry_state()` / `update_retry_state()` - Manages retry counter

---

## FAQ (Frequently Asked Questions)

### General Questions

**Q: Do I need to understand shell scripting to use this?**
A: No. The installation is straightforward, and configuration is simple. You only need basic command-line familiarity.

**Q: Can I run multiple instances on different nodes?**
A: Yes. Each node gets its own copy of the script with different configuration (different node IDs, log directories).

**Q: What if the nodes are already connected when the script runs?**
A: It just logs "connected" and exits. No action taken. Very quick.

**Q: How much will this cost to run?**
A: Free. The script is open source. Only cost is minimal disk space for logs.

### Operational Questions

**Q: How often does the script check?**
A: Every 15 minutes (or whatever you set in cron). Can be adjusted.

**Q: How long does reconnection take?**
A: Usually 5-10 seconds if successful. If failed, next check is in 15 minutes.

**Q: Will this fix network problems?**
A: No. It fixes software issues (dropped connections). Real network outages still need human intervention.

**Q: Can I get email alerts?**
A: Yes. Optional feature. Enabled in configuration.

### Technical Questions

**Q: What if Asterisk crashes while the script is running?**
A: Script will timeout after 10 seconds and try again next cycle.

**Q: Does the script conflict with other monitoring tools?**
A: No. It only reads status and issues commands. Won't interfere with Nagios, Zabbix, etc.

**Q: Can I modify the script?**
A: Yes. It's designed to be readable and modifiable. Just test thoroughly before deploying changes.

---

## Next Steps

### For Installation
1. Read [INSTALL.md](INSTALL.md)
2. Follow step-by-step instructions
3. Test manually: `/usr/local/bin/autoconnect`
4. Verify logs are being created
5. Monitor for 24 hours to ensure it's working

### For Monitoring
1. Set up log rotation (see INSTALL.md)
2. Configure email alerts (optional, see INSTALL.md)
3. Add to Nagios/Zabbix if you use them (see README.md#Integration)
4. Brief your team on what to expect

### For Ongoing Management
1. Review logs periodically (weekly)
2. Watch for persistent failure flag
3. Monitor disk space for logs
4. Update script when new versions available

---

## Document Version & Updates

- **Document Version:** 2.0.0
- **Script Version:** 2.0.0
- **Last Updated:** 2024-01-24
- **Valid For:** AllStar Auto-Connect v2.0.0 and compatible versions

For the latest information and updates, see [CHANGELOG.md](CHANGELOG.md).
