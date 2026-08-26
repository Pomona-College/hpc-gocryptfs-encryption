---
title: Quick Reference
---

# gocryptfs Quick Reference Card

---

::::::::::::::::::::::::::::::::::::: callout
## Mount Verification: `mountpoint -q` for Scripts
For verification logic in scripts, prefer `mountpoint -q $MOUNT_POINT` (exact, no false positives — exits 0 if mounted, non-zero otherwise). Use `mount | grep` only for interactive (human) inspection. Throughout this reference, `mount | grep` examples are intended for human reading; substitute `mountpoint -q` in any automation.
::::::::::::::::::::::::::::::::::::::::::::::::

## Quick-Start TL;DR

If you're in a hurry, here's the 2-minute version:

```bash
# 1. Load module
module load gocryptfs

# 2. Create directories (first time only)
mkdir -p /bigdata/lab/<labname>/sensitive_cipher
mkdir -p /scratch/$USER/sensitive_plain

# 3. Initialize vault (first time only, creates gocryptfs.conf)
gocryptfs -init /bigdata/lab/<labname>/sensitive_cipher
# Enter password twice

# 4. Mount vault (whenever you need it)
gocryptfs /bigdata/lab/<labname>/sensitive_cipher /scratch/$USER/sensitive_plain
# Enter password once

# 5. Work with files in /scratch/$USER/sensitive_plain/
cp mydata.csv /scratch/$USER/sensitive_plain/
python analyze.py < /scratch/$USER/sensitive_plain/mydata.csv

# 6. Unmount when done (FILES WILL BE ENCRYPTED AGAIN)
fusermount -u /scratch/$USER/sensitive_plain

# 7. Verify unmounted
ls /scratch/$USER/sensitive_plain
# Should be empty
```

That's it! Your data in `/bigdata/lab/<labname>/sensitive_cipher` is now encrypted.

---

## "When to Encrypt" Decision Tree

Use this tree to decide if your data needs encryption.

```
START: Does your data contain personal identifiers or sensitive information?
│
├─→ NO (published data, lab protocols, public datasets)
│   │
│   └─→ Classification: PUBLIC (green)
│       Encryption: Optional (not required)
│       Permissions: 755 (world readable)
│       Location: /bigdata/lab/<labname>/public_data/
│       Example: Published journal articles, open-source code
│
└─→ YES (identifiers, sensitive info present)
    │
    ├─→ Is it health/medical information? (HIPAA applies)
    │   ├─→ YES → RESTRICTED (mandatory encryption)
    │   └─→ NO, continue to next question
    │
    ├─→ Are these student records? (FERPA applies)
    │   ├─→ YES → RESTRICTED (mandatory encryption)
    │   └─→ NO, continue to next question
    │
    ├─→ Is it genetic data with any identifiers?
    │   ├─→ YES → RESTRICTED (mandatory encryption)
    │   └─→ NO, continue to next question
    │
    ├─→ Does your IRB or funder REQUIRE encryption?
    │   ├─→ YES → RESTRICTED (mandatory encryption)
    │   └─→ NO, continue to next question
    │
    ├─→ Is this pre-publication research with competitive value?
    │   ├─→ YES → PROPRIETARY (recommended encryption)
    │   │   Classification: PROPRIETARY (orange)
    │   │   Encryption: RECOMMENDED
    │   │   Permissions: 750 (group only)
    │   │   Location: /bigdata/lab/<labname>/research_2026_cipher/
    │   │   Examples: Pre-publication data, novel algorithms, methods
    │   │
    │   └─→ NO (general lab data, low-risk)
    │       Classification: PROPRIETARY (orange)
    │       Encryption: Optional but recommended
    │       Permissions: 750 (group only)
    │       Location: /bigdata/lab/<labname>/lab_data_cipher/ (if encrypted)
    │
    └─→ STOP: RESTRICTED data found
        Classification: RESTRICTED (red)
        Encryption: MANDATORY (non-negotiable)
        Permissions: 700+gocryptfs (owner only, encrypted)
        Location: /bigdata/lab/<labname>/[dataset]_cipher/
        Examples: HIPAA, FERPA, genetic, IRB-protected
        
        ⚠️ YOU MUST ENCRYPT THIS DATA.
           Non-compliance = institutional liability.
           Cost of encryption << cost of breach.
```

---

## Essential gocryptfs Commands (Full Reference)

### Module Management

```bash
# Load gocryptfs module (required before any gocryptfs command)
module load gocryptfs

# Check available versions
module avail gocryptfs

# Unload module (optional, usually not needed)
module unload gocryptfs

# Verify module is loaded
module list | grep gocryptfs
```

---

### Initialize (Create Encrypted Vault) — **First Time Only**

```bash
# Basic initialization
gocryptfs -init /path/to/encrypted_cipher

# What it does:
# 1. Prompts you for a password (enter twice)
# 2. Creates gocryptfs.conf file (encrypted, in the vault directory)
# 3. Initializes encryption structure (RootNode, IV, etc.)

# Example:
mkdir -p /bigdata/lab/<labname>/restricted_data_cipher
gocryptfs -init /bigdata/lab/<labname>/restricted_data_cipher

# Console output:
# Choose a password for protecting your files.
# Password:
# Repeat password:
#
# Your master key is:
# f280a2e9-a6d2-4c3e-b1f2-8d9e7f6c5b4a
#
# If the master key is lost, the data is UNRECOVERABLE.
# Please save it in a safe place.

# IMPORTANT: Save that master key in your password manager!
# (Though password is usually enough; master key is backup)
```

**After initialization:**
- `gocryptfs.conf` exists in the vault directory (encrypted)
- Vault is ready for mounting
- Password is stored in your password manager (NOT the master key alone)

---

### Mount (Make Encrypted Data Readable)

```bash
# Basic mount
gocryptfs /path/to/encrypted_cipher /mount/point

# Full example:
gocryptfs /bigdata/lab/<labname>/restricted_data_cipher /scratch/$USER/restricted_plain

# Console interaction:
# Password: [type your password]
# Decrypting master key
# Mount successful

# What happens:
# - Prompts for password (you enter once)
# - Decrypts gocryptfs.conf
# - Mounts encrypted folder to mount point
# - Files now appear readable at mount point
# - All operations go through encryption/decryption automatically

# Verify mount succeeded:
mount | grep gocryptfs
# Output: /bigdata/lab/<labname>/restricted_data_cipher on /scratch/$USER/restricted_plain type fuse.gocryptfs

# Or:
ls /scratch/$USER/restricted_plain
# Shows your files (in plain text now)
```

**Expected behavior:**
- Mount point directory must exist beforehand
- Mount point should be empty (data appears at mount point)
- Entering wrong password causes: "Decrypting master key: password incorrect"
- After successful mount, file operations are transparent (encryption automatic)

---

### Unmount (Lock Encrypted Data) — **Always do this when done**

```bash
# Basic unmount
fusermount -u /mount/point

# Full example:
fusermount -u /scratch/$USER/restricted_plain

# Console output:
# (usually none, or "Unmounting..." message)

# Verify unmount:
mount | grep gocryptfs
# Should show nothing (no gocryptfs mounts)

# Or check the mount point:
ls /scratch/$USER/restricted_plain
# Should be empty (or show "No such file or directory" if you delete the directory)

# Force unmount (if stuck):
fusermount -uz /scratch/$USER/restricted_plain
# Use -z flag if normal unmount fails
# (mount point still exists but no longer mounted)
```

**Important:**
- Always unmount after finishing work
- Don't leave mounted overnight
- Force unmount (-z) if files are still open
- After unmount, files are encrypted again

---

### View Mounted Status

```bash
# See all gocryptfs mounts
mount | grep gocryptfs

# Example output:
# /bigdata/lab/<labname>/restricted_data_cipher on /scratch/$USER/restricted_plain type fuse.gocryptfs ...
# /bigdata/lab/<labname>/proprietary_cipher on /scratch/$USER/proprietary_plain type fuse.gocryptfs ...

# Or use df (disk free):
df -h | grep gocryptfs
# Shows mount point and size

# Or use mountpoint command:
mountpoint /scratch/$USER/restricted_plain
# Output: "/scratch/$USER/restricted_plain is a mountpoint" (or not)
```

---

### Get Vault Information

```bash
# Check vault details without mounting
gocryptfs -info /path/to/encrypted_cipher

# Example:
gocryptfs -info /bigdata/lab/<labname>/restricted_data_cipher

# Output:
# Filesystem version: 2
# Plaintext block size: 4096
# Default names: false
# PlaintextNames: false
# DirIV: true
# GCMIV128: true
# SecretsPassphrase: true
# HKDF: true
# AES: AES-256-GCM
# scrypt: true
# Reverse: false

# This tells you:
# - Encryption is working correctly
# - Algorithm is AES-256-GCM
# - Key derivation uses scrypt
# - Configuration is intact
```

---

## Complete Workflow Examples

### Example 1: Initialize, Mount, Work, Unmount

```bash
# ============================================
# STEP 1: Load module
# ============================================
module load gocryptfs

# ============================================
# STEP 2: Create directories (first time only)
# ============================================
mkdir -p /bigdata/lab/<labname>/health_survey_cipher
mkdir -p /scratch/$USER/health_survey_plain

# ============================================
# STEP 3: Initialize encrypted vault
# ============================================
gocryptfs -init /bigdata/lab/<labname>/health_survey_cipher

# You'll see:
# Choose a password for protecting your files.
# Password: [type: MyHealthData#2026]
# Repeat password: [type: MyHealthData#2026]
# Your master key is: f280a2e9-a6d2-4c3e-b1f2-8d9e7f6c5b4a

# SAVE THAT MASTER KEY IN YOUR PASSWORD MANAGER

# ============================================
# STEP 4: Verify vault initialization
# ============================================
ls -la /bigdata/lab/<labname>/health_survey_cipher/
# Output:
# drwx------. 1 username groupname 4096 Apr  9 10:23 .
# drwxr-xr-x. 1 username groupname 4096 Apr  9 10:20 ..
# -rw-------. 1 username groupname  266 Apr  9 10:23 gocryptfs.conf

# gocryptfs.conf is encrypted; you can't read it directly

# ============================================
# STEP 5: Mount the vault
# ============================================
gocryptfs /bigdata/lab/<labname>/health_survey_cipher /scratch/$USER/health_survey_plain

# You'll see:
# Password: [type: MyHealthData#2026]
# Decrypting master key
# Mount successful

# ============================================
# STEP 6: Verify mount worked
# ============================================
mount | grep gocryptfs
# Output: /bigdata/lab/<labname>/health_survey_cipher on /scratch/$USER/health_survey_plain type fuse.gocryptfs

ls -la /scratch/$USER/health_survey_plain/
# Output: empty (because we just created the vault)

# ============================================
# STEP 7: Copy data into mounted vault
# ============================================
cp ~/survey_results.csv /scratch/$USER/health_survey_plain/
cp ~/survey_analysis.py /scratch/$USER/health_survey_plain/

# Verify files are there:
ls /scratch/$USER/health_survey_plain/
# Output:
# survey_results.csv
# survey_analysis.py

# ============================================
# STEP 8: Work with the data
# ============================================
cd /scratch/$USER/health_survey_plain
wc -l survey_results.csv
# Output: 501 survey_results.csv (header + 500 participants)

python survey_analysis.py > results_summary.txt

ls -la
# Output:
# survey_results.csv
# survey_analysis.py
# results_summary.txt

# ============================================
# STEP 9: Verify encrypted version is gibberish
# ============================================
# While still mounted, check the encrypted folder:
ls /bigdata/lab/<labname>/health_survey_cipher/
# Output:
# gocryptfs.conf  (encrypted config)
# [random-encrypted-filename-1]
# [random-encrypted-filename-2]
# [random-encrypted-filename-3]

# Try to read one:
file /bigdata/lab/<labname>/health_survey_cipher/[random-encrypted-filename-1]
# Output: data (binary encrypted data)

# ============================================
# STEP 10: Unmount the vault
# ============================================
fusermount -u /scratch/$USER/health_survey_plain

# Verify unmount:
mount | grep health_survey_plain
# Output: (nothing shown = success)

# Check mount point is empty:
ls /scratch/$USER/health_survey_plain/
# Output: (empty)

# ============================================
# STEP 11: Verify encryption is intact
# ============================================
ls /bigdata/lab/<labname>/health_survey_cipher/
# Output: (still encrypted gibberish)

cat /bigdata/lab/<labname>/health_survey_cipher/[random-encrypted-filename-1] | head -c 50
# Output: (binary gibberish, not readable)

# ============================================
# STEP 12: Mount again to access data later
# ============================================
gocryptfs /bigdata/lab/<labname>/health_survey_cipher /scratch/$USER/health_survey_plain
# Password: [type: MyHealthData#2026]
# Mount successful

ls /scratch/$USER/health_survey_plain/
# Output:
# survey_results.csv
# survey_analysis.py
# results_summary.txt

# Your files are back! Encryption/decryption is transparent.
```

---

### Example 2: Share Encrypted Folder with Lab Member

```bash
# ============================================
# STEP 1: (Done by original owner)
# Verify encryption is set up and backed up
# ============================================
# Original owner has:
# - /bigdata/lab/<labname>/collaborative_data_cipher/ (encrypted vault)
# - Password stored in password manager
# - gocryptfs.conf backed up separately

# ============================================
# STEP 2: Communicate vault location
# ============================================
# Original owner tells collaborator:
# "The encrypted vault is at: /bigdata/lab/<labname>/collaborative_data_cipher/"
# (via email, Slack, team meeting, etc.)

# ============================================
# STEP 3: Communicate password securely
# ============================================
# Original owner does NOT send password via email!
# Instead:
# Option A: Share via password manager (Bitwarden shared folder)
# Option B: Read password over phone call
# Option C: Use Signal, Telegram, or other encrypted messaging
# Option D: Share in-person

# ============================================
# STEP 4: Collaborator mounts the vault
# ============================================
module load gocryptfs

mkdir -p /scratch/$USER/collaborative_plain

gocryptfs /bigdata/lab/<labname>/collaborative_data_cipher /scratch/$USER/collaborative_plain
# Password: [enters shared password]
# Mount successful

# ============================================
# STEP 5: Collaborator can access data
# ============================================
ls /scratch/$USER/collaborative_plain/
# Output: (files the original owner put there)

# Read or modify files:
head -5 /scratch/$USER/collaborative_plain/dataset.csv

# Add new data:
cp ~/analysis_results.txt /scratch/$USER/collaborative_plain/

# ============================================
# STEP 6: Collaborator unmounts
# ============================================
fusermount -u /scratch/$USER/collaborative_plain

# Data is encrypted again on disk
# Only encrypted vault is backed up to snapshots
```

---

### Example 3: Backup Encrypted Vault

```bash
# ============================================
# Method 1: Snapshot (if available on filesystem)
# ============================================
ls -la /bigdata/lab/<labname>/health_survey_cipher/.snapshot/
# Output (if BeeGFS snapshots available):
# daily.0  daily.1  daily.2  ... (automatic daily snapshots)

# Copy snapshot:
cp -r /bigdata/lab/<labname>/health_survey_cipher/.snapshot/daily.0/ \
      /bigdata/lab/<labname>/backups/health_survey_backup_2026-04-09/

# Verify backup:
ls -la /bigdata/lab/<labname>/backups/health_survey_backup_2026-04-09/
# Output: gocryptfs.conf (still encrypted)

# ============================================
# Method 2: Archive to tar.gz
# ============================================
cd /bigdata/lab/<labname>/
tar -czf health_survey_archive_2026-04.tar.gz health_survey_cipher/

# Verify archive:
ls -la health_survey_archive_2026-04.tar.gz
# Output: -rw-r--r--  1 username groupname 1.2G Apr  9 11:00 health_survey_archive_2026-04.tar.gz

# ============================================
# Method 3: Remote backup (e.g., external drive)
# ============================================
# Mount external drive: /mnt/external_drive/
# (Assuming already mounted)

cp -r /bigdata/lab/<labname>/health_survey_cipher/ /mnt/external_drive/backups/

# Note: Encrypted vault is encrypted at rest
# Even if external drive is stolen, data remains protected

# ============================================
# Method 4: Backup gocryptfs.conf separately
# ============================================
# This is CRITICAL! If gocryptfs.conf is lost, data is unrecoverable.

# Backup to password manager notes:
# In Bitwarden: Edit gocryptfs entry → Notes field
# Paste the output of:
cp /bigdata/lab/<labname>/health_survey_cipher/gocryptfs.conf \
   ~/gocryptfs_config_backup_2026-04-09.txt

# View (for backup purposes):
cat /bigdata/lab/<labname>/health_survey_cipher/gocryptfs.conf | base64
# Copy this base64 output into password manager as backup

# Restore from backup (if needed):
# 1. Decode base64 from password manager
# 2. Save to: /path/to/recovered_vault/gocryptfs.conf
# 3. Mount normally: gocryptfs /path/to/recovered_vault /mount
```

---

### Example 4: Password Change (gocryptfs -passwd)

```bash
# ============================================
# STEP 1: Vault must be unmounted
# ============================================
fusermount -u /scratch/$USER/health_survey_plain
# Verify unmount
mount | grep health_survey_plain
# (should show nothing)

# ============================================
# STEP 2: Change password
# ============================================
gocryptfs -passwd /bigdata/lab/<labname>/health_survey_cipher

# Console interaction:
# Password: [type: OLD_PASSWORD]
# New password: [type: NEW_PASSWORD]
# Repeat password: [type: NEW_PASSWORD]
# Password changed successfully

# ============================================
# STEP 3: Update password manager
# ============================================
# Immediately update your Bitwarden entry:
# 1. Open Bitwarden
# 2. Find "gocryptfs - health_survey" entry
# 3. Edit password field: OLD_PASSWORD → NEW_PASSWORD
# 4. Save

# ============================================
# STEP 4: Test new password
# ============================================
mkdir -p /scratch/$USER/health_survey_plain

gocryptfs /bigdata/lab/<labname>/health_survey_cipher /scratch/$USER/health_survey_plain
# Password: [type: NEW_PASSWORD]
# Mount successful

# Verify:
ls /scratch/$USER/health_survey_plain/
# Output: (your files are accessible)

# ============================================
# STEP 5: Unmount
# ============================================
fusermount -u /scratch/$USER/health_survey_plain
```

---

### Example 5: SLURM Job Script Integration

```bash
#!/bin/bash
# File: encrypt_analysis_job.sh
# This SLURM job mounts encrypted data, analyzes it, then unmounts

#SBATCH --job-name=encrypted_analysis
#SBATCH --time=02:00:00
#SBATCH --cpus-per-task=8
#SBATCH --mem=16G
#SBATCH --output=analysis_%j.log

# ============================================
# STEP 1: Load modules
# ============================================
module load gocryptfs
module load miniconda3/py313_26.3.2-2   # check `module avail` for current versions

# ============================================
# STEP 2: Set variables
# ============================================
ENCRYPTED_VAULT="/bigdata/lab/<labname>/restricted_data_cipher"
MOUNT_POINT="/tmp/restricted_data_${SLURM_JOB_ID}"
PASSWORD_FILE="/rhome/<myusername>/.gocryptfs_password"  # ⚠️ Store securely!

# ============================================
# STEP 3: Create mount point
# ============================================
mkdir -p "$MOUNT_POINT"

# ============================================
# STEP 4: Mount encrypted vault
# ============================================
echo "Mounting encrypted vault..."
echo "$PASSWORD" | gocryptfs -passfile - "$ENCRYPTED_VAULT" "$MOUNT_POINT"

if [ $? -ne 0 ]; then
    echo "ERROR: Failed to mount vault"
    exit 1
fi

# ============================================
# STEP 5: Set trap to unmount on exit
# ============================================
# This ensures unmount happens even if job fails
cleanup() {
    echo "Cleaning up: unmounting $MOUNT_POINT"
    fusermount -u "$MOUNT_POINT"
    rmdir "$MOUNT_POINT"
}
trap cleanup EXIT

# ============================================
# STEP 6: Do your analysis work
# ============================================
echo "Starting analysis at $(date)"

# Your analysis code:
python analyze_data.py \
    --input "$MOUNT_POINT/dataset.csv" \
    --output "$MOUNT_POINT/results.txt" \
    --cores $SLURM_CPUS_PER_TASK

if [ $? -ne 0 ]; then
    echo "ERROR: Analysis failed"
    exit 1
fi

# Verify results:
echo "Analysis complete. Results:"
head -20 "$MOUNT_POINT/results.txt"

echo "Finished analysis at $(date)"
# ============================================
# STEP 7: Cleanup (trap will run this automatically)
# ============================================
# (trap cleanup runs automatically on job end)
```

**Usage:**
```bash
# Submit job:
sbatch encrypt_analysis_job.sh

# Monitor:
squeue -u username
tail -f analysis_*.log
```

**Security notes:**
- Store password in secure file: `chmod 600 ~/.gocryptfs_password`
- Or use: `echo "mypassword" | gocryptfs -passfile - ...`
- Never hardcode password in script!
- Trap ensures unmount happens even if analysis crashes

---

## File Permissions Reference

### Data Classification & Permissions

| Classification | Tier | Encryption | chmod | Who Can Access |
|---|---|---|---|---|
| **PUBLIC** | Green | Optional | 755 | World (anyone on Sagehen HPC) |
| **PROPRIETARY** | Orange | Recommended | 750 | Owner + group members only |
| **RESTRICTED** | Red | **Mandatory** | 700+gocryptfs | Owner only (encrypted) |

### Specific chmod Values

```bash
# ==========================================
# PUBLIC data (published, open)
# ==========================================
chmod 755 /bigdata/lab/<labname>/public_data/
# Readable by: world
# Writable by: owner only

# ==========================================
# PROPRIETARY data (group access, not encrypted)
# ==========================================
chmod 750 /bigdata/lab/<labname>/research_data/
# Readable by: owner + group
# Writable by: owner + group
# No access: others (world)

# ==========================================
# PROPRIETARY encrypted data
# ==========================================
chmod 700 /bigdata/lab/<labname>/proprietary_cipher/
# (After mounting, use inside mount point)

# ==========================================
# RESTRICTED data (owner only, encrypted)
# ==========================================
chmod 700 /bigdata/lab/<labname>/restricted_cipher/
# Accessible by: owner only
# Encryption adds additional layer of security
# Data is unreadable even if permissions somehow fail

# ==========================================
# Lab directory (contains encrypted vaults)
# ==========================================
chmod 710 /bigdata/lab/<labname>/
# Readable by: owner + group (so they can mount their own vaults)
# Contains encrypted_vault_1_cipher/, encrypted_vault_2_cipher/, etc.
# Group members can mount if they know password
```

### Changing Permissions Example

```bash
# Set private (owner only)
chmod 700 /bigdata/lab/<labname>/restricted_cipher

# Set group-accessible (owner + group)
chmod 750 /bigdata/lab/<labname>/proprietary_cipher

# Set world-readable (public)
chmod 755 /bigdata/lab/<labname>/public_data

# Set world-readable, not writable
chmod 644 /bigdata/lab/<labname>/public_file.txt

# Verify permissions:
ls -la /bigdata/lab/<labname>/
# Output: drwx------  (700 = private)
#         drwxr-x---  (750 = group)
#         drwxr-xr-x  (755 = public)
```

---

## Troubleshooting Matrix (10+ Common Issues)

| Issue | Error Message | Cause | Solution |
|-------|---|---|---|
| **Module not found** | `gocryptfs: command not found` | Module not loaded | `module load gocryptfs` |
| **Module not available** | `No module(s) matched` | gocryptfs not installed | Email its-hpc@pomona.edu to request installation |
| **Wrong password** | `Bad password (password incorrect)` during mount | Typed password incorrectly | Try again, get password from password manager |
| **Vault already mounted** | `Filesystem is already mounted` | Tried to mount twice | `fusermount -u /mount/point`, then mount again |
| **Files still open** | `Resource temporarily unavailable` during unmount | Application still using files | Close applications, `cd` away, then `fusermount -u` |
| **Mount point doesn't exist** | `No such file or directory` during mount | Directory never created | `mkdir -p /tmp/mountpoint` first |
| **Permission denied on vault** | `Permission denied` when accessing vault | Wrong permissions on directory | `chmod 700 /path/to/vault`, or ask PI to add you to group |
| **Quota exceeded** | `Disk quota exceeded` | Lab has no remaining space | `quota_check.sh` to verify, email its-hpc@pomona.edu for increase |
| **Lost password** | Can't mount; no way in | Password forgotten, not in password manager | **Data is UNRECOVERABLE**. Restore from backup if available. |
| **Lost gocryptfs.conf** | `Decrypting master key: no such file or directory` | Configuration file deleted | **Data is UNRECOVERABLE** (password alone isn't enough). Restore from backup. |
| **du command doesn't work** | `du: cannot access '/bigdata/...': Permission denied` | BeeGFS doesn't support du properly | Use `quota_check.sh` instead of `du -sh` |
| **File corruption** | Files appear empty or corrupted after mount | Premature unmount or crash while file open | Restore from backup snapshot (.snapshot/daily.0/) |

### Detailed Troubleshooting Examples

**Issue: "gocryptfs: command not found"**
```bash
# Check if module is available
module avail gocryptfs

# If nothing shown:
# Email: its-hpc@pomona.edu
# Subject: "Request gocryptfs module for Sagehen"
# Body: "I need gocryptfs installed for Workshop 15"
```

**Issue: "Bad password (password incorrect)"**
```bash
# 1. Check password manager for correct password
# 2. Try again carefully (copy/paste from password manager)
gocryptfs /path/to/vault /mount/point
# Password: [copy from Bitwarden]

# 3. If still fails, reset password (see Example 4 above)
```

**Issue: "Resource temporarily unavailable" during unmount**
```bash
# 1. Close all applications using files in mount point
# 2. Leave the mounted directory:
cd /tmp
# (or any directory NOT mounted)

# 3. Try unmount:
fusermount -u /scratch/$USER/restricted_plain

# 4. If still fails, force unmount:
fusermount -uz /scratch/$USER/restricted_plain

# 5. Re-mount and try again:
gocryptfs /path/to/vault /scratch/$USER/restricted_plain
```

**Issue: "Disk quota exceeded"**
```bash
# 1. Check current usage
quota_check.sh

# 2. If available = 0, quota is full
# 3. Delete unnecessary files
rm -r /bigdata/lab/<labname>/old_data/

# 4. Or request increase from PI
# Email: its-hpc@pomona.edu
# Subject: "Request quota increase for [labname]"
# Include: "Current usage: [X] TB, need [Y] TB total"
```

**Issue: "Lost encryption password"**
```bash
# Unfortunately: DATA IS UNRECOVERABLE without password
# gocryptfs uses scrypt key derivation (brute-force resistant)
# Even Pomona IT cannot recover data

# What to do:
# 1. Check password manager (Bitwarden, KeePass, etc.)
# 2. Check browser password autofill history
# 3. Ask colleagues if they know the password
# 4. Email its-hpc@pomona.edu (they CANNOT recover it)
# 5. Restore from backup if available:
tar -xzf /bigdata/lab/<labname>/backups/vault_backup_2026-03.tar.gz

# PREVENTION: Store password immediately in password manager!
```

---

## gocryptfs.conf Backup Procedure (Step-by-Step)

### Why Backup gocryptfs.conf?

- Contains encrypted configuration
- Loss means data is UNRECOVERABLE (even with password!)
- Disaster recovery: If vault becomes corrupted
- Size: Only ~300 bytes, easy to backup

### Backup Steps

**Step 1: Verify vault is unmounted**
```bash
mount | grep gocryptfs
# Should show nothing (or not show your vault)
```

**Step 2: Locate gocryptfs.conf**
```bash
ls -la /bigdata/lab/<labname>/restricted_cipher/
# Output: -rw------- gocryptfs.conf
```

**Step 3: Create base64 copy (for password manager)**
```bash
cat /bigdata/lab/<labname>/restricted_cipher/gocryptfs.conf | base64 > gocryptfs_config_backup.txt

# View it:
cat gocryptfs_config_backup.txt
# Output: (long base64 string)
```

**Step 4: Store in password manager**
```bash
# In Bitwarden (or your password manager):
# 1. Open entry: "gocryptfs - restricted_data"
# 2. Edit → Notes field
# 3. Paste: [base64 string from step 3]
# 4. Also add:
#    - Vault location: /bigdata/lab/<labname>/restricted_cipher
#    - Mount point: /scratch/$USER/restricted_plain
#    - Created: 2026-04-09
#    - Last tested: 2026-04-09
# 5. Save
```

**Step 5: Create filesystem copy (backup location)**
```bash
mkdir -p /bigdata/lab/<labname>/backups/configs/

cp /bigdata/lab/<labname>/restricted_cipher/gocryptfs.conf \
   /bigdata/lab/<labname>/backups/configs/gocryptfs_restricted_cipher_2026-04-09.conf

# Verify:
ls -la /bigdata/lab/<labname>/backups/configs/
# Output: -rw------- gocryptfs_restricted_cipher_2026-04-09.conf
```

**Step 6: Test recovery (optional but recommended)**
```bash
# Simulate data loss by moving original vault temporarily:
mv /bigdata/lab/<labname>/restricted_cipher/gocryptfs.conf /tmp/gocryptfs_hidden.conf

# Restore from backup:
cp /bigdata/lab/<labname>/backups/configs/gocryptfs_restricted_cipher_2026-04-09.conf \
   /bigdata/lab/<labname>/restricted_cipher/gocryptfs.conf

# Try to mount:
mkdir /tmp/test_recovery
gocryptfs /bigdata/lab/<labname>/restricted_cipher /tmp/test_recovery
# Password: [your password]
# Mount successful?

# If yes, your backup works! Unmount:
fusermount -u /tmp/test_recovery

# If no, try restoring from password manager base64
```

---

## Maintenance Schedule

### Daily
- [ ] Check gocryptfs mounts: `mount | grep gocryptfs`
- [ ] Verify mounted vaults have expected files
- [ ] Unmount when done working

### Weekly
- [ ] Verify backup exists: `ls -la /bigdata/lab/<labname>/backups/`
- [ ] Spot-check backup is not corrupted

### Monthly
- [ ] Update password manager entries (if password changed)
- [ ] Review quota usage: `quota_check.sh`
- [ ] Check for unusual file access in mounted vault

### Quarterly
- [ ] Test password recovery (can you mount after restarting?)
- [ ] Test gocryptfs.conf recovery (test restore from backup)
- [ ] Review file permissions on vault directory

### Annually
- [ ] Archive old encrypted vaults: `tar -czf archive_2025.tar.gz vault_old/`
- [ ] Review encryption needs with PI (still RESTRICTED?)
- [ ] Update this maintenance log

---

## Emergency Contacts & Escalation

### Level 1: Self-Help (Start Here)
1. Check this reference guide (search by error message)
2. Run: `gocryptfs --help` and `gocryptfs -h`
3. Check Sagehen wiki: https://www.pomona.edu/its/
4. Google error message + "gocryptfs"

### Level 2: Password Manager Support
- **Bitwarden**: support@bitwarden.com
- **KeePass**: Community forums at keepass.info
- **1Password**: support.1password.com
- **LastPass**: support.lastpass.com

### Level 3: Sagehen HPC Support
- **Email**: its-hpc@pomona.edu
- **Expected response time**: 1-2 business days
- **Include in email**:
  - Your username
  - Error message (exact text)
  - Command you ran
  - Output of: `module list | grep gocryptfs`
  - Output of: `gocryptfs --version`

**Example email**:
```
Subject: gocryptfs mount failing with "Bad password"

Body:
I'm trying to mount an encrypted vault but getting "Bad password" error.

Command: gocryptfs /bigdata/lab/<labname>/data_cipher /scratch/$USER/data_plain
Error: Decrypting master key: password incorrect

Password is 16 characters, stored in Bitwarden.

Details:
- Username: msmith
- gocryptfs version: gocryptfs v2.x.x
- Module loaded: module list | grep gocryptfs → gocryptfs/2.x.x

I've tried 3 times with different passwords from my manager.
Can you help troubleshoot?
```

### Level 4: Escalation (Data Loss)
- **If data is lost**: Immediately email its-hpc@pomona.edu with "URGENT"
- **Include**:
  - When data was lost (date/time)
  - What you were doing
  - Can you restore from backup? (if yes, escalation may not be needed)
  - Is this RESTRICTED data? (regulatory implications)

- **If vault is corrupted**: Check snapshots immediately
  ```bash
  ls -la /bigdata/lab/<labname>/vault/.snapshot/
  # Try to restore: cp -r .snapshot/daily.0/* ./
  ```

### Level 5: Pomona IT Security (Breach Suspected)
- **If you suspect unauthorized access to encrypted vault**:
- Email: security@pomona.edu (and copy its-hpc@pomona.edu)
- Subject: "Suspected unauthorized access to encrypted vault"
- Include:
  - Vault location
  - When you noticed the issue
  - Whether data appears modified
  - Whether vault was mounted unexpectedly
  - Any unusual system activity

---

## Password Change Procedure (gocryptfs -passwd)

See Example 4 above for complete procedure. Quick version:

```bash
# 1. Unmount vault
fusermount -u /tmp/mountpoint

# 2. Change password
gocryptfs -passwd /path/to/vault_cipher
# Old password: [type old]
# New password: [type new]
# Repeat: [type new]

# 3. Update password manager IMMEDIATELY

# 4. Test new password
gocryptfs /path/to/vault_cipher /tmp/test_mount
# Password: [type new password]
# Unmount when done: fusermount -u /tmp/test_mount
```

---

## When to Encrypt: Complete Decision Table

| Data Type | FERPA? | HIPAA? | Genetic? | IRB? | Funder? | Pre-pub? | **Tier** | **Encryption** |
|---|---|---|---|---|---|---|---|---|
| Published journal article | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | PUBLIC | Optional |
| Student grades/records | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | RESTRICTED | **MANDATORY** |
| Patient health data | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | RESTRICTED | **MANDATORY** |
| Genetic variants + IDs | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | RESTRICTED | **MANDATORY** |
| IRB-protected research | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | RESTRICTED | **MANDATORY** |
| Pre-publication research | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | PROPRIETARY | Recommended |
| Lab protocols (published) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | PUBLIC | Optional |
| Lab notebooks (unpublished) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | PROPRIETARY | Recommended |
| Raw instrument data | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | PROPRIETARY | Recommended |
| Open-source code | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | PUBLIC | Optional |

---

## Best Practices Checklist

::::::::::::::::::::::::::::::::::: checklist

### Password Management
- [ ] Password is 14+ characters (preferably 16+)
- [ ] Password is unique (not used elsewhere)
- [ ] Password is stored in password manager (Bitwarden, KeePass, etc.)
- [ ] Master key is also stored in password manager (as backup)
- [ ] PI/lab manager knows where password is stored (for emergencies)
- [ ] Password is NOT in email, text, or plain text file

### Vault Management
- [ ] Vault is at: `/bigdata/lab/<labname>/`[dataset]_cipher/
- [ ] Mount point is on node-local scratch: /scratch/$USER/[dataset]_plain/ (never /bigdata or /rhome — BeeGFS)
- [ ] Vault has permissions: 700 (owner only)
- [ ] gocryptfs.conf is backed up separately
- [ ] Vault is only mounted when needed
- [ ] Vault is unmounted before leaving

### Backup Strategy
- [ ] Automated snapshots: ls -la `/bigdata/lab/<labname>/.`[snapshots/] working
- [ ] Manual archive: tar -czf backup_2026-04.tar.gz vault_cipher/
- [ ] gocryptfs.conf backed up to password manager
- [ ] At least 2 backup locations (filesystem + password manager)
- [ ] Backup tested: Can restore from backup successfully

### SLURM Integration
- [ ] Job script has trap for cleanup (unmount on job end)
- [ ] Password passed via -passfile (not hardcoded)
- [ ] Mount point is unique per job ($SLURM_JOB_ID)
- [ ] Results are saved to mounted point (encrypted automatically)
- [ ] Cleanup code: fusermount -u and rmdir

### Monitoring
- [ ] Check mounted vaults daily: mount | grep gocryptfs
- [ ] Verify quota monthly: quota_check.sh
- [ ] Test password quarterly (can you mount?)
- [ ] Review backup integrity quarterly

:::::::::::::::::::::::::::::::::::

---

## Common Operations Quick Ref

```bash
# Load module
module load gocryptfs

# Create directories
mkdir -p /bigdata/lab/<labname>/data_cipher /scratch/$USER/data_plain

# Initialize (first time)
gocryptfs -init /bigdata/lab/<labname>/data_cipher

# Mount
gocryptfs /bigdata/lab/<labname>/data_cipher /scratch/$USER/data_plain

# Work with files
cp myfile.csv /scratch/$USER/data_plain/
python analyze.py < /scratch/$USER/data_plain/myfile.csv

# Unmount
fusermount -u /scratch/$USER/data_plain

# Check mounts
mount | grep gocryptfs

# Verify vault
gocryptfs -info /bigdata/lab/<labname>/data_cipher

# Change password
gocryptfs -passwd /bigdata/lab/<labname>/data_cipher

# Backup vault
tar -czf backup_2026-04.tar.gz /bigdata/lab/<labname>/data_cipher/

# Check quota
quota_check.sh
```

---

## Resources

- **gocryptfs documentation**: https://nuetzlich.net/gocryptfs/
- **Sagehen HPC wiki**: https://www.pomona.edu/its/
- **Pomona College security**: https://www.pomona.edu/its/
- **NIST password guidelines**: https://pages.nist.gov/800-63-3/
- **Email support**: its-hpc@pomona.edu

---

**Last Updated**: April 2026  
**Version**: 2.1  
**For help**: Email its-hpc@pomona.edu

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
