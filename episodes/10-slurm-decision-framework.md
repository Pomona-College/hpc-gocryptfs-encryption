---
title: "SLURM Integration: Decision Framework"
teaching: 15
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions
- How do you decide between pre-mounting and mount-in-script approaches?
- What are the security and performance tradeoffs?
- How do you handle passwords securely in job scripts?
- What password handling options are available?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand decision framework for pre-mount vs mount-in-script
- Implement secure password handling in job scripts
- Apply security best practices for password files
- Choose appropriate password strategy based on workflow type
::::::::::::::::::::::::::::::::::::::::::::::::

## Two Approaches: Decision Framework

When working with encrypted data in SLURM jobs, you have two main strategies. The choice depends on your workflow type, job duration, and security requirements.

### Approach 1: Pre-mount in Terminal (Best for Interactive & Short Jobs)

Mount your encrypted directory in an interactive session before submitting jobs:

```bash
gocryptfs /bigdata/group/secret_cipher /bigdata/group/secret_plain
# Password: [enter password interactively]
# Keep terminal open or background with nohup
```

Then submit your job:

```bash
sbatch analysis.sh
```

Job script accesses the mounted plain directory normally:

```bash
#!/bin/bash
#SBATCH --job-name=quick_analysis
#SBATCH --time=00:30:00

python3 analysis.py /bigdata/group/secret_plain/data.csv
```

**When to use pre-mount:**
- Interactive analysis where you need frequent access
- Short jobs (< 1 hour)
- Debugging encrypted data workflows
- Ad-hoc analysis where job script stability is less critical
- When the same encryption mount serves multiple jobs

**Pros**:
- Simple job script (no password exposure)
- Flexible: can access decrypted data across multiple job submissions
- Good for exploratory work

**Cons**:
- Manual mounting step before job submission
- Must keep terminal/session open or use nohup
- If connection drops, mount may be lost
- Not reproducible: depends on external state

### Approach 2: Mount in Job Script (Best for Production & Overnight Jobs)

Mount encrypted directories within the job script itself. This is self-contained and reproducible:

```bash
#!/bin/bash
#SBATCH --job-name=secure_analysis
#SBATCH --time=02:00:00

CIPHER=/bigdata/group/secret_cipher
PLAIN=/bigdata/group/secret_plain

# Create mount point
mkdir -p $PLAIN

# Mount encrypted directory
echo "password" | gocryptfs $CIPHER $PLAIN -

# Do work
python3 analysis.py $PLAIN/data.csv

# Unmount
fusermount -u $PLAIN

# Cleanup
rmdir $PLAIN
```

**When to use mount-in-script:**
- Production runs that must be reproducible
- Overnight/batch jobs
- Long-running jobs (4+ hours)
- When you need guaranteed cleanup
- Job arrays where each task mounts independently
- Pipelines where encryption state must be part of the job

**Pros**:
- Self-contained: everything needed is in the script
- Reproducible: re-running the script produces identical results
- Guaranteed cleanup: no orphaned mounts left behind
- Proper error handling: can detect mount failures

**Cons**:
- Password handling is more complex (security concern if not done right)
- Slight overhead to mount/unmount per job
- Not ideal for interactive debugging

::::::::::::::::::::::::::::::::::::: callout
## Decision Quick Reference

| Workflow | Approach | Why |
|----------|----------|-----|
| Quick data exploration | Pre-mount | Less overhead, easier debugging |
| 5-minute debugging job | Pre-mount | Not worth mount/unmount time |
| Production ML training | Mount-in-script | Reproducible, proper cleanup |
| Overnight batch processing | Mount-in-script | Guaranteed cleanup on completion |
| Interactive analysis, hourly jobs | Pre-mount | Keep decryption context active |
| Job arrays (100 tasks) | Mount-in-script | Each task mounts independently |
| Continuous monitoring | Pre-mount | Once mounted, many jobs use it |
| Archival/compliance pipeline | Mount-in-script | Full audit trail of what accessed what |

::::::::::::::::::::::::::::::::::::::::::::::::

## Secure Password Handling in Job Scripts

If you mount encrypted data in your job script, you must handle the password securely. There is no perfectly safe way to put a password in a job script, but some approaches are much better than others.

### Password Security Comparison Table

| Method | Security | Recovery | Complexity | Recommendation |
|--------|----------|----------|-----------|-----------------|
| **Hardcoded in script** | NEVER | N/A | Simple | ❌ NEVER DO THIS |
| **Environment variable** | Acceptable | High (keep export command) | Medium | ✓ Use for testing |
| **Password file (chmod 600)** | Recommended | High (keep file backed up) | Medium | ✓ Recommended |
| **SSH agent / key derivation** | Excellent | Expert only | Complex | ✓ Advanced only |

#### Option 1: Hardcoded Password - NEVER DO THIS

```bash
#!/bin/bash
# WRONG - password visible in job script, git history, job output, admin logs
echo "MySecretPassword123" | gocryptfs $CIPHER $PLAIN -
```

**Risks:**
- Visible in your submitted script file
- Visible to SLURM job history (admins can see it)
- If you commit job script to git, password in git history forever
- Visible to anyone with read access to your home directory
- Defeats the purpose of encryption

#### Option 2: Environment Variable - Acceptable for Testing

Set password in interactive session before submitting:

```bash
$ export GOCRYPTFS_PW="securepassword1234"
$ sbatch --export=GOCRYPTFS_PW job.sh
```

In your job script:

```bash
#!/bin/bash
#SBATCH --job-name=ml_training

echo "$GOCRYPTFS_PW" | gocryptfs $CIPHER $PLAIN -
```

**Pros:**
- Password not in script file
- Can be different for different job submissions
- Useful for testing

**Cons:**
- Still visible to other users via `ps` command briefly during execution
- Not suitable for automated/scheduled jobs
- Requires manual setup before submission

#### Option 3: Password File with Restrictive Permissions - RECOMMENDED

Create a password file and protect it:

```bash
# Create password file in your home directory
$ echo "securepassword1234" > ~/.gocryptfs_pw
$ chmod 600 ~/.gocryptfs_pw
$ ls -la ~/.gocryptfs_pw
# Output: -rw------- ... ~/.gocryptfs_pw
```

In your job script:

```bash
#!/bin/bash
#SBATCH --job-name=ml_training

PASSWORD=$(cat ~/.gocryptfs_pw)
echo "$PASSWORD" | gocryptfs $CIPHER $PLAIN -
```

**Why this is recommended:**
- Password not in script file itself
- `chmod 600` means only you can read it
- File can be backed up securely
- Works with scheduled jobs (cron, etc.)

**Security best practices with password files:**
- Store `~/.gocryptfs_pw` on `/rhome` (backed up regularly)
- NEVER commit to git
- NEVER copy to shared systems
- NEVER email the password file
- Change password if suspect compromise
- Verify permissions monthly: `ls -la ~/.gocryptfs_pw` should show `-rw-------`

#### Option 4: Advanced - SSH Agent / Encrypted Key Store

For extreme security (beyond this course scope):

```bash
# Use ssh-agent to manage password securely
ssh-agent bash
ssh-add ~/.ssh/gocryptfs_key
# Then retrieve password from agent
```

This is for advanced users with security expertise. Not recommended for basic research workflows.

::::::::::::::::::::::::::::::::::::: callout
## Critical: Secure Your Password File

**IF** you store password in a file:

1. **Set permissions immediately:**
   ```bash
   chmod 600 ~/.gocryptfs_pw
   ls -la ~/.gocryptfs_pw  # Should be: -rw------
   ```

2. **Back it up:**
   ```bash
   cp ~/.gocryptfs_pw ~/backup/gocryptfs_pw.backup
   ```

3. **Never commit to git:**
   ```bash
   echo ".gocryptfs_pw" >> ~/.gitignore
   ```

4. **Verify periodically:**
   ```bash
   ls -la ~/.gocryptfs_pw  # Check permissions still 600
   ```

**IF** you don't follow these steps, you're undermining all encryption benefits. The password file is as sensitive as the encrypted data itself.

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Choose the Right Approach for Your Workflow

For each scenario, decide: should you pre-mount or mount-in-script? Why?

**Scenario 1:** You're debugging a new Python analysis script on encrypted data. The script needs tweaks. You'll probably run 10-15 test jobs over the next 2 hours.

**Scenario 2:** You need to run a production machine learning pipeline that processes encrypted patient data, trains a model, and saves results. This job runs overnight every day.

:::::::::::::::::::::::::::::::::::: solution

**Scenario 1 Answer: Pre-mount**

**Reasoning:**
- Short duration (2 hours of interactive work)
- Multiple small jobs (10-15 test runs)
- Frequently changing job script (just modifying analysis.py)
- Pre-mounting avoids mounting/unmounting overhead for each test
- You can run the same mount point across all test jobs
- Good for exploratory debugging work

**Steps:**
1. In terminal: `gocryptfs /bigdata/group/data_cipher /bigdata/group/data_plain`
2. Keep terminal open
3. Submit 15 jobs that all read from `/bigdata/group/data_plain`
4. After debugging complete, unmount: `fusermount -u /bigdata/group/data_plain`

**Scenario 2 Answer: Mount-in-script**

**Reasoning:**
- Production workflow that must be reproducible
- Long-running job (6+ hours overnight)
- Need guaranteed cleanup when job ends
- Job runs unattended (no human supervision)
- Same job script runs every day consistently
- Need full audit trail of what accessed data when
- Job arrays or scheduled runs should be self-contained

**Design:**
1. Store password in ~/.gocryptfs_pw with chmod 600
2. Job script mounts at start, unmounts at end via trap cleanup
3. Script is self-contained: no pre-mount dependency
4. Can schedule with cron or sbatch --begin flag
5. Each run is reproducible from the script alone

:::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Choose Your Approach

For each workflow scenario, decide whether **pre-mount** or **mount-in-script** is the better approach. Explain your reasoning.

1. You need to run a job array of 50 tasks, each processing a different subset of HIPAA-protected patient records. The array will run overnight unattended.
2. You are interactively exploring a new encrypted dataset to understand its structure, running quick one-off commands like `head`, `wc -l`, and `column` for about 30 minutes.
3. You have a weekly compliance pipeline that archives encrypted research data, generates a checksum report, and emails the PI. It runs every Sunday at 2 AM via cron.

::::::::::::::::::::::::::::::::::::: solution

## Solution

1. **Mount-in-script** — Each job array task should mount independently within its own script. Reasons: (a) 50 tasks may land on different compute nodes, so a pre-mount on the login node would not be visible to them; (b) overnight unattended jobs need guaranteed cleanup via `trap cleanup EXIT`; (c) each task mounting independently provides an audit trail of which task accessed which data and when; (d) HIPAA-regulated data demands reproducible, self-contained workflows.

2. **Pre-mount** — This is short, interactive exploration with no batch processing. Pre-mounting once avoids the overhead of writing a script for ad-hoc commands. You mount in your terminal session, explore the data, and unmount when done. There is no benefit to mount-in-script here since you are running commands manually.

3. **Mount-in-script** — A cron-scheduled job runs with no human present, so pre-mounting is not an option. The script must be entirely self-contained: mount at start, do work, unmount via trap. Reproducibility is essential for compliance pipelines, and the script itself serves as documentation of the archival process.

::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Pre-mount for interactive work and debugging; mount-in-script for production
- Never hardcode passwords in job scripts
- Use password files with chmod 600 for recommended security
- Understand your workflow type before choosing approach
- Reproducibility and cleanup are key for production jobs
::::::::::::::::::::::::::::::::::::::::::::::::
