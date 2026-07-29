---
title: "Troubleshooting and Best Practices"
teaching: 10
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions

- What should I do if authentication fails?
- How do I monitor transfer logs and verify successful syncs?
- What are the best practices for production rclone use?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Troubleshoot common authentication and connection errors
- Monitor transfer logs and manage log files
- Apply best practices for reliable rclone workflows
- Know where to get help

::::::::::::::::::::::::::::::::::::::::::::::

## Troubleshooting Authentication Errors

### Browser Doesn't Open Automatically

If the OAuth browser doesn't open during setup or reconnect, manually paste the URL shown in your terminal into your web browser.

### "Connection refused" Error

**Symptom:**
```
ERROR : error: connection refused on oauth callback
```

**Cause:** You forgot the `-L 53682:localhost:53682` flag in SSH.

**Solution:** Exit and reconnect SSH with port forwarding, then retry.

### "OAuth redirect mismatch"

**Symptom:**
```
ERROR : oauth2: server redirect URL does not match configured redirect URI
```

**Cause:** You're running the command on a different machine than where you set it up.

**Solution:** Run `rclone config` on Sagehen with port forwarding `-L 53682:localhost:53682`.

### "rclone: command not found"

You forgot to load the module. Run:

```bash
module load rclone
```

### "Access Denied" During OAuth (OneDrive)

1. Ensure you're using your Pomona email account (not a personal Microsoft account)
2. Check that your account has OneDrive enabled (contact IT if unsure)
3. Sign out of all Microsoft accounts in your browser, sign back in with Pomona only
4. Retry the rclone configuration

### Force Re-authentication

If rclone caches a token it thinks is valid but isn't working:

```bash
rclone config reconnect pomona-box: --force
```

## Monitoring Transfer Logs

### Check Logs for Errors

```bash
# View recent errors
grep ERROR $HOME/.rclone*.log | tail -5

# View successful transfers
grep "Transferred:" $HOME/.rclone*.log | tail -5
```

### Monitor Transfer Sizes

Before large syncs, check how much data you're about to transfer:

```bash
rclone size /scratch/results
```

Check destination free space:

```bash
rclone about pomona-box:
```

### Log Cleanup Script

Create a script to remove old log files:

```bash
#!/bin/bash
# File: ~/cleanup-rclone-logs.sh
# Remove rclone logs older than 30 days
find $HOME -name ".rclone*.log" -mtime +30 -delete
find $HOME -name ".sync*.log" -mtime +30 -delete
echo "Cleaned up logs older than 30 days"
```

Schedule it monthly:

```bash
crontab -e
# Add: 0 0 1 * * ~/cleanup-rclone-logs.sh
```

## Best Practices for Production Use

1. **Monitor tokens**: Check `rclone about remote:` monthly
2. **Set reminders**: Refresh tokens proactively (don't wait for failures)
3. **Test cron jobs**: Run manually first, then schedule
4. **Use absolute paths**: Always use `/full/path`, not `~/relative`
5. **Log everything**: Always use `--log-file` for debugging
6. **Verify transfers**: Use `rclone check` after important syncs
7. **Plan for failures**: Email alerts, backup directories, dry-run before sync

## Getting Help

For questions or issues:
- **HPC Support**: its-hpc@pomona.edu
- **rclone Documentation**: https://rclone.org/docs/
- **Pomona Box Help**: IT Service Desk
- **OnDemand Portal**: https://ondemand.hpc.pomona.edu

## Congratulations!

You've completed the entire Cloud Storage Integration with rclone workshop! You now know:

1. Why cloud storage integration is important
2. How to set up Box and OneDrive with rclone
3. How to execute basic file operations
4. How to implement syncing workflows
5. How to automate transfers and maintain your setup

You're ready to integrate cloud storage into your research workflows on Sagehen HPC!

## Next Steps

- **Set up your primary workflow**: Choose Box or OneDrive and start syncing your research data
- **Implement automated backups**: Schedule nightly syncs of important results
- **Share with collaborators**: Use Box for collaboration with other Pomona users
- **Monitor your tokens**: Set a calendar reminder to refresh tokens every 2 months
- **Document your setup**: Create notes about which folders sync to where

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 12.1: Monitor and Manage Logs

Inspect your rclone logs and create a cleanup routine:

```bash
# View all rclone logs
ls -lah $HOME/.rclone*.log $HOME/.sync*.log 2>/dev/null

# View recent errors
grep ERROR $HOME/.rclone*.log | tail -5

# View successful transfers
grep "Transferred:" $HOME/.rclone*.log | tail -5
```

Create and run the cleanup script:

```bash
cat > ~/cleanup-rclone-logs.sh << 'EOF'
#!/bin/bash
find $HOME -name ".rclone*.log" -mtime +30 -delete
find $HOME -name ".sync*.log" -mtime +30 -delete
echo "Cleaned up logs older than 30 days"
EOF

chmod +x ~/cleanup-rclone-logs.sh
~/cleanup-rclone-logs.sh
```

**Record**: How many old log files were found and deleted?

::::::::::::::::::::::::::::::::::::: solution

## Solution

Running the log inspection commands shows your existing log files with sizes and dates. The `grep ERROR` command highlights any failed transfers; if no errors, it returns no output (good).

The cleanup script output:

```
Cleaned up logs older than 30 days
```

If you just created the logs today, nothing is deleted. This is expected. The script becomes useful over time as logs accumulate. Schedule it monthly with `0 0 1 * * ~/cleanup-rclone-logs.sh` in crontab.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- Common errors like "connection refused" and "command not found" have straightforward fixes related to port forwarding and module loading
- Monitor rclone logs with --log-file to debug failed transfers and track successful synchronizations over time
- Use rclone size and rclone about to check data volumes and available space before large transfers
- Contact its-hpc@pomona.edu for HPC-specific issues and consult rclone.org/docs for rclone documentation

::::::::::::::::::::::::::::::::::::::::::::::
