---
title: "Setting Up Pomona Box"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I configure rclone for Pomona's Box storage?
- How does OAuth authentication work with Box?
- How do I verify the Box connection?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Configure a new rclone remote for Box using OAuth
- Complete the Box OAuth authorization flow
- Verify the connection with rclone about and rclone lsd

::::::::::::::::::::::::::::::::::::::::::::::

## Configuring rclone for Box

This episode assumes you are connected to Sagehen with port forwarding and have rclone loaded (see Episode 2). Start the configuration:

```bash
rclone config
```

Type `n` to create a new remote.

### Name Your Remote

rclone will ask for a name. This is what you'll use in commands to reference your Box storage:

```
name> pomona-box
```

You can use any name you like (e.g., `my-box`, `box-backup`). We'll use `pomona-box` in examples.

### Select the Storage Type

rclone will list available storage types. Type `box` or the corresponding number:

```
Storage> box
```

### Configure Box Application

rclone asks about Box application credentials. Press Enter for both prompts to use rclone's built-in credentials:

```
client_id> [press Enter]
client_secret> [press Enter]
```

### Select User Type

Choose individual user access (the default):

```
app_type> [press Enter]
```

### Enable Auto-Configuration

Type `y` to allow rclone to open your browser for OAuth:

```
y/n> y
```

## OAuth Authentication (Browser)

::::::::::::::::::::::::::::::::::::: callout

## Understanding the OAuth Authorization Flow

rclone will automatically open your default web browser and direct you to Box's authorization page. If the browser doesn't open automatically, you'll see a URL like:

```
If your browser doesn't open automatically, go to the following link: http://127.0.0.1:53682/auth?state=...
```

Copy and paste this URL into your web browser.

### Box OAuth Flow:

1. **You'll see**: A Box login page
2. **Sign in with**: Your Pomona NetID and password
3. **SSO redirect**: Box will redirect you to Pomona's Single Sign-On (SSO) page
4. **Authorization**: You'll see a screen asking "Rclone wants to access your Box account"
5. **Grant access**: Click the "Grant" or "Authorize" button
6. **Confirmation**: You'll see "The app is now authorized. You can close this window."

### Important: If Two-Factor Authentication is Enabled

If your Pomona account has 2FA enabled (recommended), you'll also be prompted to enter a code from your authenticator app or receive one via email during the OAuth flow.

::::::::::::::::::::::::::::::::::::::::::::::

### Back on Terminal

Once you've granted access in the browser, rclone will automatically receive the token:

```
2026/03/05 14:22:05 NOTICE: Box (pomona-box) Waiting for local client...
2026/03/05 14:22:15 NOTICE: Box authorization successful
```

### Skip Advanced Configuration

Type `n` when asked about advanced config:

```
y/n> n
```

### Answer the Web-Browser Question

rclone then asks one more question the older docs didn't show:

```
Use web browser to automatically authenticate rclone with remote?
 * Say Y if the machine running rclone has a web browser you can use
 * Say N if running rclone on a (remote) machine without web browser access
If not sure try Y. If Y failed, try N.

y) Yes (default)
n) No
y/n>
```

**Answer `n` on Sagehen** — the cluster has no web browser. rclone will then
show instructions to run `rclone authorize "box"` on your own laptop (which
does have a browser) and paste the resulting JSON blob back into the
`config_token>` prompt on Sagehen. Paste the *entire* JSON output, braces
included — a bare token string will fail with a decode error.

![The two prompts in sequence: decline advanced config, then answer `n` to the browser question because Sagehen is a remote machine without one.](fig/03-box-advanced-config-and-browser-prompt.png){alt='Terminal showing the rclone Box setup wizard. The Edit advanced config question has been answered n. rclone then asks whether to use a web browser to automatically authenticate, explaining to say N if rclone runs on a remote machine without browser access. The y/n prompt is waiting for input.'}

### Confirm and Exit

Type `y` to confirm your configuration, then `q` to quit:

```
y/e/d> y
e/n/d/r/c/s/u/m/v/b/q> q
```


## Verify the Connection

Test that rclone can access your Box account:

```bash
rclone about pomona-box:
```

**Expected output:**
```
Total:   1.0T
Used:    25.3G
Free:    974.7G
Trashed: 0B
Other:   0B
```

Also try listing your Box home directory:

```bash
rclone lsd pomona-box:/
```

**Expected output:**
```
          -1 2026-02-15 10:30:00        -1 My Research
          -1 2026-01-20 14:45:00        -1 Shared
```

If you see your Box files listed, setup is successful!

## For VS Code Remote-SSH Users (Cleanup)

If you configured port forwarding in your `.ssh/config` for VS Code, it may show port 53682 in its PORTS tab. After rclone setup is complete, you can delete this port from the PORTS tab to reduce clutter. Your rclone connection will still work.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 3.1: Configure Box Remote

Follow the steps above to set up your own Box remote. Name it `pomona-box` (or your preferred name).

At the end, run:

```bash
rclone about pomona-box:
```

**Record the output**: How much total storage do you have? How much is used?

::::::::::::::::::::::::::::::::::::: solution

## Solution

You should see output similar to:

```
Total:   1.0T
Used:    25.3G
Free:    974.7G
Trashed: 0B
Other:   0B
```

The exact numbers depend on your Box usage. Seeing storage info (no errors) confirms your `pomona-box` remote is configured correctly. If you see `ERROR: access_token_expired`, revisit the OAuth steps above.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 3.2: List Your Box Contents and Create a Test Folder

List the top-level contents of your Box account:

```bash
rclone lsd pomona-box:/
```

Then create a test folder for upcoming exercises:

```bash
rclone mkdir pomona-box:/rclone-test
rclone lsd pomona-box:/
```

You should see your new `rclone-test` folder in the list.

::::::::::::::::::::::::::::::::::::: solution

## Solution

After running `rclone mkdir pomona-box:/rclone-test`, verify with:

```bash
rclone lsd pomona-box:/
```

Expected output includes:

```
          -1 2026-03-05 15:00:00        -1 rclone-test
```

The `mkdir` command produces no output on success; if you see no errors, the folder was created.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- Box uses Pomona's Single Sign-On (SSO) for authentication, which may include two-factor authentication
- rclone configuration stores OAuth tokens securely in ~/.config/rclone/rclone.conf and does not store your password
- Test your configuration with rclone about and rclone lsd commands to verify the connection works
- The rclone remote name (e.g., pomona-box) is your local reference for this Box account and can be customized

::::::::::::::::::::::::::::::::::::::::::::::
