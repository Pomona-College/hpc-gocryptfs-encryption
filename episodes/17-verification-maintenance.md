---
title: "Verification, Sharing, and Maintenance"
teaching: 15
exercises: 15
---

:::::::::::::::::::::::::::::::::::::: questions
- How do you verify encryption is truly protecting your data?
- How do you maintain encrypted systems over time?
- What should you do if you suspect a security incident?
- How do you safely share encrypted data with collaborators?
- How do you manage data lifecycle and retention?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Verify encryption security with a complete testing protocol
- Maintain encrypted systems through a practical maintenance schedule
- Respond appropriately to potential security incidents
- Integrate gocryptfs with other Sagehen tools (OnDemand, VS Code, rclone)
- Implement secure data sharing practices with collaborators
- Manage encrypted data through its complete lifecycle
::::::::::::::::::::::::::::::::::::::::::::::::

## Verification Testing Protocol

Run this quarterly (January, April, July, October) to ensure encryption works and you can recover from disaster:

```bash
# Initialize test cipher directory
gocryptfs -init /tmp/test_cipher
mkdir -p /tmp/test_plain

# Test 1: Mount and verify plaintext is readable
gocryptfs /tmp/test_cipher /tmp/test_plain
echo "Test data $(date)" > /tmp/test_plain/test.txt
cat /tmp/test_plain/test.txt  # Should display readable text

# Test 2: Unmount and verify data is inaccessible
fusermount -u /tmp/test_plain
ls /tmp/test_plain 2>&1  # Should show error or empty

# Test 3: Remount with correct password
gocryptfs /tmp/test_cipher /tmp/test_plain
cat /tmp/test_plain/test.txt  # Data should appear again

# Test 4: Verify wrong password fails
fusermount -u /tmp/test_plain
echo "wrongpassword" | gocryptfs -q /tmp/test_cipher /tmp/test_plain 2>&1 | grep -i "wrong"  # Should fail

# Test 5: Test backup restoration
cp /tmp/test_cipher/gocryptfs.conf /tmp/gocryptfs_backup.conf
mkdir -p /tmp/restore_test
cp /tmp/gocryptfs_backup.conf /tmp/restore_test/gocryptfs.conf
gocryptfs /tmp/restore_test /tmp/restore_plain
ls /tmp/restore_plain  # Restored data should be readable
fusermount -u /tmp/restore_plain

# Document results
echo "$(date): Quarterly verification PASS" >> ~/encryption_maintenance.log
```

**If any test fails**: Contact its-hpc@pomona.edu with the exact error and output of `gocryptfs -version`.

## Sharing Encrypted Data With Collaborators

**Option 1: Separate Encrypted Directory Per Person (RECOMMENDED)**

Best for audit trails: each user has their own cipher directory and password.

```bash
# Create separate cipher directories for each collaborator
cp -r /bigdata/group/dataset_cipher /bigdata/group/dataset_alice_cipher
cp -r /bigdata/group/dataset_cipher /bigdata/group/dataset_bob_cipher

# Each person sets their own password
gocryptfs -init /bigdata/group/dataset_alice_cipher  # Alice enters password
gocryptfs -init /bigdata/group/dataset_bob_cipher    # Bob enters password

# Permissions: each person can only access their own
chmod 700 /bigdata/group/dataset_*_cipher
```

Advantages: audit trails, individual access revocation, per-person password rotation.

**Option 2: Decrypt → Transfer Securely → Re-encrypt (FOR HIPAA/FERPA DATA)**

Safest for sensitive data across institutions:

```bash
# Sender: decrypt and transfer
gocryptfs /bigdata/group/secure_data_cipher /tmp/plaintext
tar -czf /tmp/dataset.tar.gz -C /tmp plaintext/
scp /tmp/dataset.tar.gz collaborator.edu:/tmp/  # Use SSH only
shred -vfz -n 3 /tmp/plaintext

# Recipient: receive and re-encrypt
tar -xzf /tmp/dataset.tar.gz
gocryptfs -init /home/collaborator/data_cipher
gocryptfs /home/collaborator/data_cipher /tmp/mount_plain
cp -r plaintext/* /tmp/mount_plain/
fusermount -u /tmp/mount_plain
shred -vfz -n 3 /tmp/plaintext
```

**Option 3: rclone crypt for Cloud Storage**

For cloud-based access (separate from gocryptfs):

```bash
module load rclone
rclone config create mycrypt crypt --crypt-remote mycloud:
rclone sync /bigdata/group/research_data/ mycrypt:/research_data/
# Share rclone config with collaborators for cloud access
```

## Data Lifecycle and Encryption

**Encrypt immediately**: When data arrives and is classified as restricted, create encrypted directory right away:

```bash
gocryptfs -init /bigdata/group/new_study_cipher
gocryptfs /bigdata/group/new_study_cipher /tmp/mount
cp -r /tmp/incoming_data/* /tmp/mount/
fusermount -u /tmp/mount
```

**Retention timeline** (check your IRB/compliance office):
- Research data: 3-7 years after publication
- HIPAA data: 6 years minimum from last patient encounter
- FERPA data: During enrollment + 3 years after graduation
- Funded research: Per grant terms (typically 3-5 years)

Document retention dates in an inventory file.

**Deletion**: After retention period, simple deletion is sufficient (encrypted data is unreadable without the password/key):

```bash
rm -r /bigdata/group/interviews_cipher/
rm /rhome/$(whoami)/backup/gocryptfs_interviews.backup
```

## Integration With Other Sagehen Tools

**OnDemand (Jupyter, RStudio, VS Code)**: Mount encrypted data in your session script:
```bash
gocryptfs /bigdata/group/analysis_cipher /scratch/analysis_plain
# Apps access mounted plaintext directory
```

**VS Code SSH**: Mount before opening remote connection; Remote-SSH extension accesses decrypted path.

**rclone**: Mount encrypted data, then sync to cloud:
```bash
gocryptfs /bigdata/group/backup_cipher /tmp/backup_plain
rclone sync /tmp/backup_plain cloud-remote:/backup/
fusermount -u /tmp/backup_plain
```

## Maintenance Schedule

**Daily** (1 minute): Unmount encrypted directories at end of session.

**Weekly** (5 minutes): Check mounted filesystems with `mount | grep gocryptfs` (human inspection; for scripts use `mountpoint -q`) and unmount any stale mounts.

**Monthly** (10 minutes): Verify backup exists (`ls -lh ~/backup/gocryptfs_*.backup`) and test one mount/unmount cycle.

**Quarterly** (30 minutes): Run full verification protocol (see above), check for gocryptfs updates (`gocryptfs -version`), update inventory file.

**Annually** (60 minutes): Disaster recovery drill—restore from backup to test directory, mount, verify data readable, unmount and clean up. Document results.

## When to Contact Research Computing Support

**IMMEDIATE** (within 1 hour): Suspected breach, data corruption, compromised passwords.
- Contact: its-hpc@pomona.edu with subject "URGENT: Encryption security incident"
- Include: What happened, when discovered, what data affected

**SOON** (within 24 hours): Severe performance degradation, unable to mount, backup failures.
- Contact: its-hpc@pomona.edu with subject "Gocryptfs: [Issue type]"
- Include: Error message, commands run, expected vs. actual behavior

**ROUTINE** (next business day): Questions, password reset, new project setup.
- Contact: its-hpc@pomona.edu with subject "Question: gocryptfs setup"

## If You Suspect a Data Breach

1. **Don't panic or cover it up**—quick, honest response is most effective.
2. **Don't delete evidence**—preserve system state as-is.
3. **Contact immediately**: Email its-hpc@pomona.edu with when you noticed it, what suggests unauthorized access, which directories are affected, and whether you've modified anything.
4. **Gather information before IT arrives**:
   ```bash
   mount | grep -E 'cipher|plain'
   find /bigdata/group/ -type f -newermt "24 hours ago" -ls
   last -f /var/log/wtmp | head -20
   ```
5. **Let IT professionals handle the investigation**—they'll preserve evidence, check logs, advise on password changes, and coordinate compliance notification if needed.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Complete Verification Test

Run the quarterly verification protocol (about 20 minutes):

1. Initialize cipher, create and mount test data
2. Unmount and verify data is inaccessible
3. Remount; verify wrong password fails
4. Test backup restoration: copy gocryptfs.conf to new location and mount
5. Document results with timestamps; clean up

:::::::::::::::::::::::::::::::::::: solution

Successful completion shows:
- Cipher directory initialized with gocryptfs.conf
- Mounted data is readable; unmounted data is inaccessible
- Correct password succeeds; wrong password fails
- Backup restoration successful: restored config mounts and data readable
- Documentation file with timestamped results

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Security Audit

Identify all security problems in this setup and recommend fixes:

**Current Setup**:
- HIPAA patient interviews in unencrypted `/bigdata/group/research/` with `chmod 755`
- Encrypted cipher mounted 24/7 to `/home/peterson/data_plain` with `chmod 755`
- Password "research123" hardcoded in git, shared by 5 lab members
- `gocryptfs.conf` backup on unencrypted laptop (lab WiFi), never tested
- Account password is 8 characters, unchanged for 18 months

**Identify**: All security problems (at least 8)

**Recommend**: Specific fix command/procedure for each

:::::::::::::::::::::::::::::::::::: solution

1. Move patient data to cipher only: `mv /bigdata/group/research/* /tmp/plain/`
2. Fix permissions: `chmod 700 /home/peterson/data_plain` and cipher dir
3. Change password: `gocryptfs -change-password /bigdata/group/research_cipher`; remove from git
4. Unmount daily: Add to `~/.bash_logout`: `fusermount -u /home/peterson/data_plain 2>/dev/null`
5. Create per-person directories: `cp -r /bigdata/group/research_cipher /bigdata/group/research_alice_cipher`; each sets own password
6. Update account password: `passwd` to 14+ characters
7. Test backup: Restore to `/tmp/restore_test` and verify mounting
8. Move backup to secure location: `cp /laptop/backup/gocryptfs.conf /rhome/peterson/backup/`; `chmod 600`

Priority: (1) remove password from git, (2) change encryption password, (3) unmount directory, (4) create per-person directories, (5) test backup, (6) update account password

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Verify encryption with a complete testing protocol quarterly: can you mount, unmount, remount, use wrong password, restore from backup?
- Share encrypted data securely by creating separate encrypted directories per person, not sharing passwords
- Maintain regular schedules: daily unmount, weekly check, monthly verify, quarterly full test, annual disaster recovery drill
- Encrypt data immediately when it arrives and is classified as restricted; follow your IRB/compliance retention policy
- Integration with OnDemand, VS Code SSH, and rclone makes encrypted workflows practical and productive
- Contact its-hpc@pomona.edu immediately for suspected breaches; help is most effective when reported quickly
- Treat rclone crypt as a separate encryption layer for cloud-based sharing scenarios
- Document your encrypted directories, backup locations, and retention dates in an inventory file
::::::::::::::::::::::::::::::::::::::::::::::::
