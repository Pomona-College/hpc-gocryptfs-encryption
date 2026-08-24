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
mkdir -p /scratch/$USER/test_plain

# Test 1: Mount and verify plaintext is readable
gocryptfs /tmp/test_cipher /scratch/$USER/test_plain
echo "Test data $(date)" > /scratch/$USER/test_plain/test.txt
cat /scratch/$USER/test_plain/test.txt  # Should display readable text

# Test 2: Unmount and verify data is inaccessible
fusermount -u /scratch/$USER/test_plain
ls /scratch/$USER/test_plain 2>&1  # Should show error or empty

# Test 3: Remount with correct password
gocryptfs /tmp/test_cipher /scratch/$USER/test_plain
cat /scratch/$USER/test_plain/test.txt  # Data should appear again

# Test 4: Verify wrong password fails
fusermount -u /scratch/$USER/test_plain
echo "wrongpassword" | gocryptfs -q /tmp/test_cipher /scratch/$USER/test_plain 2>&1 | grep -i "wrong"  # Should fail

# Test 5: Test backup restoration
cp /tmp/test_cipher/gocryptfs.conf /tmp/gocryptfs_backup.conf
mkdir -p /tmp/restore_test
cp /tmp/gocryptfs_backup.conf /tmp/restore_test/gocryptfs.conf
gocryptfs /tmp/restore_test /scratch/$USER/restore_plain
ls /scratch/$USER/restore_plain  # Restored data should be readable
fusermount -u /scratch/$USER/restore_plain

# Document results
echo "$(date): Quarterly verification PASS" >> ~/encryption_maintenance.log
```

**If any test fails**: Contact its-hpc@pomona.edu with the exact error and output of `gocryptfs -version`.

## Sharing Encrypted Data With Collaborators

**Option 1: Separate Encrypted Directory Per Person (RECOMMENDED)**

Best for audit trails: each user has their own cipher directory and password.

```bash
# Create separate cipher directories for each collaborator
cp -r /bigdata/lab/<labname>/dataset_cipher /bigdata/lab/<labname>/dataset_alice_cipher
cp -r /bigdata/lab/<labname>/dataset_cipher /bigdata/lab/<labname>/dataset_bob_cipher

# Each person sets their own password
gocryptfs -init /bigdata/lab/<labname>/dataset_alice_cipher  # Alice enters password
gocryptfs -init /bigdata/lab/<labname>/dataset_bob_cipher    # Bob enters password

# Permissions: each person can only access their own
chmod 700 /bigdata/lab/<labname>/dataset_*_cipher
```

Advantages: audit trails, individual access revocation, per-person password rotation.

**Option 2: Decrypt → Transfer Securely → Re-encrypt (FOR HIPAA/FERPA DATA)**

Safest for sensitive data across institutions:

```bash
# Sender: decrypt and transfer (inside a compute session)
mkdir -p /scratch/$USER/plaintext
gocryptfs /bigdata/lab/<labname>/secure_data_cipher /scratch/$USER/plaintext
tar -czf /tmp/dataset.tar.gz -C /scratch/$USER plaintext/
fusermount -u /scratch/$USER/plaintext
scp /tmp/dataset.tar.gz collaborator.edu:/tmp/  # Use SSH only
shred -vfz -n 3 /tmp/dataset.tar.gz   # shred the tarball -- it holds plaintext

# Recipient: receive and re-encrypt
tar -xzf /tmp/dataset.tar.gz
gocryptfs -init /home/collaborator/data_cipher
gocryptfs /home/collaborator/data_cipher /scratch/$USER/mount_plain
cp -r plaintext/* /scratch/$USER/mount_plain/
fusermount -u /scratch/$USER/mount_plain
shred -vfz -n 3 plaintext/*   # remove the temporary decrypted copies
```

**Option 3: rclone crypt for Cloud Storage**

For cloud-based access (separate from gocryptfs):

```bash
module load rclone
rclone config create mycrypt crypt --crypt-remote mycloud:
rclone sync /bigdata/lab/<labname>/research_data/ mycrypt:/research_data/
# Share rclone config with collaborators for cloud access
```

## Data Lifecycle and Encryption

**Encrypt immediately**: When data arrives and is classified as restricted, create encrypted directory right away:

```bash
gocryptfs -init /bigdata/lab/<labname>/new_study_cipher
gocryptfs /bigdata/lab/<labname>/new_study_cipher /scratch/$USER/mount
cp -r /tmp/incoming_data/* /scratch/$USER/mount/
fusermount -u /scratch/$USER/mount
```

**Retention timeline** (check your IRB/compliance office):
- Research data: 3-7 years after publication
- HIPAA data: 6 years minimum from last patient encounter
- FERPA data: During enrollment + 3 years after graduation
- Funded research: Per grant terms (typically 3-5 years)

Document retention dates in an inventory file.

**Deletion**: After retention period, simple deletion is sufficient (encrypted data is unreadable without the password/key):

```bash
rm -r /bigdata/lab/<labname>/interviews_cipher/
rm /rhome/$(whoami)/backup/gocryptfs_interviews.backup
```

## Integration With Other Sagehen Tools

**OnDemand (Jupyter, RStudio, VS Code)**: Mount encrypted data in your session script:
```bash
gocryptfs /bigdata/lab/<labname>/analysis_cipher /scratch/$USER/analysis_plain
# Apps access mounted plaintext directory
```

**VS Code SSH**: Mount before opening remote connection; Remote-SSH extension accesses decrypted path.

**rclone**: Mount encrypted data, then sync to cloud:
```bash
gocryptfs /bigdata/lab/<labname>/backup_cipher /scratch/$USER/backup_plain
rclone sync /scratch/$USER/backup_plain cloud-remote:/backup/
fusermount -u /scratch/$USER/backup_plain
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
   find /bigdata/lab/<labname>/ -type f -newermt "24 hours ago" -ls
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
- HIPAA patient interviews in unencrypted `/bigdata/lab/<labname>/research/` with `chmod 755`
- Encrypted cipher mounted 24/7 to `/home/peterson/data_plain` with `chmod 755`
- Password "research123" hardcoded in git, shared by 5 lab members
- `gocryptfs.conf` backup on unencrypted laptop (lab WiFi), never tested
- Account password is 8 characters, unchanged for 18 months

**Identify**: All security problems (at least 8)

**Recommend**: Specific fix command/procedure for each

:::::::::::::::::::::::::::::::::::: solution

1. Move patient data to cipher only: `mv /bigdata/lab/<labname>/research/* /scratch/$USER/plain/` (with the cipher mounted there)
2. Fix permissions on the cipher dir (`chmod 700`) — and relocate the mountpoint to `/scratch/$USER/`: mounts under `/rhome` fail because it is BeeGFS
3. Change password: `gocryptfs -change-password /bigdata/lab/<labname>/research_cipher`; remove from git
4. Unmount daily: Add to `~/.bash_logout`: `fusermount -u /home/peterson/data_plain 2>/dev/null`
5. Create per-person directories: `cp -r /bigdata/lab/<labname>/research_cipher /bigdata/lab/<labname>/research_alice_cipher`; each sets own password
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

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
