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

### Approach 1: Mount in an Interactive Compute Session (Best for Exploratory Work)

Start an interactive session on a compute node, mount there, and do your work
inside that same session:

```bash
# Get an interactive shell on a compute node
srun --partition=short --time=01:00:00 --cpus-per-task=4 --mem=8G --pty bash

# Create the node-local mountpoint and mount
mkdir -p /scratch/$USER/secret_plain
gocryptfs /bigdata/lab/<labname>/secret_cipher /scratch/$USER/secret_plain
# Password: [enter password interactively]

# Work directly in this session
python3 analysis.py /scratch/$USER/secret_plain/data.csv

# Unmount before you exit
fusermount -u /scratch/$USER/secret_plain
exit
```

::::::::::::::::::::::::::::::::::::::: callout

## Why Not Mount First, Then sbatch?

A FUSE mount exists only on the node where you created it. A separately
submitted `sbatch` job will usually land on a *different* node, where your
mount doesn't exist. So a pre-created mount only serves work you run **inside
that same interactive session**. Any batch job must mount for itself
(Approach 2).

:::::::::::::::::::::::::::::::::::::::::::::::

**When to use an interactive-session mount:**
- Interactive analysis where you need frequent access
- Debugging encrypted data workflows
- Ad-hoc exploration before writing a production script

**Pros**:
- Interactive: type commands, inspect files, iterate
- Password entered interactively (no password file needed)

**Cons**:
- Must stay connected; the mount dies with your session
- Work is limited to that one node and session
- Not reproducible: depends on manual steps

### Approach 2: Mount in Job Script (Best for Production & Overnight Jobs)

Mount encrypted directories within the job script itself. This is self-contained and reproducible:

```bash
#!/bin/bash
#SBATCH --job-name=secure_analysis
#SBATCH --time=02:00:00

CIPHER=/bigdata/lab/<labname>/secret_cipher
PLAIN=/scratch/$USER/secret_plain   # node-local mountpoint

# Create mount point (scratch is cleaned between jobs)
mkdir -p "$PLAIN"

# Mount using a protected password file (see the password-handling
# comparison below -- never embed the password in the script itself)
gocryptfs --passfile ~/.gocryptfs_pass "$CIPHER" "$PLAIN"

# Do work
python3 analysis.py "$PLAIN"/data.csv

# Unmount
fusermount -u "$PLAIN"

# Cleanup
rmdir "$PLAIN"
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
1. In terminal: `gocryptfs /bigdata/lab/<labname>/data_cipher /scratch/$USER/data_plain`
2. Keep terminal open
3. Submit 15 jobs that all read from `/scratch/$USER/data_plain`
4. After debugging complete, unmount: `fusermount -u /scratch/$USER/data_plain`

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

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
