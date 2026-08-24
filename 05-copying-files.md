---
title: "Copying Files"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I copy files from HPC to cloud storage?
- What does the `remote:path` syntax mean?
- How do I preview changes before copying?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand rclone's `remote:path` syntax
- Copy files between local, Box, and OneDrive with `rclone copy`
- Use `--dry-run` to preview operations before executing
- Verify transfers with `rclone check`

::::::::::::::::::::::::::::::::::::::::::::::

## Understanding remote:path Syntax

::::::::::::::::::::::::::::::::::::: callout

## The Foundation: remote:path Notation

The most important concept in rclone is the `remote:path` syntax. When you use an rclone command, you reference cloud storage using:

```
remote_name:path/to/file
```

**remote_name** is the name you gave during configuration (`pomona-box`, `pomona-onedrive`, etc.)
**path/to/file** is the location within that remote

### Examples

```bash
# Box root
pomona-box:/

# A file in Box
pomona-box:/my-results.csv

# A folder in Box
pomona-box:/research/project-2026

# OneDrive
pomona-onedrive:/Documents/thesis.docx
```

::::::::::::::::::::::::::::::::::::::::::::::

## Copying Files: rclone copy

The `rclone copy` command copies files **from a source to a destination**, adding new files to the destination without removing existing ones.

### Syntax

```bash
rclone copy source destination
```

### Local to Cloud Examples

Copy a file from your HPC home directory to Box:

```bash
rclone copy ~/results.csv pomona-box:/rclone-test/
```

Copy a local folder to Box:

```bash
rclone copy ~/my-data pomona-box:/rclone-test/
```

### Cloud to Local Examples

Copy files from Box to your HPC home:

```bash
rclone copy pomona-box:/rclone-test/ ~/downloaded-results/
```

### Cloud to Cloud Examples

Copy from Box to OneDrive:

```bash
rclone copy pomona-box:/rclone-test pomona-onedrive:/backup
```

### Key Points About rclone copy

- **Copies recursively**: Includes all subfolders
- **Additive**: Only adds new files; doesn't delete existing ones
- **Fast**: Only copies files that aren't already at the destination
- **Preserves structure**: Folder hierarchies are maintained

## Dry-Run: Preview Before Copying

::::::::::::::::::::::::::::::::::::: callout

## Safe Testing: Always Use --dry-run First

Before running a copy, use `--dry-run` to preview what will happen:

```bash
rclone copy ~/my-data pomona-box:/backup --dry-run
```

**Expected output:**
```
2026/03/05 15:10:22 INFO  : file1.csv: Copied (dry run)
2026/03/05 15:10:22 INFO  : subfolder/file2.txt: Copied (dry run)
2026/03/05 15:10:22 INFO  : Transferred:   2 files
2026/03/05 15:10:22 INFO  : Total size: 5.3M
```

This shows what would be copied without actually doing it. **Always use `--dry-run` on large operations!**

::::::::::::::::::::::::::::::::::::::::::::::

## Checking File Integrity: rclone check

Verify that files were copied correctly by comparing checksums:

```bash
rclone check ~/my-data pomona-box:/backup
```

**Expected output if all match:**
```
2026/03/05 15:25:00 INFO  : Checks completed successfully
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 5.1: Create Test Data and Copy to Box

Create a local folder with test files, then copy to Box:

```bash
mkdir -p ~/test-data/subfolder
echo "This is file 1" > ~/test-data/file1.txt
echo "This is file 2" > ~/test-data/subfolder/file2.txt
```

Preview first with `--dry-run`:

```bash
rclone copy ~/test-data pomona-box:/rclone-test/challenge5 --dry-run
```

Then execute:

```bash
rclone copy ~/test-data pomona-box:/rclone-test/challenge5
```

Verify:

```bash
rclone tree pomona-box:/rclone-test/challenge5
```

::::::::::::::::::::::::::::::::::::: solution

## Solution

The dry-run should show:

```
2026/03/05 15:10:22 INFO  : file1.txt: Copied (dry run)
2026/03/05 15:10:22 INFO  : subfolder/file2.txt: Copied (dry run)
```

After executing, verify with `rclone tree`:

```
/
├── file1.txt
└── subfolder/
    └── file2.txt
```

The folder structure on Box should match your local `~/test-data` exactly.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 5.2: Copy from Box to OneDrive

Copy your test data from Box to OneDrive:

```bash
rclone copy pomona-box:/rclone-test/challenge5 pomona-onedrive:/rclone-test/box-backup --dry-run
```

Preview first, then execute without `--dry-run`:

```bash
rclone copy pomona-box:/rclone-test/challenge5 pomona-onedrive:/rclone-test/box-backup
```

Verify on OneDrive:

```bash
rclone tree pomona-onedrive:/rclone-test/box-backup
```

::::::::::::::::::::::::::::::::::::: solution

## Solution

The dry-run should show both files being copied from Box to OneDrive. After executing:

```bash
rclone tree pomona-onedrive:/rclone-test/box-backup
```

Expected output:

```
/
├── file1.txt
└── subfolder/
    └── file2.txt
```

This confirms cloud-to-cloud transfer works. The folder structure on OneDrive mirrors what was on Box.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- The remote:path syntax (e.g., pomona-box:/folder/file.txt) is fundamental to all rclone commands
- rclone copy is additive and safe -- it never deletes files at the destination
- The --dry-run flag previews changes without making them and should always be used before large operations
- Understanding the difference between local paths (~/file.txt) and remote paths (remote:path) is critical for avoiding mistakes

::::::::::::::::::::::::::::::::::::::::::::::
