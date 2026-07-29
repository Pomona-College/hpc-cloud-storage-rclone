---
title: "Token Management and Maintenance"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I know when my rclone token has expired?
- What is token expiration and why does it happen?
- How do I refresh an expired token?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand OAuth token lifecycle and expiration
- Check token validity with rclone about
- Refresh expired tokens using `rclone config reconnect`
- Set up reminders for proactive token renewal

::::::::::::::::::::::::::::::::::::::::::::::

## Understanding OAuth Token Expiration

::::::::::::::::::::::::::::::::::::: callout

## OAuth Token Lifecycle and Expiration

### How OAuth Works

When you ran `rclone config` and authenticated via Box or OneDrive, you received an **OAuth token**. This token is like a temporary access card:
1. You authenticated once (via the browser OAuth flow)
2. rclone received a token that proves your identity
3. The token is stored in `~/.config/rclone/rclone.conf`
4. rclone uses this token to access Box/OneDrive without your password

### Why Tokens Expire

OAuth tokens don't last forever. They typically expire every **2-3 months** for security reasons:
- **Limits damage**: If someone steals a token, they can't use it forever
- **Enforces re-authentication**: You must periodically prove your identity again
- **Improves security**: Forces you to use current authentication methods (2FA, password updates, etc.)

::::::::::::::::::::::::::::::::::::::::::::::

### When Your Token Expires

You'll see errors like these:

```
2026/03/10 14:22:15 ERROR : access_token_expired: Access token has expired
2026/03/10 14:22:15 ERROR : invalid_grant: The provided authorization grant is invalid
```

## Checking Your Token Status

The simplest way to check if your token is still valid:

```bash
rclone about pomona-box:
```

If you see storage information, your token is valid. If you see an error, it needs renewal.

Test all your configured remotes:

```bash
rclone about pomona-box:
rclone about pomona-onedrive:
```

## Renewing Expired Tokens

::::::::::::::::::::::::::::::::::::: callout

## Quick Token Renewal Methods

### Method 1: Quick Reconnect (Recommended)

```bash
rclone config reconnect pomona-box:
```

This keeps all existing settings and only refreshes the OAuth token. It launches your browser for re-authentication and takes about 30 seconds.

### Method 2: Full Reconfiguration

If reconnect fails, you can reconfigure from scratch:

```bash
rclone config
```

Choose `e` (Edit existing remote), select your remote, and go through the OAuth flow again.

::::::::::::::::::::::::::::::::::::::::::::::

### Reconnect Multiple Remotes

If all your tokens are expired:

```bash
rclone config reconnect pomona-box:
rclone config reconnect pomona-onedrive:
```

## Important: Port Forwarding for Token Renewal

Just like initial setup, token renewal needs port forwarding:

```bash
ssh username@sagehen.hpc.pomona.edu -L 53682:localhost:53682
```

Then in the SSH session, run reconnect commands.

## Setting Up Token Renewal Reminders

Create a cron job that reminds you to refresh tokens:

```bash
# Add to crontab
0 2 1 */2 * echo "Time to refresh rclone tokens!" | mail -s "Reminder" $USER@pomona.edu
```

This sends a reminder email on the first day of every other month.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 11.1: Check Your Token Status

Verify that both your Box and OneDrive tokens are still valid:

```bash
echo "Checking Box token..."
rclone about pomona-box:

echo "Checking OneDrive token..."
rclone about pomona-onedrive:
```

**Record**: Are both tokens valid? Do you see storage information for both?

::::::::::::::::::::::::::::::::::::: solution

## Solution

For valid tokens, you should see storage info for each:

```
Checking Box token...
Total:   1.0T
Used:    45.2G
Free:    954.8G

Checking OneDrive token...
Total:   1.0T
Used:    8.4G
Free:    991.6G
```

If a token has expired, you will see: `ERROR : error: invalid_grant: The provided authorization grant is invalid`. An expired token is not a problem -- renew it with `rclone config reconnect`.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 11.2: Practice Token Renewal

Practice renewing a token (even if it's not actually expired):

```bash
ssh username@sagehen.hpc.pomona.edu -L 53682:localhost:53682
module load rclone
rclone config reconnect pomona-box:
```

Go through the OAuth flow in your browser. Then verify:

```bash
rclone about pomona-box:
```

**Record**: Did the reconnect succeed?

::::::::::::::::::::::::::::::::::::: solution

## Solution

During the reconnect, you should see:

```
NOTICE: AUTHORIZATION REQUIRED
Go to the following link in your browser: http://localhost:53682/auth?code=...
```

After completing OAuth: `NOTICE: Authorization successful!`

Seeing storage info from `rclone about` confirms the token was refreshed. Remember: you must have SSH port forwarding active for reconnect to work.

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- OAuth tokens expire every 2-3 months and should be refreshed proactively with rclone config reconnect
- Use rclone about remote: to test token validity before running important automated jobs
- Token renewal requires SSH port forwarding (-L 53682:localhost:53682) just like initial setup
- Set calendar or cron reminders to refresh tokens before they expire

::::::::::::::::::::::::::::::::::::::::::::::
