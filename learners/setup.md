---
title: Setup
---

## Prerequisites

Before this workshop, you should have:

- An active **Sagehen HPC account** (request from [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu) if needed)
- Completed **[Workshop 13: HPC Security Orientation][ws-13]** (mandatory)
- Familiarity with the **Unix command line** (Workshops 1–2)
- Comfort with basic file operations (`mkdir`, `ls`, `chmod`, `cp`)

## Verify Your HPC Access

Log into Sagehen via [OnDemand](https://ondemand.hpc.pomona.edu) or SSH and run these checks:

```bash
# Confirm you can log in
ssh username@sagehen.hpc.pomona.edu

# Check your home directory
pwd
# Expected: /rhome/username

# Check your lab's storage
ls /bigdata/labname
# Replace "labname" with your lab directory name
```

If you cannot log in, contact [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu) before the workshop.

## Verify gocryptfs Is Available

```bash
# Check that the module exists
module avail gocryptfs

# Load it
module load gocryptfs

# Verify it works
gocryptfs --version
# Expected: gocryptfs v2.x.x
```

If the module is not available, email [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu) to request installation.

## Check Your Storage Quota

```bash
# Check lab quota (du does NOT work correctly on BeeGFS)
quota_check.sh
```

You will need at least **10 GB** of free space in `/bigdata/labname` for workshop exercises. If your lab is near its quota limit, contact [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu) to discuss options.

## Prepare a Strong Password

During the workshop you will create an encrypted directory that requires a password. Prepare one in advance:

- **Minimum 14 characters** (per NIST SP 800-63B)
- Mix of uppercase, lowercase, numbers, and symbols
- **Store it in a password manager** such as [Bitwarden](https://bitwarden.com) (free) or [1Password](https://1password.com)
- **There is no password recovery** — a forgotten password means permanently lost data

Do **not** write your password on a sticky note, in a plain text file, or in an email draft.

## Quick Readiness Check

Run this script to verify everything at once:

```bash
#!/bin/bash
echo "=== gocryptfs Workshop Readiness Check ==="

# Check home directory
echo -n "Home directory: "
if [ -d "$HOME" ]; then echo "OK ($HOME)"; else echo "FAIL"; fi

# Check gocryptfs module
echo -n "gocryptfs module: "
if module avail gocryptfs 2>&1 | grep -q gocryptfs; then
  module load gocryptfs
  echo "OK ($(gocryptfs --version 2>&1))"
else
  echo "FAIL — email its-hpc@pomona.edu"
fi

# Check /bigdata access
echo -n "/bigdata access: "
if [ -d "/bigdata" ]; then echo "OK"; else echo "FAIL"; fi

# Check /tmp directory creation
echo -n "Directory creation: "
TEST_DIR="/tmp/test_gocryptfs_$$"
if mkdir -p "$TEST_DIR" 2>/dev/null; then
  rmdir "$TEST_DIR"
  echo "OK"
else
  echo "FAIL"
fi

echo "=== Done ==="
```

Save this as `check_readiness.sh` and run it with `bash check_readiness.sh`. If any check fails, email [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu) with the output.

## Troubleshooting

**"gocryptfs: command not found"** — You need to load the module first: `module load gocryptfs`. If `module avail gocryptfs` shows nothing, the module is not installed; contact [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu).

**"Permission denied" on /bigdata/labname** — You may not be in the correct group. Run `groups` to check your group membership, then ask your PI or email [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu) to be added.

**"Disk quota exceeded"** — Run `quota_check.sh` to see current usage. Ask your PI about cleaning up old data, or email [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu) to request additional storage.
