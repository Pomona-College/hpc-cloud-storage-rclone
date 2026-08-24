---
title: "Automating with Cron"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How can I schedule automatic syncs?
- What is cron and how do I create cron jobs?
- How do I debug scheduled sync jobs?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand cron scheduling syntax
- Create cron jobs for recurring rclone syncs
- Build shell scripts with error handling and notifications
- Debug common cron job issues

::::::::::::::::::::::::::::::::::::::::::::::

## Why Schedule Syncs?

Instead of manually running `rclone sync`, you can schedule it to run automatically:

- **Daily backups**: Sync your HPC work to Box every night
- **Regular downloads**: Update your local data from cloud sources
- **Continuous mirroring**: Keep two locations perfectly synchronized

## Understanding Cron

Cron is a Linux job scheduler. A cron job runs a command at specified times.

**Cron format:**
```
minute hour day_of_month month day_of_week command
```

Examples:
```
0 2 * * *     # Every day at 2:00 AM
0 */4 * * *   # Every 4 hours
0 2 * * 1     # Every Monday at 2:00 AM; 1 = Monday
30 9 1 * *    # First day of month at 9:30 AM
```

## Creating a Cron Job

Edit your cron table:

```bash
crontab -e
```

Add a new line for your scheduled sync:

```bash
# Sync HPC results to Box every night at 2 AM
0 2 * * * module load rclone && rclone sync /scratch/results pomona-box:/daily-backup --log-file ~/.rclone-sync.log
```

Save and exit the editor. Verify the job is scheduled:

```bash
crontab -l
```

## Checking Cron Logs

After your job runs, check if it succeeded:

```bash
cat ~/.rclone-sync.log
```

If you see token errors, your token has expired. Use `rclone config reconnect` to refresh it.

## Advanced: Shell Script for Scheduled Syncs

For complex syncing with error handling and notifications, create a shell script:

```bash
#!/bin/bash
# File: ~/rclone-sync.sh

# Load modules
module load rclone

# Log file
LOGFILE="$HOME/.rclone-sync-$(date +%Y-%m-%d).log"

# Sync HPC results to Box
echo "Starting sync at $(date)" >> $LOGFILE
rclone sync /scratch/results pomona-box:/daily-backup \
  --log-file $LOGFILE \
  --backup-dir pomona-box:/results-archive

# Check result
if [ $? -eq 0 ]; then
  echo "Sync completed successfully" >> $LOGFILE
  echo "Sync successful!" | mail -s "rclone sync OK" $USER@pomona.edu
else
  echo "Sync FAILED" >> $LOGFILE
  echo "Check logs: $LOGFILE" | mail -s "rclone sync FAILED" $USER@pomona.edu
fi

echo "Sync finished at $(date)" >> $LOGFILE
```

Make it executable and schedule it:

```bash
chmod +x ~/rclone-sync.sh
crontab -e
# Add: 0 2 * * * ~/rclone-sync.sh
```

## Debugging Cron Jobs

Common issues when cron jobs don't work:

1. **Module not loaded**: `module: command not found` -- cron doesn't load your shell profile
2. **Path issues**: Paths must be absolute (e.g., `/scratch`, not `~/results`)
3. **Token expired**: Re-authenticate with `rclone config reconnect`
4. **Permissions**: User running cron might not have write permissions

### Use Full Paths and Absolute References

```bash
# WRONG - won't work in cron
0 2 * * * rclone sync ~/results myremote:/backup

# RIGHT - works in cron
0 2 * * * /usr/bin/rclone sync /rhome/<myusername>/results myremote:/backup
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 9.1: Create a Scheduled Sync Job

Create a cron job that syncs your test folder. First, create a test script:

```bash
cat > ~/sync-test.sh << 'EOF'
#!/bin/bash
module load rclone
echo "Sync running at $(date)" >> $HOME/.sync-test.log
rclone copy ~/test-data.txt pomona-box:/scheduled-sync-test \
  --log-file $HOME/.sync-test-$(date +%Y-%m-%d).log
echo "Sync finished at $(date)" >> $HOME/.sync-test.log
EOF

chmod +x ~/sync-test.sh
```

Schedule it to run every 5 minutes for testing:

```bash
crontab -e
# Add: */5 * * * * ~/sync-test.sh
```

Wait 5 minutes, then check the log and verify on Box. Once satisfied, remove the test job from crontab.

::::::::::::::::::::::::::::::::::::: solution

## Solution

After waiting 5 minutes, check:

```bash
cat ~/.sync-test.log
```

Expected output shows timestamps of each run. Verify the file reached Box:

```bash
rclone ls pomona-box:/scheduled-sync-test
```

After confirming, remove the cron job with `crontab -e` and delete the test line. Verify removal with `crontab -l`.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- Cron jobs require absolute file paths and may need module load rclone in the command
- Shell scripts with error handling and email notifications make automated syncs more robust
- Always test cron jobs by running the command manually first before scheduling
- Check cron logs regularly to catch token expiration and other failures early

::::::::::::::::::::::::::::::::::::::::::::::

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
