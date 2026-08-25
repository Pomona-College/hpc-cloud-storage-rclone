---
title: "Setting Up OneDrive"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I set up rclone to use OneDrive with my Pomona account?
- What's the difference between OneDrive Personal and OneDrive for Business?
- How do I select the correct drive when multiple options are available?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Configure rclone for OneDrive using OAuth
- Understand OneDrive Personal vs OneDrive for Business options
- Verify connectivity and list your drive contents

::::::::::::::::::::::::::::::::::::::::::::::

## When to Choose OneDrive

::::::::::::::::::::::::::::::::::::: callout

## Deciding Between Box and OneDrive

Use OneDrive with rclone if you:

- Want **personal control** over cloud storage
- Prefer **seamless Office 365 integration**
- Have smaller-scale data syncing needs
- Want storage that's accessible from multiple devices
- Don't need to share with other Pomona users (use Box for that)

| Aspect | Box | OneDrive |
|--------|-----|----------|
| **Intended use** | Institutional/Shared | Personal/Individual |
| **Best for** | Collaboration at Pomona | Personal backups |
| **Sharing** | Easy with Pomona users | Better with personal contacts |
| **Data governance** | Pomona manages/audits | You manage |
| **Setup complexity** | Straightforward | Slightly more options |

::::::::::::::::::::::::::::::::::::::::::::::

This episode assumes you already have an SSH session open with port forwarding and rclone loaded. If not, see Episode 2.

## Configure OneDrive

Start a new rclone configuration:

```bash
rclone config
```

Type `n` to create a new remote, then name it:

```
name> pomona-onedrive
```

### Select OneDrive Storage Type

Find and select Microsoft OneDrive:

```
Storage> onedrive
```

### Select Microsoft Cloud

Choose the global Microsoft Cloud (option 1):

```
cloud_type> 1
```

### OAuth Configuration

Type `y` to use auto-configuration:

```
y/n> y
```

### Microsoft OAuth Flow

rclone will open your browser to Microsoft's login page:

1. **Sign in page**: Enter your Pomona College email address
2. **Password page**: Enter your Pomona password
3. **Two-Factor Authentication**: If enabled, authenticate with your 2FA method
4. **Consent page**: You'll see "Rclone wants to access your resources"
5. **Grant access**: Click "Accept" or "Yes"
6. **Confirmation**: You'll see "The app is now authorized. You can close this window."

Back on terminal, you should see:

```
2026/03/05 14:35:22 NOTICE: Microsoft OneDrive authorization successful
```

### Choose Your OneDrive Type

::::::::::::::::::::::::::::::::::::: callout

## OneDrive Options: Personal vs Business

rclone will ask which OneDrive to use:

```
 0 / OneDrive Personal
   \ "onedrive"
 1 / OneDrive for Business
   \ "business"
 2 / Document Library (from SharePoint)
   \ "documentLibrary"

onedrive_type>
```

- **Type `0`** (OneDrive Personal): Personal OneDrive (typically smaller quota)
- **Type `1`** (OneDrive for Business): Business account with larger quota
- **Type `2`** (Document Library): SharePoint site documents

For most Pomona students, choose **0** (OneDrive Personal).

::::::::::::::::::::::::::::::::::::::::::::::

### Select Your Drive and Confirm

If you have multiple drives, select the appropriate number (usually `0`). Then skip advanced configuration (`n`), confirm with `y`, and quit with `q`.

## Verify the Connection

Test that rclone can access your OneDrive:

```bash
rclone about pomona-onedrive:
```

**Expected output:**
```
Total:   1.0T
Used:    8.4G
Free:    991.6G
Trashed: 0B
Other:   0B
```

List your OneDrive root directory:

```bash
rclone lsd pomona-onedrive:/
```

## Now You Have Two Remotes

If you set up both Box and OneDrive, verify both remotes are available:

```bash
rclone config show
```

You now have:

- `pomona-box:` -- Your institutional Box storage
- `pomona-onedrive:` -- Your Pomona institutional OneDrive for Business storage (M365)

You can use rclone commands with either remote, and even transfer files between them!

## OneDrive Storage Paths

When using OneDrive with rclone, remember these path conventions:

```bash
# Root of your OneDrive
rclone ls pomona-onedrive:/

# A specific folder
rclone ls pomona-onedrive:/Documents

# A nested folder
rclone ls pomona-onedrive:/Documents/Research
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 4.1: Configure OneDrive Remote

Follow the steps above to set up your OneDrive remote. Name it `pomona-onedrive`.

At the end, run:

```bash
rclone about pomona-onedrive:
```

**Record the output**: How much storage do you have on OneDrive?


::::::::::::::::::::::::::::::::::::: solution

## Solution

You should see output similar to:

```
Total:   1.0T
Used:    8.4G
Free:    991.6G
Trashed: 0B
Other:   0B
```

Seeing storage information (no errors) confirms your `pomona-onedrive` remote is configured correctly.

::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 4.2: Compare Your Remotes

List the contents of both your Box and OneDrive:

```bash
echo "=== Box Contents ==="
rclone lsd pomona-box:/

echo "=== OneDrive Contents ==="
rclone lsd pomona-onedrive:/
```

**Compare**: Which remote has more folders? Which do you think you'll use more frequently?


::::::::::::::::::::::::::::::::::::: solution

## Solution

OneDrive typically has more default folders (Documents, Downloads, Pictures). Box may have fewer folders if it is new. Use Box for institutional collaboration and OneDrive for personal storage.

::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- Pomona's institutional OneDrive for Business is the supported endpoint.
- The Microsoft OAuth flow requires signing in with your Pomona email and handling two-factor authentication
- You can configure multiple cloud remotes with rclone (Box and OneDrive) and reference them with their unique names
- Test your OneDrive configuration with rclone about and rclone lsd commands just like Box

::::::::::::::::::::::::::::::::::::::::::::::
