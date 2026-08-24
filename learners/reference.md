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
rclone copy box-storage:/data /rhome/<myusername>/data
```

**Copy from Sagehen to remote:**
```bash
rclone copy /rhome/<myusername>/data box-storage:/backup
```

**Copy single file:**
```bash
rclone copy box-storage:/results.csv /rhome/<myusername>/
```

### Synchronizing Directories

**One-way sync (remote → local):**
```bash
rclone sync box-storage:/data /rhome/<myusername>/data
```

**One-way sync (local → remote):**
```bash
rclone sync /rhome/<myusername>/data box-storage:/backup
```

**Check what would be synced (dry-run):**
```bash
rclone sync --dry-run /rhome/<myusername>/data box-storage:/backup
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
rclone check box-storage:/data /rhome/<myusername>/data
```

**Show differences:**
```bash
rclone check --one-way box-storage:/data /rhome/<myusername>/data
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

- **Personal:** `/rhome/<myusername>` (100 GB, backed up)
- **Lab/Group:** `/bigdata/lab/<labname>` (1 TB shared)
- **Temporary:** `/scratch/username` (no quota, 60-day retention)
- **Fast Temp:** `/tmpfs` (RAM-based, job duration only)

Example sync to shared lab storage:
```bash
rclone sync box-storage:/results /bigdata/lab/<labname>/results
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

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
