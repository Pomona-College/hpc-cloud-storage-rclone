---
title: "Syncing Basics"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- What is the difference between `rclone copy` and `rclone sync`?
- When should I use sync vs copy?
- What is a one-way mirror and why is it useful?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand when to use `rclone copy` vs `rclone sync`
- Implement one-way mirroring (sync from source to destination)
- Set up workflows for backing up HPC results to cloud storage

::::::::::::::::::::::::::::::::::::::::::::::

## Copy vs Sync: The Critical Difference

::::::::::::::::::::::::::::::::::::: callout

## Two Fundamentally Different Operations

### rclone copy: Additive (Safe)

```bash
rclone copy source destination
```

- Copies files from source to destination
- **Only adds** files to destination; never deletes
- If destination has extra files, they stay untouched
- Safe for backups and collaboration

**Example:**
```
Before copy:
  Source:  file1.txt, file2.txt
  Dest:    file3.txt

After copy:
  Dest:    file1.txt, file2.txt, file3.txt  <- All three present!
```

### rclone sync: Mirroring (Destructive)

```bash
rclone sync source destination
```

- Makes destination an **exact mirror** of source
- **Deletes** files in destination that don't exist in source
- Fast and powerful for one-way synchronization
- Dangerous if you don't understand what you're doing

**Example:**
```
Before sync:
  Source:  file1.txt, file2.txt
  Dest:    file1.txt, file2.txt, file3.txt

After sync:
  Dest:    file1.txt, file2.txt  <- file3.txt DELETED!
```

::::::::::::::::::::::::::::::::::::::::::::::

## When to Use Each

::::::::::::::::::::::::::::::::::::: callout

## Strategic Use Cases for Copy vs Sync

### Use `rclone copy` for:

- Backing up work (source = your files, destination = backup)
- Adding files to an archive that might have other content
- Collaboration when multiple people modify files
- Incremental backups

### Use `rclone sync` for:

- Mirroring simulation results to cloud storage (one-way mirror)
- Syncing a dataset from cloud to HPC for processing
- Maintaining an exact copy of a folder
- Publishing results to a distribution point

### Never Use `rclone sync` for:

- Bidirectional syncing between two places you both modify
- When you're unsure of the consequences of deletion
- Without `--dry-run` first!

::::::::::::::::::::::::::::::::::::::::::::::

## Workflow 1: Backing Up HPC Results to Box

You run a 3-day simulation on Sagehen HPC that produces results in `/scratch/results/`. You want to **backup these results to Box** before they age out of `/scratch`.

### The Safe Approach: Use `rclone copy`

```bash
rclone copy /scratch/results pomona-box:/research-backup/simulation-2026
```

This copies all files. Running the command again only copies changed files and **doesn't delete anything** from Box.

### Preview Before Copying Large Data

```bash
rclone copy /scratch/results pomona-box:/research-backup/simulation-2026 --dry-run -v
```

### Verification

```bash
rclone check /scratch/results pomona-box:/research-backup/simulation-2026
```

## Workflow 2: Syncing HPC Results (One-Way Mirror)

You're running daily simulations and want to keep a **mirrored copy on Box**. Each day, only the latest results matter; old results should be cleaned up.

### The Sync Approach

Always use `--dry-run` first:

```bash
rclone sync /scratch/daily-results pomona-box:/daily-results --dry-run
```

Once verified, execute:

```bash
rclone sync /scratch/daily-results pomona-box:/daily-results
```

This keeps Box in perfect sync with your local folder, automatically cleaning up old files.

## Preventing Mistakes: Key Commands

### Always preview sync operations:

```bash
rclone sync source destination --dry-run
```

### Check what would be deleted:

```bash
rclone sync source destination --dry-run -vv
```

### Use size check before large operations:

```bash
rclone size source
rclone size destination
```

## Common Mistakes to Avoid

### Mistake 1: Reversing Source and Destination

```bash
# WRONG! This deletes your HPC files!
rclone sync pomona-box:/backup /scratch/data

# RIGHT! This syncs HPC to Box
rclone sync /scratch/data pomona-box:/backup
```

### Mistake 2: Using sync when you mean backup

`sync` makes the destination *match* the source -- including deleting files
from the destination when they disappear locally. That is exactly the point of
Workflow 2's one-way **mirror** (only the latest results matter, old ones
should be cleaned up). But for a **backup** -- where losing a local file must
never delete the only remaining copy -- it is dangerous:

```bash
# WRONG for a backup! Deletes files from Box if you lose them locally
rclone sync /scratch/work pomona-box:/backup

# RIGHT for a backup: copy only adds and updates, never deletes
rclone copy /scratch/work pomona-box:/backup
```

Rule of thumb: `sync` for mirrors you want pruned, `copy` for backups you
want to keep.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 7.1: Set Up a Mirror Workflow

Create a mirror of a local folder to Box:

```bash
mkdir -p ~/mirror-test
echo "Version 1" > ~/mirror-test/file.txt

# Sync to Box (dry-run first!)
rclone sync ~/mirror-test pomona-box:/mirror-test --dry-run
rclone sync ~/mirror-test pomona-box:/mirror-test
```

Now modify locally and sync again:

```bash
echo "Version 2" > ~/mirror-test/file.txt
rclone sync ~/mirror-test pomona-box:/mirror-test
```

Verify:

```bash
rclone cat pomona-box:/mirror-test/file.txt
```

**Record**: Did the content change to "Version 2"?

::::::::::::::::::::::::::::::::::::: solution

## Solution

After modifying the file and syncing again, verify with:

```bash
rclone cat pomona-box:/mirror-test/file.txt
```

Expected output: `Version 2`

The content changed, confirming that `rclone sync` updated the modified file on Box.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 7.2: Practice Selective Deletion with Sync

Create a scenario where sync would delete files:

```bash
mkdir -p ~/sync-test
echo "Keep this" > ~/sync-test/keep.txt
echo "Delete me" > ~/sync-test/delete.txt
rclone sync ~/sync-test pomona-box:/sync-test
```

Now delete locally and preview the sync:

```bash
rm ~/sync-test/delete.txt
rclone sync ~/sync-test pomona-box:/sync-test --dry-run -vv
```

**Record**: Does the dry-run show that `delete.txt` would be deleted from Box?

::::::::::::::::::::::::::::::::::::: solution

## Solution

The dry-run with `-vv` should show:

```
2026/03/05 16:10:00 INFO  : delete.txt: Deleted (dry run)
```

After executing `rclone sync ~/sync-test pomona-box:/sync-test`, only `keep.txt` remains on Box. This demonstrates that `rclone sync` deletes destination files not present in the source.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- rclone copy is safe for backups (additive only), while rclone sync creates an exact mirror and deletes files not in the source
- Always preview sync operations with --dry-run before executing to avoid accidental data loss
- Use rclone copy for backups and rclone sync only for one-way mirroring where you want the destination to exactly match the source
- Reversing source and destination in rclone sync is a common and dangerous mistake

::::::::::::::::::::::::::::::::::::::::::::::
