# Pre-Workshop Setup

Before attending the **Cloud Storage Integration with rclone** workshop, please ensure you have completed the following steps. This will help us make the most of our time together.

## Prerequisites

### 1. Active Pomona College HPC Account

You should have an active account on the Sagehen HPC cluster (sagehen.hpc.pomona.edu). If you haven't already done so, you can request an account:

- **Request form (preferred):** [HPC account request](https://servicedesk.pomona.edu/support/catalog/items/83)
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

### 4. Understanding Sagehen HPC Storage

Please familiarize yourself with the different storage locations on Sagehen:

| Location | Quota | Purpose | Retention |
|----------|-------|---------|-----------|
| `/rhome/<myusername>` | 100 GB | Personal, backed up | Permanent |
| `/bigdata/lab/<labname>` | 1 TB | Lab/group shared | Permanent |
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

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
