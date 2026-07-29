---
title: "Listing and Inspecting"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I list files and directories in cloud storage?
- How can I visualize directory structure?
- How do I read file contents, delete files, and manage directories?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Use `rclone ls` and `rclone lsd` to list files and folders
- Visualize directory structure with `rclone tree`
- Create and delete directories with `rclone mkdir`, `rclone delete`, and `rclone purge`
- Read file contents with `rclone cat`

::::::::::::::::::::::::::::::::::::::::::::::

## Listing Contents: rclone ls

The `rclone ls` command lists files in a directory (not folders).

```bash
rclone ls pomona-box:/
```

**Expected output:**
```
    12345 file.txt
    67890 presentation.pptx
```

This shows file size (in bytes) and filename. Useful options:

```bash
# Count total files
rclone ls pomona-box:/ | wc -l

# Human-readable sizes
rclone ls pomona-box:/ --human-readable
```

## Listing Directories: rclone lsd

The `rclone lsd` command lists **only** directories (folders), not files.

```bash
rclone lsd pomona-box:/
```

**Expected output:**
```
          -1 2026-02-15 10:30:00        -1 My Research
          -1 2026-01-20 14:45:00        -1 Project Data
```

## Visualizing Structure: rclone tree

The `rclone tree` command shows the directory structure graphically:

```bash
rclone tree pomona-box:/rclone-test
```

**Expected output:**
```
/
├── file1.txt
├── file2.csv
└── subfolder/
    ├── nested-file.txt
    └── data.json
```

Limit depth with `--max-depth`:

```bash
rclone tree pomona-box:/ --max-depth 2
```

## Creating Directories: rclone mkdir

```bash
rclone mkdir pomona-box:/my-new-folder
rclone mkdir pomona-box:/rclone-test/subfolder
```

Create multiple levels one at a time:

```bash
rclone mkdir pomona-box:/level1
rclone mkdir pomona-box:/level1/level2
rclone mkdir pomona-box:/level1/level2/level3
```

## Reading File Contents: rclone cat

Display the contents of a file in cloud storage without downloading it:

```bash
rclone cat pomona-box:/rclone-test/notes.txt
```

Preview a CSV file before downloading:

```bash
rclone cat pomona-box:/results.csv | head -5
```

## Deleting Files and Folders

### rclone delete

Removes files but not the folder itself. **Use with caution: this is irreversible!**

```bash
# Delete all files in a folder
rclone delete pomona-box:/rclone-test

# Delete files matching a pattern
rclone delete pomona-box:/rclone-test --include "*.tmp"
```

### rclone purge

Deletes a folder **and all contents**:

```bash
rclone purge pomona-box:/rclone-test
```

### Always Use --dry-run First!

```bash
rclone delete pomona-box:/rclone-test --dry-run
rclone purge pomona-box:/rclone-test --dry-run
```

## Quick Reference: Common Commands

| Command | Purpose | Syntax |
|---------|---------|--------|
| `ls` | List files | `rclone ls remote:path` |
| `lsd` | List directories | `rclone lsd remote:path` |
| `tree` | Show folder structure | `rclone tree remote:path` |
| `mkdir` | Create folder | `rclone mkdir remote:path` |
| `cat` | Display file content | `rclone cat remote:path` |
| `delete` | Delete files | `rclone delete remote:path` |
| `purge` | Delete folder + contents | `rclone purge remote:path` |
| `check` | Verify integrity | `rclone check source dest` |
| `size` | Show total size | `rclone size remote:path` |

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 6.1: List and Inspect

List files in your test folder using different commands:

```bash
rclone ls pomona-box:/rclone-test
rclone lsd pomona-box:/rclone-test
rclone tree pomona-box:/rclone-test --max-depth 2
```

**Compare**: What does each command show differently?

::::::::::::::::::::::::::::::::::::: solution

## Solution

Each command shows different information:

`rclone ls` -- lists **files only** with sizes:
```
       15 file1.txt
       15 subfolder/file2.txt
```

`rclone lsd` -- lists **directories only**:
```
          -1 2026-03-05 15:10:00        -1 challenge5
```

`rclone tree --max-depth 2` -- shows the **hierarchical structure** of both files and folders:
```
/
└── challenge5/
    ├── file1.txt
    └── subfolder/
```

Use `ls` to find specific files, `lsd` to see top-level folders, and `tree` to visualize the full layout.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 6.2: Delete Safely

Clean up test files using `--dry-run` first:

```bash
rclone delete pomona-onedrive:/rclone-test/box-backup --dry-run
```

Verify the dry-run output shows exactly what you want to delete. Then delete:

```bash
rclone delete pomona-onedrive:/rclone-test/box-backup
```

Confirm files are gone:

```bash
rclone ls pomona-onedrive:/rclone-test/box-backup
```

**Note**: The folder itself remains (empty). To remove the empty folder too, use `rclone purge`.

::::::::::::::::::::::::::::::::::::: solution

## Solution

The dry-run should show files that would be deleted:

```
2026/03/05 15:20:15 INFO  : file1.txt: Deleted (dry run)
2026/03/05 15:20:15 INFO  : subfolder/file2.txt: Deleted (dry run)
```

After executing `rclone delete`, `rclone ls pomona-onedrive:/rclone-test/box-backup` should return no output, meaning no files remain. To remove the empty folder entirely, use `rclone purge pomona-onedrive:/rclone-test/box-backup`.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- rclone ls lists files, rclone lsd lists directories, and rclone tree shows the complete folder structure
- rclone cat displays file contents without downloading, useful for previewing data
- rclone delete removes files (not folders) while rclone purge removes folders and all contents
- Always preview destructive operations with --dry-run before executing them

::::::::::::::::::::::::::::::::::::::::::::::
