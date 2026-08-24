---
title: "Security Best Practices and Troubleshooting"
teaching: 15
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions
- What are the 10 essential security habits for gocryptfs?
- What are the most common mistakes and how do you avoid them?
- How do you troubleshoot mount problems systematically?
- What performance optimizations exist and how much do they help?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Implement the 10 essential security habits for encrypted research data
- Recognize and prevent common encryption mistakes through practical scenarios
- Troubleshoot gocryptfs problems using a systematic diagnostic approach
- Optimize performance for realistic encrypted workflows
::::::::::::::::::::::::::::::::::::::::::::::::

## The 10 Essential Security Habits

Make encryption automatic, not effortful. Post this checklist near your workspace:

1. **Unmount when done**: Never leave mounted overnight or unattended
2. **Verify encryption**: Check cipher directory shows encrypted filenames (starting with `gocryptfs.`)
3. **Back up gocryptfs.conf**: Keep secure backup in /rhome; data is unrecoverable without it
4. **Use strong passwords**: 14+ characters, mixed case, numbers, symbols; unique to gocryptfs
5. **Restrict permissions**: chmod 700 for cipher and plain directories
6. **Never store plaintext copies**: Don't keep unencrypted versions of restricted data
7. **Use trap in scripts**: Automatic cleanup even if script fails
8. **Test recovery**: Verify quarterly that you can restore from backup
9. **Keep inventory**: Document all encrypted directories and backup locations
10. **Report incidents**: Contact its-hpc@pomona.edu immediately if suspicious access suspected

## Common Mistakes to Avoid

**Mistake 1: Leaving mounted overnight**
If a node crashes with mount active, plaintext is readable for days until IT discovers it. Prevention: Always unmount explicitly. In scripts, use `trap cleanup EXIT` for automatic cleanup.

**Mistake 2: Storing unencrypted backup copies**
Creates multiple copies of restricted data, increases exposure risk, defeats encryption. Prevention: Keep only encrypted copy. Back up gocryptfs.conf instead—it protects all data without exposing plaintext.

**Mistake 3: Sharing passwords with lab mates**
No audit trail, cannot revoke individual access, compliance violations. Prevention: Create separate encrypted directories per person with individual passwords.

**Mistake 4: Hardcoding passwords in git repos**
Password visible in git history forever, former employees gain permanent access. Prevention: Never hardcode passwords. Use password files (chmod 600) or password prompts, never commit to git.

**Mistake 5: Never testing backup recovery**
Backups silently become corrupted, discover failure only during disaster. Prevention: Test quarterly—takes 10 minutes. Copy backup to test location, mount with password, verify files readable, clean up. Log success date.

**Mistake 6: Inconsistent directory permissions**
Plain directory (chmod 755) readable by lab mates even if cipher is protected. Prevention: Both cipher and plain must be chmod 700. Check quarterly: `ls -ld /bigdata/lab/<labname>/*` should show drwx------ for both.

## Troubleshooting Guide: Systematic Diagnosis

Use this table to diagnose gocryptfs problems systematically. Read down the "Symptom" column, find your error, then follow the "Likely Cause" and "Solution" columns.

| Symptom | Likely Cause | Solution |
|---------|-------------|----------|
| **"Mount failed: cipher dir not found"** | Incorrect path or directory doesn't exist | Verify path exists: `ls /bigdata/lab/<labname>/secret_cipher`. Use absolute paths (starting with /). Check for typos. |
| **"Wrong password" (when password is correct)** | Data corruption or incorrect backup used | Don't panic. Try mounting from gocryptfs.conf in a different location. If it fails, data may be corrupted. Contact its-hpc@pomona.edu with: full error message, the cipher directory path, and when the problem started. |
| **"Decryption failed / Cannot read directory"** | Data corruption in cipher directory | Stop immediately. Do not attempt to modify or delete cipher files. Contact storage team about recovery options. Provide error message and backup info. |
| **"FUSE not available" or "fusermount: command not found"** | FUSE module not loaded or not installed | Ensure the gocryptfs module is loaded (`module load gocryptfs`); `fusermount` itself is part of the system FUSE installation. If it is still missing, contact its-hpc@pomona.edu. |
| **"Transport endpoint is not connected"** | Stale FUSE mount from crashed process (most confusing error!) | This means a previous mount process crashed, leaving FUSE in a broken state. Fix: `fusermount -u -z /scratch/$USER/secret_plain` (force unmount), then remount normally. |
| **"Address already in use"** | Another process has same mount point | Check if directory is mounted: `mount \| grep secret_plain`. If mounted, unmount: `fusermount -u /scratch/$USER/secret_plain`. If stuck, use force: `fusermount -u -z /scratch/$USER/secret_plain`. |
| **"Permission denied"** | User doesn't own cipher directory, or permissions are too restrictive | Check ownership: `ls -ld /bigdata/lab/<labname>/secret_cipher`. Owner should be you. Check permissions: should be 700. Fix: `chmod 700 /bigdata/lab/<labname>/secret_cipher`. Contact its-hpc@pomona.edu if you don't own the directory. |
| **"Out of memory" during mount or access** | Mounted directory has very many files (thousands), using significant RAM | Close other applications to free memory. For large directories, mount to a node with more available memory. Contact support for hardware options. Alternatively, split data into multiple smaller encrypted directories. |
| **"Input/output error" when reading files** | Data corruption in plaintext layer (less common) | Stop accessing files immediately. Unmount: `fusermount -u /scratch/$USER/secret_plain`. Contact storage team. Provide when error first appeared and what operations were in progress. |

### Special Case: "Transport endpoint is not connected"

This is the most confusing gocryptfs error. It happens when a previous mount process crashes but leaves FUSE in a broken state. The system thinks the mount point still exists but can't reach it.

**What's Happening**: FUSE maintains a communication channel between your process and the encrypted filesystem. If the process crashes (power failure, job timeout, system reboot), FUSE doesn't clean up properly. Next time you try to use that directory, you get this cryptic error.

**Fix (in order)**:
```bash
# Step 1: Try normal unmount first
fusermount -u /scratch/$USER/secret_plain

# Step 2: If that doesn't work, force unmount
fusermount -u -z /scratch/$USER/secret_plain

# Step 3: Verify it's unmounted (scripts: use `mountpoint -q` instead)
mount | grep secret_plain  # Human inspection — should show nothing

# Step 4: Remount normally
gocryptfs /bigdata/lab/<labname>/secret_cipher /scratch/$USER/secret_plain

# Step 5: Verify it's mounted (scripts: use `mountpoint -q` instead)
mount | grep secret_plain  # Human inspection — should show the mount

# Step 6: Test access
ls /scratch/$USER/secret_plain  # Should work now
```

**Prevention**: Use `trap` in your scripts to unmount on exit:
```bash
trap "fusermount -u /scratch/$USER/secret_plain" EXIT
```

This ensures cleanup even if your script is interrupted.

## Performance Optimization With Concrete Benchmarks

Encryption always adds overhead, but smart mounting strategies minimize the performance impact. Here are concrete optimization techniques with estimated performance improvements.

### 1. Mount on /tmpfs for Fastest I/O

**Performance Impact**: 3-5x faster than /bigdata for file operations.

Fastest performance strategy:

```bash
mkdir -p /tmpfs/mydata_plain
gocryptfs /bigdata/lab/<labname>/mydata_cipher /tmpfs/mydata_plain

# All operations now use RAM instead of disk
# Sequential read speed: ~8000 MB/s (RAM speed)
# vs. ~600 MB/s (BeeGFS disk speed)
# Encryption overhead: ~100 MB/s (AES-256-GCM)
```

**Important Caveat**: /tmpfs uses node RAM directly. On a 256GB node, only a fraction is available for mounting encrypted data. Use this only for temporary working data during active computation.

**When to Use**:
- Interactive analysis sessions (mount data, work, unmount)
- Running computations that read a lot but don't write results to encrypted storage
- Temporary data staging during jobs

**When NOT to Use**:
- Permanent storage (disappears on reboot)
- Data larger than available node memory
- Long-running jobs that need data persistence

### 2. Batch Operations: Mount Once, Do Many Things, Unmount Once

**Performance Impact**: 50-100x faster for repeated access than mount/unmount cycles.

Instead of:
```bash
# This pattern is expensive - mount/unmount for each file
gocryptfs mount
process_one_file
fusermount umount

gocryptfs mount
process_another_file
fusermount umount
```

Do:
```bash
# Mount once, access many times, unmount once
gocryptfs /bigdata/lab/<labname>/full_cipher /scratch/$USER/work
for file in /scratch/$USER/work/*.data; do
  process_file "$file"
done
fusermount -u /scratch/$USER/work
```

**Benchmark Example**: Processing 1000 files
- Bad approach (mount/unmount per file): ~50 seconds
- Good approach (single mount): ~2 seconds
- Speedup: 25x

**Why**: Mount/unmount operations involve FUSE initialization, filesystem setup, and encryption context creation. These are expensive compared to actually reading files.

### 3. Copy Hot Data to /scratch Within Encrypted Mount

**Performance Impact**: 10x faster for hot data, frees up encryption overhead.

Strategy for computational workflows:

```bash
#!/bin/bash
# Set up
CIPHER=/bigdata/lab/<labname>/large_dataset_cipher
PLAIN=/tmpfs/large_dataset_plain
WORK=/scratch/job_${SLURM_JOB_ID}

# Mount encrypted data
gocryptfs "$CIPHER" "$PLAIN"

# Copy only the data you need to work with to /scratch (non-persistent SSD)
# /scratch is unencrypted but local SSD is much faster than BeeGFS
cp "$PLAIN"/subset_for_today.npy "$WORK"/
cp "$PLAIN"/analysis_config.json "$WORK"/

# Do analysis on /scratch (no encryption overhead here)
python analysis.py --input "$WORK"/subset_for_today.npy --output "$WORK"/results.npy

# Copy results back through encryption
cp "$WORK"/results.npy "$PLAIN"/results_$(date +%Y%m%d).npy

# Unmount
fusermount -u "$PLAIN"

# Clean up /scratch (happens automatically on job end, but explicitly here)
rm -rf "$WORK"
```

**Benchmark Example**: 
- Reading from encrypted BeeGFS: ~400 MB/s
- Reading from /scratch SSD: ~4000 MB/s (10x faster)
- Total time savings for 100GB dataset: ~25 minutes

**When to Use**: Jobs where only a subset of encrypted data is needed for computation.

### 4. Avoid Encrypting Data That Doesn't Need It

**Performance Impact**: N/A to encryption, but saves I/O budget and quota.

Strategy: Only encrypt data that actually needs protection.

```bash
# ✓ Encrypt: restricted data, HIPAA/FERPA protected, 
#   experimental data before publication
# ✗ Don't encrypt: published datasets, code, documentation, 
#   public input files, intermediate processing outputs

/bigdata/lab/<labname>/
├── project_data_cipher/          # Restricted data: ENCRYPTED
│   ├── patient_interviews/
│   └── genomic_sequences/
├── project_code/                 # Public code: NOT encrypted
│   └── analysis_pipeline.py
├── project_results/              # Public results: NOT encrypted
│   └── published_figures/
└── project_data_unencrypted/     # Public data: NOT encrypted
    └── reference_genomes/
```

**Why**: Every encrypted file access adds ~100 MB/s decryption overhead (on top of I/O time). If data doesn't need encryption, don't encrypt it.

### 5. Benchmark gocryptfs Performance on Your Node

Check actual performance on current hardware:

```bash
# Measure encryption overhead on this node
gocryptfs -speed

# Output example:
# PBKDF2-SHA256:  100000 iterations, 33.76 ms/iteration
# OpenSSL:        not detected, switching to pure Go implementation
# AES-GCM:        2039.37 MB/s
# ChaCha20-Poly1305: 1234.56 MB/s

# For your workflows:
# - Plain BeeGFS I/O: ~600 MB/s
# - Encryption overhead: only when data is encrypted/decrypted
# - Mounted plaintext performance: limited by BeeGFS (600 MB/s), not encryption
```

**Interpretation**: The speed test shows encryption operations are orders of magnitude faster than disk I/O. The actual bottleneck is the storage system (BeeGFS at 600 MB/s), not encryption.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Find the Mistakes

Dr. Chen is a new researcher on the Sagehen cluster working with FERPA-protected student survey data. Read through her workflow and identify **5 security mistakes**.

> Dr. Chen created an encrypted directory at `/bigdata/lab/chenlab/surveys_cipher/` and mounts it daily. She wrote her gocryptfs password on a sticky note attached to her monitor so she would not forget it. Her SLURM job script contains the line `echo "SurveyPass2024!" | gocryptfs /bigdata/lab/chenlab/surveys_cipher /scratch/$USER/surveys_plain -` and she committed this script to the lab's GitHub repository. After her analysis jobs finish, she often leaves the mount active overnight because she plans to continue the next morning. She also keeps an unencrypted copy of the survey responses at `/bigdata/lab/chenlab/backup/surveys_raw.csv` in case the encryption ever fails. She has never tested whether her `gocryptfs.conf` backup actually works.

::::::::::::::::::::::::::::::::::::: solution

## Solution

1. **Password on a sticky note on her monitor.** Anyone passing by can read it. Passwords should be stored in a password manager (Bitwarden) or a chmod-600 file in `/rhome`, never in plain sight.

2. **Password hardcoded in the SLURM script.** The line `echo "SurveyPass2024!" | gocryptfs ...` exposes the password in the script file, SLURM job history, and process listings. She should read the password from a secure file: `PASSWORD=$(cat ~/.gocryptfs_pw)`.

3. **Script committed to GitHub with the hardcoded password.** The password is now permanently in git history, visible to anyone with repository access (and potentially the public). She must rotate the password immediately, remove the password from the script, and add `*.gocryptfs_pw` to `.gitignore`. The git history should be cleaned with `git filter-branch` or `BFG Repo-Cleaner`.

4. **Mount left active overnight.** While mounted, the plaintext data is accessible to anyone who compromises her account. She should unmount at the end of each session and use `trap cleanup EXIT` in scripts to guarantee automatic unmounting.

5. **Unencrypted backup copy stored on `/bigdata`.** Keeping `surveys_raw.csv` unencrypted completely defeats the purpose of encryption. FERPA-protected data must be encrypted at rest. She should delete the unencrypted copy and rely on her encrypted directory plus `gocryptfs.conf` backups.

**Bonus issue:** She has never tested her `gocryptfs.conf` backup. If the backup is corrupted or incomplete, she would discover this only during a disaster. She should test backup restoration quarterly.

::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Security habits become automatic: unmount at end of session, keep gocryptfs.conf backed up, verify passwords work
- The 10 essential security habits protect encrypted data across its entire lifecycle
- Common mistakes (leaving mounted, storing plaintext copies, sharing passwords, hardcoding passwords, not testing recovery) undermine encryption—avoid all of them
- Troubleshoot systematically using the problem table; "transport endpoint not connected" is usually just a stale mount
- Performance optimization focuses on smart mount placement (/tmpfs for temporary data) and batching operations
::::::::::::::::::::::::::::::::::::::::::::::::

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
