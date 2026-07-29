# Learner Profiles: Cloud Storage Integration with rclone

These learner profiles represent typical participants in the Cloud Storage Integration with rclone workshop. Understanding their goals, experience levels, and pain points helps tailor examples and explanations.

## Profile 1: Dr. Sarah Chen, Ecology Researcher

**Background:** Tenure-track faculty in Environmental Sciences, 10 years experience

**HPC Experience:** Moderate
- Uses Sagehen for statistical modeling with R
- Familiar with command line but not expert
- Primarily uses cluster for computational work, not data transfer

**Pain Points:**
- Collects field data from multiple field sites via Box folders
- Currently downloads data manually, then manually uploads results back
- Wants automated, reproducible workflow from data collection to analysis to publication
- Field laptop sometimes has poor connectivity; wants to sync when back at office
- Lab shares results with collaborators using OneDrive

**What She'll Learn:**
- Syncing field data from Box into Sagehen for processing
- Uploading results to shared OneDrive folder for collaborators
- Using dry-run to prevent accidental file loss in shared folders
- Scripting rclone commands within SLURM job scripts for reproducibility

**Example Workflow:**
```bash
# Weekly sync of new field data from Box
rclone sync --progress box-storage:/Field_Data_2026 /rhome/schen/ecology_project/raw_data

# Process in Sagehen (R script or SLURM job)
# ...analysis happens...

# Sync results back to OneDrive for lab review
rclone sync /rhome/schen/ecology_project/results onedrive-storage:/Lab_Results/2026_Field_Season
```

---

## Profile 2: Marcus Thompson, Lab Manager

**Background:** Research operations manager for Biochemistry lab, 6 years in role

**HPC Experience:** Beginner
- Recently assigned to manage lab's computational resources
- Comfortable with spreadsheets and file management
- Never used command line before this course
- Responsible for managing 15 researchers' data and backups

**Pain Points:**
- Lab stores important raw data and backups in Box
- Difficult to coordinate who has what version of files
- Some researchers download outdated versions, then run with wrong data
- Wants reliable backup system to prevent data loss
- Needs simple documentation for lab members to follow

**What He'll Learn:**
- Setting up rclone for backup workflows (safe copy, not sync)
- Understanding Sagehen storage structure and quotas
- Using monitoring/logs to track data transfers
- Creating simple backup scripts for lab documentation
- How to guide other lab members

**Example Workflow:**
```bash
# Daily backup of all lab outputs from Sagehen to Box
# (Put in crontab for automated runs)
rclone copy --progress /bigdata/thompson_lab/outputs box-storage:/Lab_Backups/Daily_Backup_$(date +%Y%m%d)

# Check backup status
rclone check /bigdata/thompson_lab/outputs box-storage:/Lab_Backups/Latest
```

---

## Profile 3: Akari Nakamura, Graduate Student

**Background:** Second-year PhD student in Physics, first HPC user

**HPC Experience:** Low-to-Moderate
- Just learning HPC environment through advisor's recommendation
- Comfortable with terminal after taking HPC basics workshop
- Has large dataset in Google Drive from previous institute
- Needs to get data onto Sagehen to start analyzing

**Pain Points:**
- Original data in Google Drive (not currently supported by rclone config, but has Box copy)
- Overwhelmed by many storage options on Sagehen (rhome, bigdata, scratch, tmpfs)
- Worried about deleting files accidentally
- Wants to keep backup of results during analysis
- Concerned about storage quotas

**What She'll Learn:**
- Safely copying from cloud storage to Sagehen without fear of deletion
- Choosing appropriate storage tier (personal vs shared vs temporary)
- Using dry-run extensively before any sync operations
- Setting up automated backups as she works
- Understanding quota limits

**Example Workflow:**
```bash
# Import initial dataset (carefully!)
# First: list to understand structure
rclone lsd box-storage:/PhD_Data

# Second: dry-run to verify what will copy
rclone copy --dry-run --progress box-storage:/PhD_Data /rhome/anakamura/research

# Third: actually copy with verification
rclone copy --progress box-storage:/PhD_Data /rhome/anakamura/research
rclone check box-storage:/PhD_Data /rhome/anakamura/research

# During analysis: daily backup to Box
rclone sync --progress /rhome/anakamura/research/current_analysis box-storage:/PhD_Analysis_Backups
```

---

## Profile 4: Professor James Mitchell, Data Science & Business

**Background:** Faculty teaching data science, 3 years at Pomona, industry background

**HPC Experience:** Moderate
- Uses clusters regularly for teaching demos
- Comfortable with command line and scripting
- Wants to teach students reproducible data practices
- Integrates HPC examples into curriculum

**Pain Points:**
- Shares datasets with students via OneDrive
- Students struggle with large file downloads
- Wants to mount cloud storage for interactive exploration
- Needs to manage access permissions for student labs
- Teaching deadline pressure; needs quick setup

**What He'll Learn:**
- Configuring rclone remotes for classroom use
- Using mount feature for interactive data exploration with students
- Scripting rclone in learning materials
- Bandwidth limiting to prevent hogging resources
- Setting up examples for reproducibility teaching

**Example Workflow:**
```bash
# Mount class dataset for interactive exploration
mkdir ~/class-datasets
rclone mount onedrive-storage:/Teaching_Data/DS101_Spring2026 ~/class-datasets &

# Students can explore with any tool
ls ~/class-datasets
head -100 ~/class-datasets/housing.csv
# Run Python analysis on mounted files...

# Later: collect results from student submissions
rclone copy ~/class-results/student-outputs onedrive-storage:/Class_Submissions/ds101-spring-2026-results
```

---

## Profile 5: Priya Desai, Undergraduate Researcher

**Background:** Senior majoring in Computer Science, REU participant

**HPC Experience:** Beginner-to-Moderate
- Uses Sagehen for first time through REU program
- Tech-savvy, quick learner
- Wants to learn HPC best practices early in career
- Plans to go to grad school, values reproducibility

**Pain Points:**
- Collaborating with peers using shared Box folder
- Nervous about accidentally breaking shared data
- Wants to set up good practices for grad school
- Limited storage on personal machine
- Wants to understand what's actually happening under the hood

**What She'll Learn:**
- Detailed understanding of rclone mechanics
- How OAuth tokens work (security + trust)
- Writing efficient scripts for automation
- Monitoring and verification of transfers
- Setting up a personal workflow best practices

**Example Workflow:**
```bash
# REU project: process shared dataset, produce results
# First: sync team's latest dataset
rclone sync --progress box-storage:/REU_2026/team_data /scratch/prdesai/team_data

# Work on analysis in /scratch (fast, temporary)
# ...produce results...

# Verify results before uploading
rclone check /scratch/prdesai/results box-storage:/REU_2026/backup_drafts

# Sync final results to shared folder (with dry-run first!)
rclone sync --dry-run /scratch/prdesai/results box-storage:/REU_2026/priya_results
rclone sync --progress /scratch/prdesai/results box-storage:/REU_2026/priya_results

# Archive results to personal backup
rclone copy /scratch/prdesai/results onedrive-storage:/My_Backups/REU_2026
```

---

## Teaching Considerations by Profile

### For Dr. Chen (Researcher):
- Lead with reproducibility angle
- Show how to embed rclone in SLURM scripts
- Discuss publication supplementary materials workflow
- Emphasize --dry-run for shared folders

### For Marcus (Lab Manager):
- Use simple, concrete examples
- Provide copy-paste command templates
- Focus on backup reliability
- Discuss documentation/knowledge sharing with lab

### For Akari (PhD Student):
- Emphasize safety and verification (dry-run, check)
- Address quota concerns with storage location guidance
- Show incremental workflow (copy first, sync later)
- Explain each flag clearly

### For Professor Mitchell (Educator):
- Discuss teaching integration possibilities
- Show how to script rclone in assignments
- Address classroom file distribution logistics
- Highlight reproducibility messaging

### For Priya (Undergrad):
- Encourage deeper learning ("why" not just "how")
- Discuss security model (OAuth, tokens)
- Show how to automate and optimize
- Introduce scripting and bash best practices

---

## Diversity of Use Cases

This workshop serves remarkably diverse learners:

| Aspect | Researcher | Manager | PhD Student | Educator | Undergrad |
|--------|-----------|---------|-------------|----------|-----------|
| Primary Goal | Automation | Reliability | Data Access | Teaching | Learning |
| Risk Tolerance | Medium | Low | Medium | Medium | High |
| Script Comfort | Medium | Low | Low-Med | High | High |
| Typical Transfer Size | Large (GB) | Mixed | Large (GB) | Small-Med | Medium |
| Frequency | Regular | Daily | Frequent | Occasional | During lab |
| Key Feature | Sync/Reproducibility | Copy/Backup | Copy/Safety | Mount | All basics |

Teaching to this mix requires:
- **Clear basics** that work for everyone
- **Practical examples** across different domains
- **Safety emphasis** (dry-run, verification)
- **Scalability** (works for 1 MB and 1 TB)
- **Flexibility** (different tools: copy, sync, mount)

---

## Common Concerns Across All Profiles

### 1. "Will I accidentally delete something?"
**Answer:** With `copy`, nothing is deleted. With `sync`, always use `--dry-run` first.

### 2. "How long will this take?"
**Answer:** Show speed expectations:
- Small files (< 100 MB): seconds to minutes
- Medium (100 MB - 1 GB): minutes
- Large (> 1 GB): depends on network/bandwidth limiting
- Use `--transfers N` to adjust speed vs. resources

### 3. "Is my data secure?"
**Answer:** 
- OAuth means Pomona never handles your password
- Data transfers use HTTPS encryption
- Config file is readable only by you
- Tokens refresh automatically

### 4. "What if something goes wrong?"
**Answer:**
- Transfers can be re-run; rclone remembers progress
- Use `rclone check` to verify after transfer
- Review logs: `rclone lsd source` to see what's there
- Contact: its-hpc@pomona.edu for help

### 5. "Which storage should I use?"
**Answer:** Quick decision tree:
- **Personal research files** → `/rhome` (backed up, 100 GB)
- **Lab/group shared** → `/bigdata` (lab quota, permanent)
- **Large temporary data** → `/scratch` (fast SSD, 60-day cleanup)
- **Very fast temporary** → `/tmpfs` (RAM, job duration only)
