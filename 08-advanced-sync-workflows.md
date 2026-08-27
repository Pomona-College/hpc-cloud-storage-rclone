---
title: "Advanced Sync Workflows"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I download large datasets from cloud to HPC?
- How do I implement bidirectional workflows safely?
- How can I archive deleted files instead of removing them?
- How do I sync selectively by file type or directory?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Download datasets from cloud storage to HPC
- Implement safe bidirectional syncing patterns using separate folders
- Use `--backup-dir` to archive old files during sync
- Apply filters with `--include` and `--exclude` for selective syncing

::::::::::::::::::::::::::::::::::::::::::::::

## Downloading Datasets from Cloud to HPC

A collaborator has shared a large dataset on OneDrive. Download it to Sagehen HPC for processing:

This example uses a second remote named `collaborator-onedrive` -- configured
with the same `rclone config` process as Episode 3, choosing **onedrive**
instead of **box**. If you haven't set one up, substitute a remote you do have
(such as `pomona-box:`) to follow along; otherwise rclone will report
`didn't find section in config file`.

```bash
rclone sync collaborator-onedrive:/large-dataset /scratch/raw-data --dry-run
```

Once satisfied, execute:

```bash
rclone sync collaborator-onedrive:/large-dataset /scratch/raw-data
```

If the collaborator updates the dataset, re-run sync to get only the changes.

### Important: Watch `/scratch` Space

Large datasets can fill `/scratch` quickly. Monitor your usage:

```bash
rclone size /scratch/raw-data
```

Remember that `/scratch` is non-persistent SSD storage that is deleted when your job completes.

## Bidirectional Syncing (Separate Folders)

### The Challenge

Sometimes you want to sync in **both directions** without conflicts or accidental deletions.

### The Safe Pattern: Separate Sync Directories

Instead of syncing the same folder both ways, use separate folders:

```
HPC:
  /work/myproject/source/      <- You edit files here
  /work/myproject/from-cloud/  <- Cloud files sync here

Cloud (Box):
  /myproject/hpc-updates/      <- Your HPC updates go here
  /myproject/shared/           <- Shared files everyone edits here
```

### Setup Commands

From HPC, copy new HPC-generated files to Box:

```bash
rclone copy /work/myproject/source pomona-box:/myproject/hpc-updates
```

Download shared files from Box to HPC:

```bash
rclone sync pomona-box:/myproject/shared /work/myproject/from-cloud
```

### Why This Works

- Each direction has its own folder
- No conflicts between local and remote edits
- You explicitly decide what goes where
- No accidental deletions from automatic syncing

## Backing Up Deleted Files with --backup-dir

When syncing, `--backup-dir` moves deleted files to an archive folder instead of permanently removing them:

```bash
rclone sync /scratch/results pomona-box:/results \
  --backup-dir pomona-box:/results-archive
```

**Before:**
```
HPC /scratch/results:  file1.txt, file2.txt
Box /results:          file1.txt, file2.txt, file3.txt (old)
```

**After sync:**
```
Box /results:          file1.txt, file2.txt
Box /results-archive:  file3.txt  <- Moved here instead of deleted!
```

To recover an archived file:

```bash
rclone copy pomona-box:/results-archive/file3.txt pomona-box:/results/
```

## Selective Syncing with Filters

### Filter by Extension

Sync only certain file types:

```bash
rclone sync pomona-box:/large-dataset /scratch/data \
  --include "*.csv" \
  --exclude "*"
```

### Filter by Directory

Sync only a specific subdirectory:

```bash
rclone sync pomona-box:/large-dataset/month-2026-03 /scratch/march-data
```

### Bandwidth Limiting

To avoid overwhelming the network during large syncs:

```bash
rclone sync pomona-box:/data /scratch/data \
  --bwlimit 500M  # Limit to 500 MB/s
```

## Summary

| Task | Command | Destructive? |
|------|---------|--------------|
| Backup HPC results | `rclone copy /scratch pomona-box:/backup` | No |
| Mirror HPC to cloud | `rclone sync /scratch pomona-box:/mirror` | Yes |
| Download dataset | `rclone sync pomona-box:/data /scratch/data` | Yes |
| Archive deleted files | Add `--backup-dir` to sync | No (archived) |
| One-way bidirectional | Use separate folders + copy | No |

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 8.1: Cross-Cloud Transfer

Copy files from Box to OneDrive (using copy, not sync, to be safe):

```bash
echo "Box data" > ~/test.txt
rclone copy ~/test.txt pomona-box:/copy-test
rclone copy pomona-box:/copy-test pomona-onedrive:/copy-test
rclone cat pomona-onedrive:/copy-test/test.txt
```

**Record**: Did the file successfully transfer from Box to OneDrive?

::::::::::::::::::::::::::::::::::::: solution

## Solution

After running the cross-cloud copy:

```bash
rclone cat pomona-onedrive:/copy-test/test.txt
```

Expected output: `Box data`

Cross-cloud transfers work seamlessly with `rclone copy` -- rclone downloads from one provider and uploads to the other without saving to local disk.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 8.2: Backup with Archive

Set up a sync with `--backup-dir` to archive deleted files:

```bash
mkdir -p ~/backup-test
echo "File 1" > ~/backup-test/file1.txt
echo "File 2" > ~/backup-test/file2.txt

# Sync to Box
rclone sync ~/backup-test pomona-box:/backup-test \
  --backup-dir pomona-box:/backup-test-archive

# Delete locally and sync again
rm ~/backup-test/file2.txt
rclone sync ~/backup-test pomona-box:/backup-test \
  --backup-dir pomona-box:/backup-test-archive

# Check archive
rclone ls pomona-box:/backup-test-archive
```

**Record**: Is `file2.txt` now in the archive folder instead of deleted?

::::::::::::::::::::::::::::::::::::: solution

## Solution

After the second sync, check the archive:

```bash
rclone ls pomona-box:/backup-test-archive
```

Expected output:

```
        7 file2.txt
```

The `--backup-dir` flag moved `file2.txt` to the archive instead of permanently deleting it. This provides a safety net for recovering accidentally removed files.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- One-way mirroring (sync from HPC to cloud) is appropriate for daily results; use separate folders for bidirectional workflows
- The --backup-dir flag preserves deleted files in an archive instead of permanently removing them during sync
- Selective syncing with --include and --exclude filters allows you to sync only specific file types or directories
- Bandwidth limiting with --bwlimit prevents large transfers from overwhelming the shared network

::::::::::::::::::::::::::::::::::::::::::::::
