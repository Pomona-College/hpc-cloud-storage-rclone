# Instructor Notes: Cloud Storage Integration with rclone

**Workshop Instructor:** Andrew Wilson  
**Contact:** its-hpc@pomona.edu  
**Cluster:** Sagehen (sagehen.hpc.pomona.edu)  
**Scheduler:** SLURM  
**Module System:** Lmod

## Overview

This 6-episode workshop teaches researchers how to integrate cloud storage (Box and OneDrive) with the Sagehen HPC cluster using rclone. Participants will learn to transfer data, synchronize directories, mount cloud storage, and troubleshoot common issues.

**Target Audience:** Faculty, staff, graduate students, and undergraduates working with data across cloud and HPC environments.

**Total Workshop Duration:** Approximately 2.5-3 hours (including breaks)

## Episode Breakdown and Timing

### Episode 1: What is rclone and Why Use It?
**Duration:** 20-25 minutes

**Learning Objectives:**
- Understand what rclone is and its capabilities
- Recognize use cases for cloud-to-HPC workflows
- Understand rclone's advantages over manual downloads

**Key Teaching Points:**
- rclone is a command-line tool for syncing cloud storage with local/HPC filesystems
- Supports Box, OneDrive, Google Drive, Dropbox, S3, and many other services
- More efficient than manual downloads, especially for large datasets
- Pomona's Box and OneDrive integrations work seamlessly with rclone

**Demo Suggestions:**
- Show a typical research workflow: data in Box → download → upload results
- Explain why this is inefficient for large files or repeated transfers
- Demonstrate speed advantage: rclone syncs only changed files

**Common Questions:**
- Q: Is rclone secure for sensitive data?
  A: Yes. OAuth tokens are encrypted in rclone's config. Data transfers use HTTPS. Tokens auto-refresh.
- Q: Can I mount Box/OneDrive like a network drive?
  A: Yes! The mount feature allows this, covered in Episode 5.

---

### Episode 2: Setting Up Remote Connections
**Duration:** 30-35 minutes

**Learning Objectives:**
- Configure Box and OneDrive remotes using OAuth
- Understand the difference between local and remote storage concepts
- Verify configuration was successful

**Key Teaching Points:**
- OAuth eliminates the need to store passwords in plain text
- Pomona AD accounts integrate directly with Box
- Microsoft accounts work natively with OneDrive
- Configuration is stored locally in `~/.config/rclone/rclone.conf`

**Demo Setup (IMPORTANT):**
Before the workshop, prepare:
1. A test Box folder with sample data files
2. A test OneDrive folder with sample CSV/text files
3. Have your own remote already configured for live demo
4. Prepare a secondary account (if available) to demonstrate OAuth flow

**OAuth Flow Walkthrough:**
- Show the `rclone config` interactive prompt
- Walk through naming the remote
- When "Auto config" is selected, show browser window
- Point out Pomona login screen and consent page
- Emphasize that rclone doesn't see passwords; it only receives OAuth token

**Live Demo:**
```bash
# Show existing remotes
rclone listremotes

# Walk through config interactively (you pre-configure this)
rclone config show

# List files from configured remotes
rclone ls box-storage:/
rclone ls onedrive-storage:/
```

**Troubleshooting Common Issues:**

**Issue: "Auto config not working"**
- Solution: Use `--no-open-browser` flag if X11 forwarding unavailable
- Provide custom URL for manual browser authorization

**Issue: "Token already expired"**
- This shouldn't happen immediately after config
- If it does, may indicate system clock issues
- Ask participant to verify: `date` command on Sagehen

**Issue: Pomona login fails with DUO prompt**
- Expected behavior! DUO MFA is required
- Ensure participant has phone ready to approve MFA
- Can take 30+ seconds; be patient

**Issue: "Box organization not accessible"**
- Likely personal Box account, not Pomona Box
- Ask: "Did you log in with Pomona AD credentials?"
- Solution: Log out, use AD account instead

---

### Episode 3: Copying and Moving Data
**Duration:** 35-40 minutes

**Learning Objectives:**
- Use `rclone copy` for one-way transfers
- Understand `copy` vs `sync` semantics
- Transfer data between cloud and Sagehen

**Key Teaching Points:**
- `copy` transfers files without deleting anything
- Safe for first-time transfers or backups
- Parallel transfers speed up large operations
- Progress tracking helps monitor long transfers

**Live Demo Setup:**
Prepare sample files in Box/OneDrive:
- A few small text files (< 1 MB)
- A medium CSV file (5-10 MB)
- A folder structure with nested files

**Demonstration Sequence:**

1. **List files on cloud:**
   ```bash
   rclone ls box-storage:/workshop-data --human-readable
   ```

2. **Copy single file:**
   ```bash
   rclone copy --progress box-storage:/workshop-data/sample.csv /rhome/awilson/demo/
   ```
   Point out: "Checking remote" and "Copying" status

3. **Copy with multiple transfers:**
   ```bash
   rclone copy --transfers 8 --progress box-storage:/workshop-data /rhome/awilson/demo/
   ```
   Explain: Parallel transfers speed up multiple files

4. **Copy the other direction:**
   ```bash
   # Create sample results
   echo "Results from analysis" > /rhome/awilson/demo/results.txt
   
   # Copy back to Box
   rclone copy --progress /rhome/awilson/demo/results.txt box-storage:/workshop-results/
   ```

**Bandwidth Considerations:**
- Sagehen uses shared network infrastructure
- Large transfers may impact cluster performance
- Demonstrate bandwidth limiting:
  ```bash
  rclone copy --bwlimit 10M --progress source dest
  ```
- Recommend off-peak hours for very large transfers (>10 GB)
- Suggest using `/scratch` for intermediate operations (faster SSD)

**Hands-On Exercise:**
1. Provide Box/OneDrive folder with sample files
2. Ask participants to: `rclone copy --progress box-storage:/workshop-data ~/download-test/`
3. Verify files arrived with `ls -lh ~/download-test/`
4. Ask: "Which storage location would be best for this data? Why?"

**Troubleshooting:**

**Issue: "Permission denied" errors**
- Solution: Verify file access in Box/OneDrive web interface
- Check Sagehen storage quota: `quota`
- Suggest trying small file first

**Issue: Very slow transfers**
- Likely high network congestion
- Suggest: Try `--transfers 2` to reduce load
- Check: `top` or `htop` for other processes
- Normal on shared systems

**Issue: Transfer stops mid-way**
- Network interruption (re-run, rclone resumes)
- Explain: rclone caches what's already transferred
- Show: How to check with `rclone ls dest:/`

---

### Episode 4: Synchronizing Directories
**Duration:** 35-40 minutes

**Learning Objectives:**
- Understand `rclone sync` semantics (dangerous!)
- Use `--dry-run` to preview changes
- Keep directories in sync with minimal transfers

**Key Teaching Points:**
- `sync` is bidirectional and **deletes** from destination
- Always use `--dry-run` first
- `sync` is more efficient than `copy` for repeated operations
- Perfect for backup workflows

**CRITICAL SAFETY WARNING:**
Before any sync demo, emphasize:

> "IMPORTANT: `rclone sync` DELETES files from the destination to match the source. If you sync the wrong direction, you can lose data. ALWAYS use `--dry-run` first."

**Live Demo Setup:**
Create a scenario:
1. Original folder with files A, B, C in Box
2. Destination folder on Sagehen with A, B, C, D, E
3. Demonstrate what `--dry-run` shows (would delete D and E)

**Demonstration Sequence:**

1. **Show what exists:**
   ```bash
   rclone ls box-storage:/workshop-sync/original/
   rclone ls /rhome/awilson/demo/sync-dest/
   ```

2. **Dry-run to preview:**
   ```bash
   rclone sync --dry-run --verbose box-storage:/workshop-sync/original/ /rhome/awilson/demo/sync-dest/
   ```
   Explain: "Would DELETE files not in source"

3. **Actually sync (with permission granted):**
   ```bash
   rclone sync --verbose box-storage:/workshop-sync/original/ /rhome/awilson/demo/sync-dest/
   ```

4. **Reverse direction (less common):**
   ```bash
   rclone sync --dry-run /rhome/awilson/demo/sync-dest/ box-storage:/workshop-sync/backup/
   ```

**Advanced Sync Options:**

Show how to handle special cases:

```bash
# Exclude temporary files
rclone sync --exclude "*.tmp" source dest

# Only sync CSV files
rclone sync --include "*.csv" --include "*.txt" source dest

# Don't traverse destination (faster for huge remote folders)
rclone sync --no-traverse source dest
```

**Hands-On Exercise:**

Set up a practice scenario:

1. Create sample box folder with 3 files
2. Ask: "Write the rclone command to sync this to `/rhome/yourusername/sync-test/`"
3. Require them to use `--dry-run` first
4. Have them verify the dry-run output
5. Have them execute without `--dry-run`

**Troubleshooting:**

**Issue: "Are you sure you want to delete X files?"**
- Expected! This is the safety prompt
- Type 'y' to confirm (or --no-confirm-delete to suppress)
- This is deliberate design

**Issue: Sync seems slow**
- May be checking each file
- Use `--no-traverse` if destination is very large
- Show: `--checkers N` to parallelize checking

**Issue: "Source and destination overlap"**
- Attempting to sync folder to subfolder of itself
- Common mistake with paths like `box-storage:/` → `/home/user/box/`
- Solution: Use specific subdirectories

---

### Episode 5: Mounting Cloud Storage
**Duration:** 25-30 minutes

**Learning Objectives:**
- Mount cloud storage as filesystem
- Understand mount limitations and performance
- Use mounted storage for transparent access

**Key Teaching Points:**
- FUSE mounts allow treating cloud storage like local filesystem
- Useful for applications that expect local files
- Performance may be slower than direct copy
- Mounts are process-specific (not persistent)

**Live Demo Setup:**

Prepare Box/OneDrive folder with files ready to mount

**Demonstration Sequence:**

1. **Create mount point:**
   ```bash
   mkdir -p ~/box-mount
   ```

2. **Start mount:**
   ```bash
   rclone mount box-storage:/ ~/box-mount &
   ```
   Explain the `&` (backgrounded process)

3. **Verify mount exists:**
   ```bash
   ls -la ~/box-mount/
   ```

4. **List cloud files as if local:**
   ```bash
   ls -lh ~/box-mount/workshop-data/
   find ~/box-mount -name "*.csv" -type f
   ```

5. **Open file with application:**
   ```bash
   cat ~/box-mount/workshop-data/sample.csv
   head -20 ~/box-mount/workshop-data/results.txt
   ```

6. **Show it's real-time:**
   - Add file to Box via web interface
   - Show it appears in mount: `ls ~/box-mount/`

7. **Unmount:**
   ```bash
   fusermount -u ~/box-mount
   ls ~/box-mount/  # Empty again
   ```

**Mount Performance Considerations:**

Explain trade-offs:
- **Pros:** Transparent access, no copying needed, real-time updates
- **Cons:** Slower than local files, higher latency, requires FUSE
- **Best for:** Occasional file access, browsing, small scripts
- **Not best for:** High-performance computing, frequent random access

**Hands-On Exercise:**

1. Have participants create own mount point
2. Mount your shared Box folder
3. Copy a file from mount to local: `cp ~/box-mount/data.csv ~/local-copy.csv`
4. Compare speeds: direct copy vs mount copy

**Troubleshooting:**

**Issue: "fusermount: user has no permission to run this program"**
- FUSE daemon not configured for user
- Contact: its-hpc@pomona.edu for FUSE setup
- Workaround: Use `rclone copy` instead of mount

**Issue: "Transport endpoint is not connected"**
- Mount has become stale/disconnected
- Solution: `fusermount -u ~/box-mount` and remount
- If persistent, check rclone logs: `rclone mount --log-level DEBUG`

**Issue: Files listed but give "Permission denied" when accessing**
- Token may have expired
- Try: `rclone config reconnect box-storage`
- Unmount and remount

**Issue: Very slow file access through mount**
- Expected for large files or many small files
- Show: `--transfers 8` to parallelize
- Recommend: Use `copy` instead for large transfers

---

### Episode 6: Troubleshooting and Best Practices
**Duration:** 20-25 minutes

**Learning Objectives:**
- Diagnose common rclone issues
- Implement secure workflows
- Optimize for Sagehen environment

**Key Teaching Points:**
- Token expiration is most common issue
- Logs help diagnose problems
- Security best practices protect credentials
- Sagehen quotas and policies affect strategy

**Common Issues and Solutions:**

**1. Token Expired / Authentication Failures**

Symptoms: "Invalid token" or "401 unauthorized"

Causes:
- Token naturally expires after months
- System clock drift (rare but happens)
- Token revoked in Pomona Box/OneDrive portal

Solutions:
```bash
# Reconnect with new token
rclone config reconnect box-storage

# Check system clock
date

# Check config file permissions
ls -la ~/.config/rclone/
```

Demo: Actually reconnect live to show OAuth flow again

**2. Network Timeouts on Sagehen**

Symptoms: Transfer stalls or fails mid-operation

Causes:
- Network congestion (shared infrastructure)
- Very large file > transfer timeout
- Firewall issues (rare)

Solutions:
```bash
# Reduce parallel transfers
rclone copy --transfers 2 source dest

# Set explicit timeout
rclone copy --timeout 10m source dest

# Use bandwidth limit to reduce congestion
rclone copy --bwlimit 5M source dest
```

**3. Storage Quota Exceeded**

Symptoms: "Quota exceeded" errors

On Sagehen:
- `/rhome`: 100 GB limit
- `/bigdata`: Lab quota (typically 1 TB)
- `/scratch`: No quota but 60-day auto-deletion

Solutions:
```bash
# Check quota
quota

# Use compression for transfers
rclone copy --compression 9 source dest

# Offload to appropriate storage
# - Downloaded data? Use /scratch first, then move
# - Lab backups? Use /bigdata
# - Personal files? Use /rhome
```

**4. Permission Issues**

Symptoms: "Permission denied" or "Access forbidden"

Causes:
- Box/OneDrive file not accessible to logged-in user
- Sagehen folder permissions too restrictive
- Group storage access issues

Solutions:
```bash
# Verify file accessible in web UI first
# Check Sagehen folder ownership
ls -ld /rhome/username/folder

# Fix permissions
chmod 755 /path/to/folder
chmod 644 /path/to/file
```

**Security Best Practices:**

1. **Protect your config file:**
   ```bash
   # Readable only by you
   chmod 600 ~/.config/rclone/rclone.conf
   
   # View without exposing credentials
   rclone config show
   ```

2. **Rotate credentials periodically:**
   - Reconnect remotes every 6 months
   - Revoke old tokens in Box/OneDrive portal

3. **Don't share remotes across users:**
   - Each user should have their own rclone config
   - OAuth tokens are personal, not shareable

4. **Use exclude patterns for sensitive data:**
   ```bash
   # Avoid syncing sensitive files
   rclone sync --exclude "*.password" --exclude ".ssh/*" source dest
   ```

**Optimization for Sagehen:**

1. **Choose right storage location:**
   - Small working files: `/rhome`
   - Shared lab data: `/bigdata`
   - Large computation inputs: `/scratch` (faster SSD)
   - Temporary jobs: `/tmpfs` (RAM, fastest but limited)

2. **Batch transfers efficiently:**
   ```bash
   # One large transfer vs many small = faster
   # Group files by day/project
   rclone copy --transfers 4 /rhome/myproject/data /bigdata/backup/
   ```

3. **Schedule off-peak:**
   - Heavy rclone work during off-hours
   - Reduces cluster congestion
   - Better for other users' jobs

4. **Monitor transfers:**
   ```bash
   # Use progress for large ops
   rclone copy --progress source dest
   
   # Check resource use
   top
   ```

**Hands-On: Troubleshooting Scenarios**

Present real-world failures and have participants diagnose:

**Scenario 1:** "I got 'error 500' halfway through syncing 50 GB from Box"
- Likely: Network timeout or Box service interruption
- Solution: Re-run sync command (rclone continues)
- Question: Which rclone command should you use? (sync, not copy; only transfers changed)

**Scenario 2:** "My rclone mount shows files but I can't open them"
- Likely: Token expired
- Solution: Reconnect remote, remount
- Prevention: Monitor token age in config

**Scenario 3:** "I synced to /rhome but now I'm over quota"
- Likely: Didn't plan storage carefully
- Solution: Move to /bigdata or /scratch
- Prevention: Check quota before large transfers

**Hands-On Exercise:**

Have participants run through complete workflow:
1. Configure a test remote (if not already)
2. List cloud files
3. Copy sample file with --progress
4. Create local file
5. Copy back to cloud
6. Sync a folder with --dry-run
7. Mount and browse files
8. Unmount

## Workshop Materials

### Pre-Workshop
- Email setup.md checklist to participants 1 week before
- Test rclone config (prepare test Box/OneDrive folders)
- Verify all participants have HPC accounts
- Send OnDemand portal link: https://ondemand.hpc.pomona.edu/

### During Workshop
- Have reference.md available to project/distribute
- Print or share learner-profiles.md for context
- Keep instructor-notes.md handy for timing/troubleshooting
- Have test Box/OneDrive data ready
- Test network connectivity before starting

### Post-Workshop
- Provide reference.md PDF
- Share command examples as file
- Offer office hours for individual issues
- Collect feedback via survey

## Technical Setup Checklist (Before Workshop)

- [ ] Verify rclone module loads on Sagehen: `module load rclone`
- [ ] Test Box configuration: `rclone ls box-storage:/`
- [ ] Test OneDrive configuration: `rclone ls onedrive-storage:/`
- [ ] Prepare sample Box folder with test files
- [ ] Prepare sample OneDrive folder with test files
- [ ] Test mount point creation: `mkdir test-mount && rclone mount box:/test-mount &`
- [ ] Verify OnDemand access: https://ondemand.hpc.pomona.edu/
- [ ] Check FUSE daemon availability for mount demonstrations
- [ ] Prepare bandwidth-limited test (--bwlimit)
- [ ] Have sample job script showing rclone in SLURM context

## Troubleshooting for Instructors

**Issue: OnDemand portal unavailable**
- Contact: its-hpc@pomona.edu
- Fallback: Use SSH terminal directly if participants have tools

**Issue: rclone module not loading**
- Check: `module avail rclone`
- May need to load dependency: `module load rclone-base` or similar
- Fallback: Use locally installed rclone

**Issue: Box/OneDrive OAuth flow hangs**
- Ensure X11 forwarding available or use `--no-open-browser`
- Provide manual authorization URL
- May need network checks (firewall, proxy)

**Issue: Participants' DUO approval fails**
- Check: Participant has DUO app on phone
- Verify: Network can reach DUO servers
- May need: IT to re-enroll user

## Contact Information

- **Cluster Admin:** Pomona College HPC Team
- **Email:** its-hpc@pomona.edu
- **Instructor:** Andrew Wilson
- **Cluster:** sagehen.hpc.pomona.edu
- **OnDemand:** https://ondemand.hpc.pomona.edu/

## Additional Resources

- rclone Documentation: https://rclone.org/docs/
- rclone Box Guide: https://rclone.org/box/
- rclone OneDrive Guide: https://rclone.org/onedrive/
- FUSE Documentation: https://github.com/libfuse/libfuse
- Pomona Box: https://pomona.box.com
