# Cloud Storage Integration with rclone - Workshop Repository

A complete Carpentries Workbench workshop teaching researchers how to integrate cloud storage (Box and OneDrive) with high-performance computing systems using rclone.

## Workshop Overview

**Title**: Cloud Storage Integration with rclone  
**Carpentry**: Incubator  
**Level**: Intermediate (requires basic Linux/SSH knowledge)  
**Duration**: 2.5 hours (6 episodes)  
**Contact**: its-hpc@pomona.edu  

## What You'll Learn

By the end of this workshop, you will be able to:

- Set up rclone for Pomona College's Box (institutional) and OneDrive (personal) storage
- Execute basic rclone operations: list, copy, delete, and organize files
- Implement syncing workflows to keep HPC results and cloud storage synchronized
- Automate transfers with cron jobs and SLURM job scripts
- Maintain your rclone setup, including token renewal

## Quick Navigation

| Episode | Topic | Duration | Type |
|---------|-------|----------|------|
| [01](episodes/01-why-cloud-storage.md) | Why Cloud Storage Integration Matters | 20 min | Teaching + Challenges |
| [02](episodes/02-rclone-setup-box.md) | Setting Up rclone with Box at Pomona | 30 min | Demo + Hands-on |
| [03](episodes/03-rclone-setup-onedrive.md) | Setting Up rclone with OneDrive at Pomona | 30 min | Demo + Hands-on |
| [04](episodes/04-basic-operations.md) | Basic rclone Operations and Commands | 35 min | Demo + Practice |
| [05](episodes/05-syncing-workflows.md) | Syncing Workflows: Copy vs Sync | 45 min | Concepts + Workflows |
| [06](episodes/06-automation-maintenance.md) | Automation and Token Maintenance | 35 min | Advanced Topics |

## For Learners

### Getting Started

1. **Read [the introduction](index.md)** to understand the workshop goals
2. **Start with [Episode 1](episodes/01-why-cloud-storage.md)** for context
3. **Follow Episodes 2-3** to set up your cloud storage
4. **Progress through Episodes 4-6** at your own pace

### Prerequisites

- Pomona College NetID with HPC access
- Basic comfort with command-line tools and SSH
- A web browser (for OAuth authentication)
- Approximately 2-3 hours

### Key Resources

- **[Quick Reference Card](reference.md)** :  Command cheatsheet for quick lookup
- **[Troubleshooting Guide](instructor-notes.md#troubleshooting-guide-for-instructors)** :  Common issues and fixes
- **[Links to Resources](links.md)** :  Additional rclone and HPC documentation

## For Instructors

### Teaching the Workshop

1. **Review [Setup Instructions](setup.md)** (recommended 1 week before)
2. **Study [Instructor Notes](instructor-notes.md)** for teaching strategies
3. **Test all demos beforehand** using your own Box/OneDrive accounts
4. **Have [Learner Profiles](learner-profiles.md)** in mind for your audience

### Workshop Materials

- **Full teaching notes**: [instructor-notes.md](instructor-notes.md)
- **Setup checklist**: [setup.md](setup.md)
- **Quick reference for learners**: [reference.md](reference.md)
- **Learner profiles**: [learner-profiles.md](learner-profiles.md)

### Key Teaching Points

- Episode 1: "rclone solves real HPC problems"
- Episode 2-3: OAuth setup is simple and secure
- Episode 4: `remote:path` syntax is the core mental model
- Episode 5: **Copy vs sync distinction is critical** (copy = safe, sync = mirror)
- Episode 6: Token renewal is routine maintenance

## Workshop Structure

### Episodes

Each episode consists of:

1. **Objectives** :  What you'll learn
2. **Concepts** :  Key ideas with examples
3. **Hands-on Challenges** :  Practice problems with solutions
4. **Takeaways** :  Summary of learning

### Types of Content

- **Teaching Sections** :  Explanation and conceptual material
- **Code Blocks** :  Actual commands to type and run
- **Challenges** :  Hands-on practice with guided solutions
- **Callout Boxes** :  Important tips, warnings, and key points

## Pomona-Specific Features

This workshop is tailored for Pomona College researchers:

- **Box at Pomona**: Institutional cloud storage with SSO
- **OneDrive**: Personal cloud storage via Microsoft 365
- **Sagehen HPC Cluster**: Specific to Pomona's HPC infrastructure
- **SSH Port Forwarding**: Required for OAuth on HPC
- **Contact**: its-hpc@pomona.edu for Pomona-specific questions

## Key Concepts

### rclone

Command-line tool for syncing files with cloud storage providers (Box, OneDrive, Google Drive, etc.).

### remote:path Syntax

All rclone commands reference cloud storage using: `remote_name:path/to/file`

Examples:
```
pomona-box:/my-folder
pomona-onedrive:/Documents
/scratch/local-folder  (local path, no remote)
```

### Copy vs Sync

- **`rclone copy`** :  Adds files to destination, never deletes (safe for backups)
- **`rclone sync`** :  Makes destination an exact mirror of source (can delete!)

**Golden Rule**: Always use `--dry-run` before destructive sync operations!

### OAuth Tokens

Temporary credentials issued by Box/OneDrive. They:
- Expire every 2-3 months
- Can be renewed with `rclone config reconnect remote:`
- Never expire your password stored: only the token
- Are secure and Pomona-approved

## File Structure

```
cloud-storage-rclone/
├── config.yaml                      # Carpentries Workbench configuration
├── index.md                         # Workshop introduction
├── README.md                        # This file
├── reference.md                     # Quick command reference
├── setup.md                         # Pre-workshop setup guide
├── instructor-notes.md              # Detailed teaching notes
├── learner-profiles.md              # Target audience descriptions
├── links.md                         # Additional resources and links
└── episodes/
    ├── 01-why-cloud-storage.md      # Episode 1: Why Cloud Storage?
    ├── 02-rclone-setup-box.md       # Episode 2: Box Setup
    ├── 03-rclone-setup-onedrive.md  # Episode 3: OneDrive Setup
    ├── 04-basic-operations.md       # Episode 4: Basic Commands
    ├── 05-syncing-workflows.md      # Episode 5: Sync Strategies
    └── 06-automation-maintenance.md # Episode 6: Automation
```

## Technical Details

### rclone Information

- **Latest version**: v1.65.2+ (as of March 2026)
- **Available on Sagehen**: `module load rclone`
- **Installation**: Pre-installed; no local installation needed
- **Documentation**: https://rclone.org/

### Cloud Storage

- **Box**: Pomona's institutional cloud storage, managed by IT
- **OneDrive**: Personal Microsoft 365 storage
- Both support OAuth authentication (no passwords shared)
- Both accessible via rclone

### HPC System

- **Cluster**: Sagehen at Pomona College
- **Job Scheduler**: SLURM
- **Module System**: Lmod
- **SSH Access**: Required for all HPC work

## Learning Path

### Beginner (Just starting)
1. Read Episode 1
2. Follow Episode 2 or 3 (Box or OneDrive setup)
3. Follow Episode 4 (basic commands)
4. Try Episode 5 (simple workflows)

### Intermediate (Experienced with HPC)
1. Skim Episode 1
2. Follow Episodes 2-3 (setup both if interested)
3. Focus on Episodes 4-5 (commands and workflows)
4. Explore Episode 6 (automation)

### Advanced (Ready for production)
1. Review reference.md for command syntax
2. Jump to Episode 5 (workflows)
3. Focus on Episode 6 (automation and scheduling)
4. Implement custom scripts and cron jobs

## Common Use Cases

### Research Data Backup
"I run simulations on Sagehen that produce 50 GB of results. I want to back them up to Box automatically."

**Solution**: Use `rclone copy` in a cron job to backup nightly.

### Collaboration
"I need to share research results with my advisor who's off-campus."

**Solution**: Use `rclone copy` to upload results to Box, share Box link.

### Dataset Distribution
"My lab has a 100 GB dataset. I want to distribute it to students on the HPC cluster."

**Solution**: Upload to Box, have students `rclone sync` to HPC.

### Off-Campus Access
"I need to access HPC results from my laptop at home."

**Solution**: `rclone sync` results to OneDrive, access from anywhere.

### Automation
"After each job, I want results automatically uploaded to the cloud."

**Solution**: Add `rclone copy` to SLURM job script post-processing.

## Getting Help

### For Pomona-Specific Issues

**Email**: its-hpc@pomona.edu  
**Contact**: Andrew Wilson (course designer)

### For rclone Questions

- **Official Documentation**: https://rclone.org/
- **Forum**: https://forum.rclone.org/
- **GitHub Issues**: https://github.com/rclone/rclone/issues

### For HPC Issues

- **Pomona IT Service Desk**: its-hpc@pomona.edu
- **Sagehen Documentation**: (internal Pomona link)

## Version Information

- **Workshop Created**: March 5, 2026
- **Last Updated**: March 5, 2026
- **rclone Target Version**: 1.65.2+
- **Carpentries Version**: Workbench (latest)

## License

This workshop is licensed under Creative Commons Attribution 4.0 (CC-BY 4.0).

You are free to:
- Share and adapt this material
- Use it for teaching
- Modify and distribute modified versions

**Conditions**:
- Provide attribution to Pomona College HPC and course designers
- Retain the license for any adaptations

See [LICENSE](LICENSE) for full details.

## Contributing and Feedback

Found an issue or have feedback? Please:

1. **Check existing issues**: GitHub Issues (if available)
2. **Email instructors**: its-hpc@pomona.edu
3. **Provide specific feedback**: What works? What's confusing?

Your feedback helps improve the workshop for future learners!

## Citation

If you use this workshop or adapt it, please cite:

> Pomona College HPC Training. (2026). Cloud Storage Integration with rclone. Workshop 20. Pomona College. Retrieved from [workshop repository URL]

## Acknowledgments

- **Carpentries**: For the Workbench framework
- **rclone developers**: For the powerful tool
- **Pomona IT**: For Box support and HPC infrastructure
- **Andrew Wilson**: Course designer and HPC support specialist

---

Ready to get started? Begin with [the introduction](index.md) or jump straight to [Episode 1](episodes/01-why-cloud-storage.md)!
