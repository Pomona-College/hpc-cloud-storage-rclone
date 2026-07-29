# Pre-Workshop Setup

Before attending the **Cloud Storage Integration with rclone** workshop, please ensure you have completed the following steps. This will help us make the most of our time together.

## Prerequisites

### 1. Active Pomona College HPC Account

You should have an active account on the Sagehen cluster (sagehen.hpc.pomona.edu). If you haven't already done so, you can request an account by contacting:

- **Email:** its-hpc@pomona.edu
- **Instructor:** Andrew Wilson

Your account will provide access to:
- `/rhome` (100GB personal storage for your files)
- `/bigdata` (1TB lab storage shared with your research group)
- `/scratch` (SSD temporary storage for large computations)
- `/tmpfs` (RAM-based temporary storage for very fast I/O)

**Verify your access:** Log in to the OnDemand portal at https://ondemand.hpc.pomona.edu/ using your Pomona AD credentials and DUO MFA to confirm your account is active.

### 2. Cloud Storage Account

You must have at least one of the following accounts:
- **Box account** (Pomona College provides Box storage for faculty and staff)
- **OneDrive account** (Microsoft account or institutional)

If you're unsure whether you have Box access, contact its-hpc@pomona.edu or check https://pomona.box.com.

### 3. Terminal Familiarity

You should be comfortable with basic terminal commands such as:
- `ls` (list files and directories)
- `cd` (change directory)
- `pwd` (print working directory)
- `mkdir` (create directories)
- `cat` (view file contents)

If you need a refresher on command-line basics, we recommend reviewing a basic tutorial before the workshop. Don't worry; we'll review essential commands together during the session.

### 4. Understanding Sagehen Storage

Please familiarize yourself with the different storage locations on Sagehen:

| Location | Quota | Purpose | Retention |
|----------|-------|---------|-----------|
| `/rhome/username` | 100 GB | Personal, backed up | Permanent |
| `/bigdata/groupname` | 1 TB | Lab/group shared | Permanent |
| `/scratch/username` | No limit | Temporary compute | 60-day deletion policy |
| `/tmpfs` | RAM-dependent | Very fast temporary | Job duration only |

When syncing with cloud storage, consider which Sagehen location makes sense for your workflow.

## What to Have Ready

- Your Pomona username and password
- Your DUO phone for MFA authentication
- Access to your Box or OneDrive account credentials
- A modern web browser (Chrome, Firefox, Safari, or Edge)
- Approximately 500 MB of free disk space on Sagehen for workshop exercises

## Technical Setup

### Load Required Modules

During the workshop, we'll use rclone and other command-line tools. You can test your environment by logging in to Sagehen and running:

```bash
module available
```

This should show available modules on the system. We'll load the rclone module during the workshop, so don't worry about this in advance.

### Optional: Pre-Install rclone Locally (Advanced)

If you're interested in using rclone on your personal computer, you can install it before the workshop:
- **Windows:** Download from https://rclone.org/install/
- **macOS:** `brew install rclone`
- **Linux:** Use your package manager or download from https://rclone.org/install/

This is optional and not required for the workshop.

## Questions?

If you encounter any issues with your account, cloud storage access, or need clarification on any prerequisites, please reach out:
- **Email:** its-hpc@pomona.edu
- **Instructor:** Andrew Wilson
- **Contact:** Pomona College HPC Team

We look forward to seeing you in the workshop!
