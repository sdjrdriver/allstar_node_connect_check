# AllStar Auto-Connect Script - Project Overview

> **Automated monitoring and recovery for AllStar VoIP node connections**

---

## 🎯 Project At A Glance

**What:** A monitoring script that keeps AllStar radio nodes connected by automatically detecting and fixing disconnections.

**Why:** Manual reconnection is slow and labor-intensive. This script catches problems within 15 minutes and fixes them automatically.

**For Whom:** Amateur radio operators and network administrators managing AllStar VoIP nodes.

**Status:** ✅ Production Ready (Version 2.0.0)

---

## 📋 Quick Facts

| Aspect | Details |
|--------|---------|
| **Type** | Bash shell script |
| **Size** | Single ~500-line script |
| **Dependencies** | Bash, cron, Asterisk |
| **License** | MIT |
| **Cost** | Free |
| **Maintenance** | Low (runs automatically) |
| **Learning Curve** | Low (straightforward setup) |

---

## 🚀 Key Features

### Intelligent Monitoring
- Checks connection status every 15 minutes
- Detects when nodes become disconnected
- Logs all activity with timestamps

### Automatic Recovery
- Attempts to reconnect automatically
- Verifies reconnection actually succeeded
- Stops after 3 attempts to prevent network spam

### Smart Alerting
- Logs all actions in readable format
- Sends to syslog for integration with monitoring systems
- Optional email alerts on persistent failures
- Creates failure flag for external monitoring

### Operational Safety
- Timeout protection (won't hang indefinitely)
- Concurrent execution prevention (lock file)
- Input validation before starting
- Graceful error handling

### Production Features
- Detailed structured logging (machine-parsable)
- Backward compatible with older versions
- Configurable via external config file
- Auto-creates log directories and files

---

## 📊 Before & After

### Before This Script

| Scenario | Time to Detect | Time to Fix | Manual Work |
|----------|---|---|---|
| Connection drops at 3 AM | 1-8 hours | Human intervention | 30 minutes |
| User reports connection | Minutes | Depends on availability | 15 minutes |
| Weekly disconnections | Days to weeks | Each time | 30 min/week |

### After This Script

| Scenario | Time to Detect | Time to Fix | Manual Work |
|----------|---|---|---|
| Connection drops at 3 AM | 15 minutes | Automatic | None |
| User reports connection | Minutes | Likely already fixed | 2 minutes |
| Weekly disconnections | 15 minutes | Automatic | Investigation only |

---

## 🔧 Installation Summary

**Complexity:** Low
**Time Required:** 10-15 minutes
**Required Skills:** Basic command-line familiarity

### Quick Steps

```bash
# 1. Copy script
sudo cp autoconnect /usr/local/bin/
sudo chmod 755 /usr/local/bin/autoconnect

# 2. Create config file
cp autoconnect.conf.example ~/.autoconnect.conf
# Edit with your node IDs and log directory

# 3. Add to cron
crontab -e
# Add: */15 * * * * /usr/local/bin/autoconnect

# 4. Test
/usr/local/bin/autoconnect
tail ~/.autoconnect.conf  # Check logs
```

**[Full installation guide →](INSTALL.md)**

---

## 📚 Documentation

| Document | Purpose | Audience |
|----------|---------|----------|
| [README.md](README.md) | User guide, features, troubleshooting | Everyone |
| [INSTALL.md](INSTALL.md) | Step-by-step installation | System admins |
| [TECHNICAL_OVERVIEW.md](TECHNICAL_OVERVIEW.md) | Non-technical explanation of how it works | Decision makers, managers |
| [CHANGELOG.md](CHANGELOG.md) | Version history and what's new | Upgraders |
| [autoconnect.conf.example](autoconnect.conf.example) | Configuration template | Setup users |

---

## 🏗️ How It Works (Simplified)

```
Every 15 minutes:
    1. Check if nodes are connected
    2. If yes → Log success, exit
    3. If no → Check retry count
    4. If retries < 3 → Attempt reconnection
    5. Verify connection succeeded
    6. If max retries exceeded → Alert admin
    7. Log everything
    8. Exit
```

More detailed explanation: [See TECHNICAL_OVERVIEW.md](TECHNICAL_OVERVIEW.md)

---

## 💡 Real-World Example

**Scenario:** Your repeater's link to the main hub unexpectedly disconnects

**What would happen WITHOUT this script:**
- 3 AM: Connection drops (nobody notices)
- 6 AM: Repeater operators realize users can't reach the link
- 7 AM: Hub operator gets contacted
- 8 AM: Someone logs in and manually reconnects
- **Total downtime: 5 hours**

**What happens WITH this script:**
- 3 AM: Connection drops
- 3:15 AM: Script detects and attempts automatic reconnection
- 3:16 AM: Connection restored
- **Total downtime: 1 minute**

---

## ✅ Benefits Summary

### For Operators
- **Less downtime** - Fixed within 15 minutes automatically
- **Better visibility** - Know exactly what's happening with logs
- **Peace of mind** - Works 24/7 without manual intervention

### For Network Administrators
- **Easier management** - One less thing to monitor manually
- **Clear diagnostics** - Detailed logs for troubleshooting
- **Integration ready** - Works with Nagios, Zabbix, etc.

### For Organizations
- **Higher reliability** - Connections maintained reliably
- **Reduced labor** - No need for overnight monitoring
- **Lower cost** - Free software, minimal maintenance

---

## 📋 System Requirements

### Minimum
- Linux/Unix system (Ubuntu, CentOS, Debian, etc.)
- Asterisk PBX installed and configured
- Bash shell
- Cron daemon

### Recommended
- Dedicated log directory: `/var/log/allstar/`
- Dedicated user account (not root)
- Log rotation configured (optional)
- Monitoring system integration (optional)

---

## 🔒 Safety & Reliability

### Design Safeguards
- ✅ Won't spam reconnection attempts (max 3 retries)
- ✅ Prevents concurrent execution (lock file)
- ✅ Timeout protection (won't hang forever)
- ✅ Validates configuration before starting
- ✅ Handles errors gracefully

### What It Won't Do
- ❌ Fix hardware problems
- ❌ Repair network outages
- ❌ Replace manual configuration
- ❌ Solve Asterisk crashes (only detects them)

---

## 📈 Success Metrics

### Measurable Improvements

After deploying this script, you should see:

1. **Faster Recovery Time**
   - Before: Hours to detect and fix
   - After: Minutes with automatic fix

2. **Reduced Manual Intervention**
   - Before: Someone needed to monitor 24/7
   - After: Automatic, audit logs only

3. **Better Diagnostics**
   - Before: No record of what happened
   - After: Detailed log of every action

4. **Higher Availability**
   - Before: Connection may be down for hours
   - After: Down for ≤15 minutes maximum

---

## 🚦 Getting Started

### Step 1: Review the Documentation
- Read [README.md](README.md) for overview
- Read [TECHNICAL_OVERVIEW.md](TECHNICAL_OVERVIEW.md) if non-technical
- Review [autoconnect.conf.example](autoconnect.conf.example) for configuration

### Step 2: Install (10 minutes)
- Follow [INSTALL.md](INSTALL.md) step-by-step
- Copy script to binary directory
- Configure with your node IDs
- Add to crontab

### Step 3: Verify (5 minutes)
- Run script manually to test
- Check that logs are created
- Verify connection is detected correctly

### Step 4: Monitor (ongoing)
- Check logs weekly
- Review for patterns or issues
- Adjust configuration if needed

---

## 🆘 Troubleshooting

### Common Issues
- **"Script won't start"** → See [README.md Troubleshooting](README.md#troubleshooting)
- **"Connection keeps dropping"** → Check Asterisk logs
- **"Cron isn't running it"** → See [INSTALL.md](INSTALL.md#problem-cron-job-not-running)
- **"Need help"** → See [TECHNICAL_OVERVIEW.md FAQ](TECHNICAL_OVERVIEW.md#faq)

---

## 📝 Version Information

**Current Version:** 2.0.0 (Production Ready)

**Latest Changes:**
- ✅ 3-attempt retry system
- ✅ Reconnection verification
- ✅ Structured logging
- ✅ Configuration validation
- ✅ Email notifications
- ✅ Syslog integration

**Compatibility:** Works with Asterisk 11+, AllStar nodes

For detailed version history: [CHANGELOG.md](CHANGELOG.md)

---

## 📄 License

This project is released under the **MIT License**. See [LICENSE](LICENSE) for details.

**In summary:**
- ✅ Free to use
- ✅ Free to modify
- ✅ Free to distribute
- ⚠️ Provided as-is (no warranty)
- ⚠️ Not liable for problems it causes

---

## 👥 For Different Roles

### For Amateur Radio Operators
Start here: [README.md](README.md)
- Understand what it does
- Install it
- Monitor your connection

### For System Administrators
Start here: [INSTALL.md](INSTALL.md)
- Detailed setup instructions
- Configuration options
- Integration with monitoring systems

### For Decision Makers / Managers
Start here: [TECHNICAL_OVERVIEW.md](TECHNICAL_OVERVIEW.md)
- Non-technical explanation
- Benefits and costs
- Implementation timeline

### For Developers / Advanced Users
Start here: [autoconnect script](autoconnect)
- Review source code
- Modify as needed
- Contribute improvements

---

## 🔗 Quick Links

| Topic | Link |
|-------|------|
| **Download** | Clone repository or download script |
| **Installation** | [INSTALL.md](INSTALL.md) |
| **Usage Guide** | [README.md](README.md) |
| **Version History** | [CHANGELOG.md](CHANGELOG.md) |
| **Configuration Example** | [autoconnect.conf.example](autoconnect.conf.example) |
| **Technical Details** | [TECHNICAL_OVERVIEW.md](TECHNICAL_OVERVIEW.md) |
| **License** | [LICENSE](LICENSE) |

---

## 📞 Support & Contribution

### Getting Help
1. Check [README.md Troubleshooting](README.md#troubleshooting)
2. Review logs: `tail -f /var/log/allstar/autoconnect.log`
3. Read [TECHNICAL_OVERVIEW.md FAQ](TECHNICAL_OVERVIEW.md#faq)
4. Check [INSTALL.md Troubleshooting](INSTALL.md#troubleshooting-installation)

### Reporting Issues
- Describe what you expected vs. what happened
- Provide relevant log excerpts
- Include your configuration (without sensitive data)
- Note your Asterisk and system version

### Contributing Improvements
- Fork the repository
- Test thoroughly
- Document your changes
- Submit with clear description

---

## 🎓 Learning Resources

### Understanding AllStar
- [AllStar Documentation](http://www.allstarlink.org/)
- [Asterisk PBX](https://www.asterisk.org/)

### Understanding This Script
- [TECHNICAL_OVERVIEW.md](TECHNICAL_OVERVIEW.md) - How it works
- [README.md](README.md) - Feature documentation
- [CHANGELOG.md](CHANGELOG.md) - What changed

### Bash Scripting (if you want to modify)
- [Bash Guide](https://mywiki.wooledge.org/BashGuide)
- [ShellCheck](https://www.shellcheck.net/) - Script analyzer

---

## ✨ Key Takeaways

1. **Simple** - Single script, easy to understand
2. **Reliable** - Handles errors gracefully
3. **Automatic** - Works without manual intervention
4. **Observable** - Detailed logs for troubleshooting
5. **Free** - Open source, no licensing costs
6. **Proven** - Tested and used in production

---

## 📊 Project Statistics

- **Script Size:** ~500 lines with documentation
- **Setup Time:** 10-15 minutes
- **Learning Curve:** Low (straightforward)
- **Maintenance:** Minimal (runs automatically)
- **Cost:** Free
- **Reliability:** Production-ready

---

## 🎯 Next Steps

### Ready to Install?
→ Go to [INSTALL.md](INSTALL.md)

### Want to Learn More?
→ Read [TECHNICAL_OVERVIEW.md](TECHNICAL_OVERVIEW.md)

### Need Detailed Information?
→ Check [README.md](README.md)

### Interested in Updates?
→ Watch the repository for new versions

---

**Last Updated:** 2024-01-24
**Version:** 2.0.0
**Status:** Production Ready ✅

For the complete project status and roadmap, see [CHANGELOG.md](CHANGELOG.md)
