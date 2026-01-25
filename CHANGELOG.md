# Changelog

All notable changes to the AllStar Auto-Connect script will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2024-01-24

### Added

#### Retry System
- **3-attempt retry mechanism** with state persistence via `retry_state.txt`
- Automatic retry on connection failures up to configurable limit
- State tracking: timestamp, attempt count, and last status
- Automatic state reset on successful reconnection
- Configurable max retries via `MAX_RETRIES` variable

#### Reconnection Verification
- Verification step after reconnect command to confirm connection succeeded
- Configurable verification delay via `VERIFY_DELAY` variable
- Only counts as successful if reconnection is verified

#### Error Handling & Robustness
- **Timeout protection** for all asterisk commands (default 10 seconds)
- Timeout is configurable via `ASTERISK_TIMEOUT` variable
- Proper exit codes for different failure scenarios:
  - `0`: Success (connected or reconnected)
  - `1`: Persistent failure (max retries exceeded)
  - `2`: Configuration error
  - `3`: Already running (lock file active)
  - `4`: Asterisk communication error
- Input validation with clear error messages
- Stale lock file detection and cleanup

#### Structured Logging
- ISO 8601 timestamp format: `2024-01-24T15:30:00-06:00`
- Structured log format: `[TIMESTAMP] [LEVEL] Message | metadata`
- Log levels: DEBUG, INFO, WARNING, ERROR, SUCCESS
- Metadata fields for contextual information
- Multiple log files:
  - `autoconnect.log` - Main structured activity log
  - `autoconnect_summary.log` - Daily summaries (for future use)
  - `cron_check.txt` - Legacy format (backward compatibility)

#### Configuration Management
- Environment variable support for all configuration options
- Optional external configuration file support:
  - User config: `~/.autoconnect.conf`
  - System config: `/etc/autoconnect.conf`
  - Override via: `AUTOCONNECT_CONFIG` environment variable
- INI-style configuration format for readability
- Configuration precedence: env vars → user config → system config → defaults

#### File & Directory Management
- Auto-creation of log directory if missing
- Auto-creation of log files if missing
- Automatic creation of legacy `cron_check.txt` for backward compatibility
- Permission validation with clear error messages
- Lock file mechanism (`autoconnect.lock`) to prevent concurrent execution

#### Notification System
- **Syslog integration** (enabled by default)
- Optional email notifications for persistent failures
- Failure flag file (`PERSISTENT_FAILURE`) for external monitoring systems
- Configurable notifications via `ENABLE_SYSLOG`, `ENABLE_EMAIL` variables
- Prevention of notification spam (only on persistent failures)

#### Input Validation
- Node IDs must be numeric with validation
- Configuration file path doesn't allow spaces
- Asterisk binary existence and executability check
- Timeout values must be numeric
- Max retries must be numeric
- Clear error messages for each validation failure

#### Code Quality
- Comprehensive inline documentation
- Function headers with purpose, arguments, and return values
- Well-organized code structure (~500 lines with comments)
- Proper bash practices: `set -euo pipefail`, quote variables, use `local` in functions
- Trap handlers for cleanup on exit

### Fixed

- **Windows artifact bug** (line 40): Changed `\asterisk` to `/usr/sbin/asterisk`
- **Dead example code** (lines 53-60): Removed leftover example comments
- **Hardcoded asterisk path**: Now configurable via `ASTERISK_BIN` variable
- **Missing error handling**: All asterisk commands now have proper error handling
- **No reconnection verification**: Reconnection success is now verified
- **No logging of errors**: Comprehensive error logging added
- **No retry system**: Complete retry implementation with state management

### Changed

- **Script size**: Increased from ~61 to ~500 lines (with comprehensive documentation)
- **Log format**: Changed to structured, parsable format
- **Configuration**: Can now be externalized to config file (backward compatible)
- **Error messages**: More descriptive with specific guidance
- **Command timeout**: All asterisk commands now timeout after 10 seconds

### Security Improvements

- Lock file prevents accidental concurrent execution
- Input validation prevents injection attacks
- Configuration validation prevents syntax errors
- Explicit path handling prevents execution from unexpected locations

### Backward Compatibility

✓ **Fully backward compatible** with version 1.x

- Existing configuration variables still work
- Script variable format unchanged
- `cron_check.txt` still written in legacy format
- Same cron job syntax works without modification
- Existing installations upgrade without changes needed
- Optional config file (not required)

**Migration Path**: Simply replace the script file. On first run:
- Creates new log files automatically
- Initializes retry state management
- Maintains existing cron_check.txt for compatibility

### Deprecation Notes

- `cron_check.txt` format deprecated but still supported for backward compatibility
- Recommend migrating to structured `autoconnect.log` for new monitoring systems
- Instructions for parsing new log format provided in documentation

### Documentation Additions

- Comprehensive README.md with feature list and configuration guide
- Updated INSTALL.md with setup procedures and testing steps
- Example configuration file: `autoconnect.conf.example`
- This CHANGELOG.md for version tracking
- Inline code documentation for all functions

### Testing

Tested scenarios:
1. Normal operation (connected) → success
2. Single disconnection → reconnects and verifies
3. Retry progression → attempts 1-3, then failure
4. Recovery after max retries → resets on reconnection
5. Asterisk unavailable → proper error handling
6. Timeout handling → commands timeout appropriately
7. Configuration validation → clear errors for invalid configs
8. Concurrent execution → blocked by lock file
9. Missing permissions → clear error messages
10. Config file loading → reads from external config

### Known Issues / Future Enhancements

- Log rotation not built-in (recommend `logrotate` config)
- Summary log not yet auto-populated (framework in place)
- Web dashboard for monitoring (future enhancement)
- Multiple destination nodes support (future enhancement)
- Prometheus metrics export (future enhancement)

### Installation Notes

Users upgrading from 1.x:
1. Backup existing configuration (if customized in script)
2. Replace script file with new version
3. (Optional) Create config file: `~/.autoconnect.conf` or `/etc/autoconnect.conf`
4. First run will auto-create new log files
5. Existing `cron_check.txt` will continue to be written

No breaking changes - upgrade is safe.

---

## [1.x] - Previous Versions

- Basic connection monitoring
- Simple logging to `cron_check.txt`
- No retry mechanism
- No error handling
- Hardcoded configuration within script

---

For upgrade instructions, see [INSTALL.md](INSTALL.md)
For configuration details, see [README.md](README.md)
