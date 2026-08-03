---
title: "Advanced SLURM Patterns"
teaching: 15
exercises: 15
---

:::::::::::::::::::::::::::::::::::::: questions
- How do you run job arrays with encrypted data?
- How can you build multi-step pipelines with encrypted workflows?
- What patterns work for GPU jobs with encrypted data?
- What error handling strategies prevent common failures?
- How do you log encrypted workflows responsibly?
- What does SLURM record about your job?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Implement job arrays with encrypted data
- Build multi-step pipelines that maintain encrypted context
- Run GPU jobs with encrypted datasets
- Apply advanced error handling patterns
- Implement security-aware logging practices
- Understand SLURM audit trail implications
::::::::::::::::::::::::::::::::::::::::::::::::

## Job Arrays with Encrypted Data

Use SLURM job arrays for many similar jobs. Each task mounts independently using `SLURM_ARRAY_TASK_ID`:

```bash
#!/bin/bash
#SBATCH --array=0-99
#SBATCH --time=00:30:00

trap cleanup EXIT

cleanup() {
    fusermount -u -l "$PLAIN" 2>/dev/null
    rmdir "$PLAIN" 2>/dev/null
}

CIPHER=/bigdata/group/data_cipher
PLAIN=$TMPDIR/plain_${SLURM_ARRAY_TASK_ID}

mkdir -p "$PLAIN"
PASSWORD=$(cat ~/.gocryptfs_pw)
echo "$PASSWORD" | gocryptfs "$CIPHER" "$PLAIN" - || exit 1

python3 analyze.py --input "$PLAIN/data.csv" \
    --param-set $SLURM_ARRAY_TASK_ID \
    --output results/result_${SLURM_ARRAY_TASK_ID}.txt
```

**Key points:** Each task mounts independently with unique mount point, cleanup is automatic via trap.

## Multi-Step Pipeline

Mount once, use for all steps (preprocess → train → evaluate):

```bash
#!/bin/bash
#SBATCH --time=06:00:00

trap cleanup EXIT

cleanup() {
    fusermount -u -l "$MOUNT" 2>/dev/null
    rmdir "$MOUNT" 2>/dev/null
}

CIPHER=/bigdata/group/raw_data
MOUNT=$TMPDIR/encrypted_input
SCRATCH=/scratch/$USER/pipeline_${SLURM_JOB_ID}

mkdir -p "$MOUNT" "$SCRATCH"
PASSWORD=$(cat ~/.gocryptfs_pw)
echo "$PASSWORD" | gocryptfs "$CIPHER" "$MOUNT" - || exit 1

# Step 1: Preprocess
python3 preprocess.py --input "$MOUNT/data.tar.gz" \
    --output "$SCRATCH/preprocessed.pkl" || exit 1

# Step 2: Train
python3 train.py --data "$SCRATCH/preprocessed.pkl" \
    --model "$SCRATCH/model.pkl" || exit 1

# Step 3: Evaluate
python3 evaluate.py --model "$SCRATCH/model.pkl" \
    --testdata "$MOUNT/test_data.tar.gz" \
    --output "$SCRATCH/results.json" || exit 1

# Archive results
mkdir -p /bigdata/group/results/pipeline_${SLURM_JOB_ID}
cp -r "$SCRATCH"/* /bigdata/group/results/pipeline_${SLURM_JOB_ID}/
```

**Pattern:** Encrypted input mounted for all steps, intermediate results in /scratch, final results in /bigdata.

## GPU Jobs with Encrypted Data

```bash
#!/bin/bash
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --time=04:00:00
#SBATCH --mem=32G

trap cleanup EXIT

cleanup() {
    fusermount -u -l "$DATA" 2>/dev/null
    rmdir "$DATA" 2>/dev/null
}

module load pytorch/2.0 cuda/12.0 python/3.11

DATA=$TMPDIR/patient_data
mkdir -p "$DATA"
PASSWORD=$(cat ~/.gocryptfs_pw)
echo "$PASSWORD" | gocryptfs /bigdata/group/patient_cipher "$DATA" - || exit 1

# Verify GPU
python3 -c "import torch; assert torch.cuda.is_available(), 'GPU not available'" || exit 1

# Run training
python3 train_gpu.py --data "$DATA/images" --labels "$DATA/labels.csv" \
    --output model.pt --device cuda --epochs 50 || exit 1
```

## Error Handling Patterns

**Pattern 1: Check mount success**
```bash
echo "$PASSWORD" | gocryptfs "$CIPHER" "$PLAIN" - || { echo "Mount failed"; exit 1; }
```

**Pattern 2: Verify data after mount**
```bash
[ -f "$PLAIN/required_data.csv" ] || { echo "Data missing"; exit 1; }
```

**Pattern 3: Check free space**
```bash
AVAIL=$(df "$PLAIN" | tail -1 | awk '{print $4}')
[ $AVAIL -gt 1000000 ] || { echo "Insufficient space"; exit 1; }
```

**Pattern 4: Verify work output**
```bash
python3 analysis.py "$PLAIN/input" > "$PLAIN/output" || exit 1
```

## Logging Best Practices

**What to log:**
```bash
LOG="job_${SLURM_JOB_ID}.log"
echo "$(date): Job starting, ID: $SLURM_JOB_ID" >> "$LOG"
echo "$PASSWORD" | gocryptfs "$CIPHER" "$PLAIN" - >> "$LOG" 2>&1 && \
    echo "$(date): Mount successful" >> "$LOG" || echo "$(date): Mount failed" >> "$LOG"
python3 analysis.py "$PLAIN/data" >> "$LOG" 2>&1
fusermount -u "$PLAIN" && echo "$(date): Cleanup complete" >> "$LOG"
```

**What NEVER to log:**
- Password: `echo "Password: $PASSWORD"` ← WRONG
- Decrypted data: `cat "$PLAIN/sensitive_data.csv"` ← WRONG
- Revealing paths: `echo "Processing $PLAIN/patient_123/records.pdf"` ← WRONG

**Smart masking:**
```bash
echo "Processing encrypted data..." >> "$LOG"
echo "Mount size: $(du -sh "$PLAIN" | cut -f1)" >> "$LOG"  # OK: sizes don't reveal content
```

## SLURM Audit Trail

SLURM admins see: job ID, user, script filename, arguments, nodes, resources, exit code, output files.

SLURM admins do NOT see: password contents, decrypted data, specific files accessed.

**Security rules:**
- Store passwords in files outside scripts (not hardcoded, not via --export)
- Use generic mount point names: `$TMPDIR/data` not `$TMPDIR/patient_study_123`
- Use generic job names: `--job-name=analysis` OK, `--job-name=patient_123_analysis` risky

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Complete GPU Training Script

Write a complete SLURM script for GPU-accelerated training on 50GB encrypted patient imaging data.

**Requirements:**
1. Mount point in /tmpfs (explain why)
2. Results saved to /bigdata (explain why)
3. Modules loaded and GPU verified
4. Error checks: mount, data exists, space available, GPU, training completion
5. Comprehensive logging (no passwords or decrypted paths)
6. Trap for cleanup

**Scenario:**
- Encrypted data: `/bigdata/group/patient_imaging_cipher/`
- Password file: `~/.gocryptfs_pw`
- GPU time: 4 hours
- Input: 50GB images + labels.csv (encrypted)
- Output: model.pt to persistent storage

:::::::::::::::::::::::::::::::::::: solution

```bash
#!/bin/bash
#SBATCH --job-name=gpu_patient_ml
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --time=04:00:00
#SBATCH --mem=32G

# DESIGN: /tmpfs for I/O-intensive data (faster than /bigdata network),
# /bigdata for results (persistent, needed for compliance), 
# password from file (never hardcoded or in env vars),
# trap ensures cleanup on success/failure/timeout.

set -o pipefail

CIPHER=/bigdata/group/patient_imaging_cipher
MOUNT=$TMPDIR/patient_data
RESULTS=/bigdata/group/patient_ml_results/run_${SLURM_JOB_ID}
LOG="${RESULTS}/job.log"

PASSWORD=$(cat ~/.gocryptfs_pw) || exit 1

cleanup() {
    local code=$?
    echo "Cleanup: exit code $code" >> "$LOG"
    mountpoint -q "$MOUNT" 2>/dev/null && fusermount -u -l "$MOUNT"
    [ -d "$MOUNT" ] && rmdir "$MOUNT" 2>/dev/null
    exit $code
}

trap cleanup EXIT

# Create dirs and setup logging
mkdir -p "$MOUNT" "$RESULTS"
exec 1> >(tee -a "$LOG")
exec 2>&1

echo "Job start: $(date), ID: $SLURM_JOB_ID, User: $(whoami)"

# Load modules
module load pytorch/2.0 cuda/12.0 python/3.11 || exit 1

# Verify GPU
python3 -c "import torch; assert torch.cuda.is_available()" || exit 1

# Mount
echo "Mounting encrypted data..."
echo "$PASSWORD" | gocryptfs "$CIPHER" "$MOUNT" - || exit 1

# Verify data
[ -d "$MOUNT/images" ] && [ -f "$MOUNT/labels.csv" ] || { echo "Data missing"; exit 1; }

# Check space
AVAIL=$(df "$MOUNT" | tail -1 | awk '{print $4}')
[ $AVAIL -gt $((50 * 1024 * 1024)) ] || { echo "Insufficient space"; exit 1; }

# Train
echo "Starting training..."
python3 train_model.py \
    --data "$MOUNT/images" \
    --labels "$MOUNT/labels.csv" \
    --output "$RESULTS/model.pt" \
    --device cuda --epochs 100 || exit 1

# Verify output
[ -f "$RESULTS/model.pt" ] || { echo "Model not created"; exit 1; }

echo "Job end: $(date), Results: $RESULTS"
```

**Design decisions:**
- **Mount in /tmpfs:** 50GB imaging I/O is much faster from RAM than network BeeGFS
- **Results in /bigdata:** Persistent storage needed for future inference and compliance
- **Password from file:** Secure, allows automation; never hardcode or use env vars
- **All error checks:** Fail fast if any step fails; trap ensures cleanup always runs

:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Debug Broken Script

This script has 5 errors. Identify and fix them.

```bash
#!/bin/bash
#SBATCH --job-name=broken_job
#SBATCH --time=00:30:00

CIPHER=/bigdata/group/data_cipher
PLAIN=$TMPDIR/decrypted
PASSWORD="MySecretPassword123"  # ERROR 1

mkdir -p $PLAIN
echo "$PASSWORD" | gocryptfs "$CIPHER/wrong_subdir" $PLAIN -  # ERROR 2

python3 analysis.py $PLAIN/input.csv > $PLAIN/output.csv  # ERROR 3

echo "Done"  # ERROR 4,5
```

::::::::::::::::::::::::::::::::::::: solution

**Errors:**

1. Hardcoded password (visible to admins)
2. Wrong mount path (silent failure)
3. No error check after mount (wrong SLURM status)
4. No cleanup trap (mount persists, blocks future jobs)
5. No unmount/rmdir (mount point directory left behind)

**Corrected:**

```bash
#!/bin/bash
cleanup() {
    local code=$?
    fusermount -u -l "$PLAIN" 2>/dev/null
    rmdir "$PLAIN" 2>/dev/null
    exit $code
}
trap cleanup EXIT

CIPHER=/bigdata/group/data_cipher
PLAIN=$TMPDIR/decrypted
PASSWORD=$(cat ~/.gocryptfs_pw) || exit 1

mkdir -p $PLAIN
echo "$PASSWORD" | gocryptfs "$CIPHER" $PLAIN - || exit 1
python3 analysis.py $PLAIN/input.csv > $PLAIN/output.csv || exit 1
echo "Done"
```

:::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Use job arrays to run many similar tasks with independent mounts
- Build multi-step pipelines that maintain encrypted context across steps
- GPU jobs need module loading and verification before mounting encrypted data
- Implement all 4 error handling patterns: mount success, data verification, work completion, space checks
- Log mount success/failure and work progress but never log passwords or decrypted paths
- Understand SLURM audit trail: admins see job names, scripts, arguments but not encrypted data contents
- Always include cleanup trap to guarantee unmounting on success, failure, timeout, or cancellation
::::::::::::::::::::::::::::::::::::::::::::::::
