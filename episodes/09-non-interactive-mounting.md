---
title: "Non-Interactive Mounting and Troubleshooting"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions
- How do you mount gocryptfs non-interactively in SLURM scripts?
- What are the security tradeoffs between different password methods?
- What happens to mounts when jobs complete or nodes reboot?
- What are the common mount errors and how do you fix them?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Mount gocryptfs non-interactively in SLURM job scripts
- Compare security of three non-interactive mounting methods
- Understand password storage security on Sagehen
- Troubleshoot common mount errors
- Understand mount persistence across reboots and job completion
- Implement proper error handling in SLURM scripts
::::::::::::::::::::::::::::::::::::::::::::::::

## Non-Interactive Mounting for SLURM Scripts

Interactive mounting (typing password at prompt) works for manual use. For batch jobs, you need non-interactive mounting.

### Three Approaches: Security Comparison

**Approach 1: Password via stdin**
```bash
echo "mypassword" | gocryptfs /cipher /plain -
```
Simple but password visible in history and process list. Avoid for production.

**Approach 2: Password from file (RECOMMENDED)**
```bash
echo "mypassword" > ~/.gocryptfs_password
chmod 600 ~/.gocryptfs_password
PASSWORD=$(cat ~/.gocryptfs_password)
echo "$PASSWORD" | gocryptfs /cipher /plain -
```
Password protected with 600 permissions. Store in encrypted /rhome directory. Use for all SLURM jobs.

**Approach 3: Environment variable**
```bash
export GOCRYPTFS_PASSWORD="mypassword"
gocryptfs /cipher /plain
```
Not supported on all builds. Avoid unless your gocryptfs explicitly supports it.

### Example SLURM Job Script

```bash
#!/bin/bash
#SBATCH --job-name=encrypt_analysis
#SBATCH --time=02:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4

# Set paths
CIPHER=/bigdata/lab/<labname>/research_cipher
PLAIN=/scratch/$USER/research_plain

# Create plain directory if needed
mkdir -p $PLAIN

# Mount encrypted directory (non-interactive)
PASSWORD=$(cat /rhome/<myusername>/secure_configs/gocryptfs_password.txt)
echo "$PASSWORD" | gocryptfs $CIPHER $PLAIN -

# Check mount succeeded
if ! mountpoint -q "$PLAIN"; then
    echo "ERROR: Mount failed"
    exit 1
fi

# Do your work
python3 analysis.py $PLAIN/data.csv > $PLAIN/results.txt

# Important: Always unmount before job ends
fusermount -u $PLAIN

# Verify unmount
if mountpoint -q "$PLAIN"; then
    echo "WARNING: Mount still active after unmount command"
    fusermount -uz $PLAIN
fi

echo "Job completed and unmounted"
```

**Key points**:
1. Create plain directory if it doesn't exist
2. Load password from secure file (600 permissions)
3. Mount using stdin method
4. Verify mount succeeded before using it
5. Always unmount at end (success or failure)
6. Verify unmount actually succeeded

### Password Storage Security

**In /rhome (recommended)**:
- Your home directory is encrypted on /rhome
- Password file is encrypted at rest
- File permissions (600) prevent other users
- You still own the encryption key

**Never in /bigdata**:
- Only encrypted by gocryptfs, not by system
- Defeats the purpose

**Never in /scratch or /tmpfs**:
- Deleted when job completes
- Not persistent

## Common Mount Errors and Fixes

| Error | Cause | Solution |
|-------|-------|----------|
| **"Plain directory not empty"** | Directory has files/hidden files | `ls -la /path`, then `rm -rf /path/*` and `rm -rf /path/.*` |
| **"Wrong password"** | Incorrect password | Verify CAPS LOCK, check password, verify cipher directory path |
| **"Address already in use"** | Directory already mounted | `mount \| grep gocryptfs` to check, then `fusermount -u /path` to unmount |
| **"Permission denied"** | Don't own directory or read-only storage | `ls -ld /path` to check ownership, `chown user /path` to fix |
| **"Password is correct but still fails"** | Data corruption or wrong backup | Don't modify cipher files. Contact its-hpc@pomona.edu with error message |

## Password Storage Best Practices

**Create a password file**:
```bash
echo "mypassword" > ~/.gocryptfs_pw
chmod 600 ~/.gocryptfs_pw  # Only you can read
```

**Use in SLURM scripts**:
```bash
PASSWORD=$(cat ~/.gocryptfs_pw)
echo "$PASSWORD" | gocryptfs /cipher /plain -
if [ $? -ne 0 ]; then
    echo "Mount failed"
    exit 1
fi
```

**Security rules**:
- Store password file in /rhome (encrypted at rest)
- Never store in /bigdata or /scratch (defeats encryption)
- Permissions must be 600 (no group/other access)
- Never commit passwords to git
- Rotate passwords quarterly
- Test recovery of gocryptfs.conf backup quarterly

## Complete SLURM Job Script Example

```bash
#!/bin/bash
#SBATCH --job-name=gocryptfs_analysis
#SBATCH --time=04:00:00
#SBATCH --cpus-per-task=4

# Setup
CIPHER=/bigdata/lab/<labname>/data_cipher
PLAIN=$TMPDIR/data_plain
PASSWORD=$(cat ~/.gocryptfs_pw)

# Cleanup on exit
cleanup() {
    fusermount -u -l "$PLAIN" 2>/dev/null
    rmdir "$PLAIN" 2>/dev/null
}
trap cleanup EXIT

# Create and mount
mkdir -p "$PLAIN"
echo "$PASSWORD" | gocryptfs "$CIPHER" "$PLAIN" -
[ $? -ne 0 ] && exit 1

# Verify data exists
[ -f "$PLAIN/input.csv" ] || { echo "Missing input.csv"; exit 1; }

# Process
python3 analysis.py "$PLAIN/input.csv" > results.txt

# Cleanup and exit automatically
```

Key patterns:
- Always use `trap cleanup EXIT` for guaranteed unmounting
- Create mount point with `mkdir -p`
- Check mount succeeded before accessing data
- Return proper exit codes for job tracking
- Store password in /rhome with chmod 600
- Verify mount with `mountpoint -q` (script-safe) or `ls` before processing

## Verifying Mount Success

After mounting, always verify before using data:

::::::::::::::::::::::::::::::::::::: callout
## Prefer `mountpoint -q` for Script Verification
For verification logic in scripts, prefer `mountpoint -q $MOUNT_POINT` (exact, no false positives). Use `mount | grep` only for interactive inspection.
::::::::::::::::::::::::::::::::::::::::::::::::

```bash
# Method 1: Check mount exists (script-safe)
if ! mountpoint -q "$PLAIN"; then
    echo "ERROR: Mount failed"
    exit 1
fi

# Method 2: Try accessing data
if [ ! -f "$PLAIN/expected_file.txt" ]; then
    echo "ERROR: Expected file not found"
    exit 1
fi

# Method 3: List contents
ls "$PLAIN/" > /dev/null || { echo "ERROR: Cannot read mount"; exit 1; }
```

Mount persistence: Mounts do NOT survive job completion or node reboots. Always create fresh mount at job start and clean up at end using trap.

::::::::::::::::::::::::::::::::::::: challenge

## Troubleshooting Challenges

**Device busy**: Use `lsof /path` to find processes using mount, `kill [PID]` to close them, retry `fusermount -u /path`.

**Wrong password**: gocryptfs.conf can't decrypt with wrong password. Data is NOT recoverable if password lost. Get correct password from cipher creator.

**Hidden files blocking mount**: Use `ls -la` to see dot-prefixed files, remove with `rm -rf /path/*` and `rm -rf /path/.*`.

**Permission denied**: Check ownership with `ls -ld /path`. You must own the directory. Fix with `chown username /path` or create own mount point.

:::::::::::::::::::::::::::::::::::: solution

**Device busy**: `lsof /path` → identify process → `kill [PID]` → retry unmount or use `fusermount -uz`
**Wrong password**: Obtain correct password from cipher creator, retry with correct password
**Hidden files**: `ls -la` reveals them, remove with `rm -rf /path/.*` (careful pattern!)
**Permission denied**: Fix ownership with `chown` or create own mount point

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Spot the Security Flaw

Each SLURM script snippet below contains a security problem. Identify the flaw in each and explain how to fix it.

**Snippet 1:**
```bash
#!/bin/bash
#SBATCH --job-name=analysis
echo "R3search$ecure2024!" | gocryptfs /bigdata/lab/<labname>/cipher /scratch/$USER/plain -
python3 analyze.py /scratch/$USER/plain/data.csv
fusermount -u /scratch/$USER/plain
```

**Snippet 2:**
```bash
#!/bin/bash
#SBATCH --job-name=analysis
PASSWORD=$(cat /bigdata/lab/<labname>/shared/gocryptfs_password.txt)
echo "$PASSWORD" | gocryptfs /bigdata/lab/<labname>/cipher /scratch/$USER/plain -
python3 analyze.py /scratch/$USER/plain/data.csv
fusermount -u /scratch/$USER/plain
```

**Snippet 3:**
```bash
#!/bin/bash
#SBATCH --job-name=analysis
export GOCRYPTFS_PW="MySecretPass"
echo "$GOCRYPTFS_PW" | gocryptfs /bigdata/lab/<labname>/cipher /scratch/$USER/plain -
python3 analyze.py /scratch/$USER/plain/data.csv
fusermount -u /scratch/$USER/plain
```

::::::::::::::::::::::::::::::::::::: solution

## Solution

**Snippet 1 — Hardcoded password in script:**
The password `R3search$ecure2024!` is written directly in the script file. It is visible to anyone who can read the script, will appear in SLURM job history, and if the script is committed to git the password is in the repository forever.
**Fix:** Store the password in a separate file with restrictive permissions:
```bash
PASSWORD=$(cat ~/.gocryptfs_pw)   # file with chmod 600
echo "$PASSWORD" | gocryptfs /bigdata/lab/<labname>/cipher /scratch/$USER/plain -
```

**Snippet 2 — Password file stored on `/bigdata` (world/group-readable location):**
The password file is at `/bigdata/lab/<labname>/shared/gocryptfs_password.txt`. Even with correct file permissions, storing the password on `/bigdata` means it sits on the same unencrypted filesystem as the cipher directory, and the path `shared/` suggests group access. This defeats the purpose of encryption.
**Fix:** Store the password file in `/rhome` (which is encrypted at rest) and ensure permissions are 600:
```bash
PASSWORD=$(cat /rhome/<myusername>/.gocryptfs_pw)   # chmod 600, on /rhome
```

**Snippet 3 — Password in environment variable without cleanup:**
The password is exported as an environment variable and never unset. Exported variables are visible via `/proc/[pid]/environ` and can be read by child processes or other tools. The variable persists for the lifetime of the job.
**Fix:** Read from a password file instead, or at minimum unset the variable immediately after use:
```bash
PASSWORD=$(cat ~/.gocryptfs_pw)
echo "$PASSWORD" | gocryptfs /bigdata/lab/<labname>/cipher /scratch/$USER/plain -
unset PASSWORD
```

::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Non-interactive mounting: Use password file (Approach 2) with chmod 600 for SLURM jobs
- Store password in encrypted /rhome, never in /bigdata or /scratch
- Three verification methods: mountpoint -q (script-safe), df, and ls
- Always verify mount succeeded before using encrypted data
- Always unmount at job end, even if job fails
- Wrong password = data permanently inaccessible (no recovery)
- Common errors: directory not empty, wrong password, permission denied, address in use
- Use lsof to find open files causing "device busy" errors
- Lazy unmount (fusermount -uz) is last resort
- Mounts do NOT survive reboots or job completion
- Implement proper error handling and unmounting in SLURM scripts
::::::::::::::::::::::::::::::::::::::::::::::::

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
