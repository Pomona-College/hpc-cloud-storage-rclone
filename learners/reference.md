# Quick Reference: rclone Commands

This reference guide covers essential rclone commands and configuration steps for syncing data between the Sagehen cluster and cloud storage (Box or OneDrive).

## Loading rclone on Sagehen

Before using rclone, load the module on Sagehen:

```bash
module load rclone
rclone version
```

## Configuring Remote Storage (OAuth Setup)

### Initial Configuration

To configure a new remote (Box or OneDrive), start the interactive setup:

```bash
rclone config
```

### Box Configuration

When prompted, follow these steps:

```
name> box-storage
Type of storage> box
Box App ID and Secret> (leave blank for Pomona account)
Use Box API> N
Scope that rclone should use> rclone
Advanced config> N
Auto config> Y
```

A browser window will open for OAuth authentication. Log in with your Pomona credentials and authorize rclone access.

### OneDrive Configuration

When prompted:

```
name> onedrive-storage
Type of storage> onedrive
Client ID and Secret> (leave blank for standard config)
Region> global
Advanced config> N
Auto config> Y
```

Again, authorize in your browser using your Microsoft account credentials.

### View Existing Remotes

```bash
rclone listremotes
```

### Manage Configurations

```bash
rclone config show
rclone config edit
```

## Essential rclone Commands

### Listing Files and Directories

**List all files in a remote:**
```bash
rclone ls box-storage:/
```

**List directories only:**
```bash
rclone lsd box-storage:/
```

**List with detailed information:**
```bash
rclone ls --human-readable box-storage:/path/to/folder
```

### Copying Files

**Copy from remote to Sagehen:**
```bash
rclone copy box-storage:/data /rhome/username/data
```

**Copy from Sagehen to remote:**
```bash
rclone copy /rhome/username/data box-storage:/backup
```

**Copy single file:**
```bash
rclone copy box-storage:/results.csv /rhome/username/
```

### Synchronizing Directories

**One-way sync (remote → local):**
```bash
rclone sync box-storage:/data /rhome/username/data
```

**One-way sync (local → remote):**
```bash
rclone sync /rhome/username/data box-storage:/backup
```

**Check what would be synced (dry-run):**
```bash
rclone sync --dry-run /rhome/username/data box-storage:/backup
```

### Mounting Cloud Storage

**Mount remote as filesystem:**
```bash
rclone mount box-storage:/ ~/box-mount &
```

**List mounted content:**
```bash
ls ~/box-mount
```

**Unmount:**
```bash
fusermount -u ~/box-mount
```

### Checking File Integrity

**Verify copied files match:**
```bash
rclone check box-storage:/data /rhome/username/data
```

**Show differences:**
```bash
rclone check --one-way box-storage:/data /rhome/username/data
```

## Common Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `--progress` | Show transfer progress | `rclone copy --progress source dest` |
| `--dry-run` | Preview changes without executing | `rclone sync --dry-run source dest` |
| `--transfers N` | Parallel transfers (default 4) | `rclone copy --transfers 8 source dest` |
| `--checkers N` | Parallel checkers (default 8) | `rclone check --checkers 16 source dest` |
| `--verbose` | Detailed output | `rclone ls --verbose remote:/` |
| `--human-readable` | Human-friendly file sizes | `rclone ls --human-readable remote:/` |
| `--exclude PATTERN` | Exclude files matching pattern | `rclone copy --exclude "*.tmp" source dest` |
| `--include PATTERN` | Include only matching files | `rclone copy --include "*.csv" source dest` |
| `--delete-excluded` | Delete excluded files from destination | `rclone sync --delete-excluded source dest` |
| `--no-traverse` | Don't scan destination | `rclone copy --no-traverse source dest` |
| `--bwlimit RATE` | Bandwidth limit | `rclone copy --bwlimit 1M source dest` |

## Troubleshooting Token Issues

### Token Expiration

If you encounter "invalid token" errors:

```bash
rclone config reconnect box-storage
```

Then reauthenticate in the browser window that opens.

### Re-authorize a Remote

```bash
rclone config edit
# Select your remote and update the token
rclone config reconnect box-storage
```

## Sagehen-Specific Paths

When using rclone on Sagehen, remember storage locations:

- **Personal:** `/rhome/username` (100 GB, backed up)
- **Lab/Group:** `/bigdata/groupname` (1 TB shared)
- **Temporary:** `/scratch/username` (no quota, 60-day retention)
- **Fast Temp:** `/tmpfs` (RAM-based, job duration only)

Example sync to shared lab storage:
```bash
rclone sync box-storage:/results /bigdata/mylab/results
```

## Performance Tips

1. **Use `--transfers` for large transfers:** `rclone copy --transfers 12 source dest`
2. **Limit bandwidth on shared systems:** `rclone copy --bwlimit 10M source dest`
3. **Use `/scratch` for temporary downloads:** Faster SSD storage
4. **Monitor transfer status:** Add `--progress` flag
5. **Verify large transfers:** Use `rclone check` afterward

## Getting Help

**View help for any command:**
```bash
rclone help command
```

**Full rclone documentation:**
```bash
man rclone
```

**HPC Support:**
- Email: its-hpc@pomona.edu
- Instructor: Andrew Wilson
