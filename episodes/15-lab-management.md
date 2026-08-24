---
title: "Lab Management and Disaster Recovery"
teaching: 15
exercises: 15
---

:::::::::::::::::::::::::::::::::::::: questions
- What procedures should a lab have when a member with encryption access leaves?
- How do you plan for succession if a PI becomes unavailable?
- How do you track multiple encrypted directories in a lab?
- What automated backup strategies can a lab implement?
- What are the main disaster recovery scenarios and recovery procedures?
- What compliance documentation is required for encrypted research data?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Implement lab procedures for handling personnel transitions with encrypted data
- Create succession plans for PI's encrypted data
- Maintain inventory of multiple encrypted directories
- Implement automated backup scripts with cron scheduling
- Understand and recover from five major disaster scenarios
- Document compliance requirements for encrypted research (NIST, HIPAA)
- Maintain audit trails and testing records for encryption management
::::::::::::::::::::::::::::::::::::::::::::::::

## Lab Group Key Management

When multiple people work with encrypted data, you need procedures for sharing, transitions, and handoffs.

### Handling Encryption When Lab Member Leaves

When a lab member with encryption access departs, follow this sequence:

```bash
# Step 1: Secure transition (2 weeks notice)
# Departing member shares password via in-person or encrypted channel
# New member tests access before departure

# Step 2: Change password
gocryptfs -passwd /bigdata/lab/<labname>/patient_data_cipher/

# Step 3: Back up new gocryptfs.conf
cp /bigdata/lab/<labname>/patient_data_cipher/gocryptfs.conf \
   ~/backup/gocryptfs_patient_data.conf

# Step 4: Store backup (use 3-2-1 rule: 3 copies, 2 media types, 1 offsite)
```

### Succession Planning for PI's Encrypted Data

Before you're needed to transition, create a succession plan:

1. Identify successor (co-PI, senior postdoc, lab manager)
2. Create sealed document with encrypted directory list and successor contact
3. Store written passwords in safe deposit box (not office)
4. Brief successor annually on procedures
5. Test that successor can access directories

### Multiple Encrypted Directories: Inventory Tracking

Maintain an inventory listing: path, contents, password storage, backup locations, compliance, access control.

## Automated Backup Script

Back up all gocryptfs.conf files:

```bash
#!/bin/bash
# backup_gocryptfs.sh - Back up gocryptfs.conf files
BACKUP_DIR="${HOME}/backup/gocryptfs_keys"
mkdir -p "$BACKUP_DIR"
chmod 700 "$BACKUP_DIR"

CIPHER_DIRS=("/bigdata/lab/<labname>/patient_study_a_cipher" "/bigdata/lab/<labname>/patient_study_b_cipher")

for CIPHER_DIR in "${CIPHER_DIRS[@]}"; do
    CONF_FILE="${CIPHER_DIR}/gocryptfs.conf"
    [ -f "$CONF_FILE" ] && cp "$CONF_FILE" "$BACKUP_DIR/gocryptfs_$(basename $CIPHER_DIR)_$(date +%Y-%m-%d).conf" && chmod 600 "$_"
done

find "$BACKUP_DIR" -name "gocryptfs_*.conf" -mtime +180 -delete  # Clean old backups
```

**Add to cron**: `0 9 1 * * /rhome/<myusername>/backup_gocryptfs.sh` (monthly, first day 9 AM)

## Disaster Recovery Scenarios

| Scenario | Have | Result | Recovery | Time |
|----------|------|--------|----------|------|
| 1. gocryptfs.conf + password | Both | FULL RECOVERY | Restore conf, mount | <5 min |
| 2. Lost conf, no backup | Password only | PERMANENT | Check system backups | N/A |
| 3. Forgot password, have conf | Config only | PERMANENT | No recovery possible | N/A |
| 4. Lost conf with backup | Both | FULL RECOVERY | Restore conf, mount | <2 min |
| 5. Storage hardware failure | None | PERMANENT | Unless offsite backup | N/A |

**Key facts**:
- Scenario 1 & 4: Restore `gocryptfs.conf`, re-mount directory, verify data
- Scenario 2: Password cannot decrypt without conf (contains master key)
- Scenario 3: scrypt hashing is one-way; no recovery possible
- Scenario 5: Back up both conf AND encrypted files to USB/cloud

## Recovery Contacts

**HPC Help Desk** (its-hpc@pomona.edu): Mount issues, system resources, /bigdata backups
**IT Security** (security@pomona.edu): Password compromise, data breach
**Disaster Recovery** (via its-hpc@pomona.edu): Complete /bigdata failure (24-72hr response)

## Compliance and Audit Requirements

If encrypted data contains HIPAA/FERPA/NIST requirements, document:
- System name, classification (PHI/Restricted)
- Storage location, encryption method (AES-256-GCM FIPS 140-2)
- Master key storage location and backup procedure
- Responsible person and date of backup restoration test

Create lab compliance document listing each encrypted directory with path, contents, encryption method, backup locations, and last verification date. Include annual checklist: directories documented, passwords in vault, backups to 3 locations, backups tested, rotation completed.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Implement Complete Backup Strategy

**Scenario**: You have `/bigdata/lab/<labname>/confidential_results_cipher/` containing HIPAA PHI (cannot be lost).

**Your task**:
1. Create backup directory in `/rhome/` with permissions 700
2. Back up `gocryptfs.conf` with correct permissions (600)
3. Create second timestamped backup copy
4. Test restore and mount from backup
5. Create inventory document
6. Implement password security

**Deliverables**:
- Backup files: `ls -la ~/backup/gocryptfs_*.conf`
- Successful mount test output
- Inventory document listing cipher path, contents, backup locations
- Written summary of your 3-2-1 backup strategy

:::::::::::::::::::::::::::::::::::: solution

```bash
# Create backup directory with correct permissions
mkdir -p ~/backup/gocryptfs_keys && chmod 700 ~/backup/gocryptfs_keys

# Back up gocryptfs.conf (two copies)
cp /bigdata/lab/<labname>/confidential_results_cipher/gocryptfs.conf \
   ~/backup/gocryptfs_keys/gocryptfs_confidential_results.conf
chmod 600 ~/backup/gocryptfs_keys/gocryptfs_confidential_results.conf
cp ~/backup/gocryptfs_keys/gocryptfs_confidential_results.conf \
   ~/backup/gocryptfs_keys/gocryptfs_confidential_results_$(date +%Y-%m-%d).conf

# Test restoration: mount original and backup
mkdir -p /scratch/$USER/test_original /scratch/$USER/test_backup
gocryptfs --passfile ~/.gocryptfs_pass /bigdata/lab/<labname>/confidential_results_cipher /scratch/$USER/test_original
ls /scratch/$USER/test_original  # Verify decrypted files

# Mount from backup copy
mkdir -p /tmp/test_cipher && cp ~/backup/gocryptfs_keys/gocryptfs_confidential_results.conf /tmp/test_cipher/gocryptfs.conf
echo "[password]" | gocryptfs /tmp/test_cipher /scratch/$USER/test_backup -
diff <(ls /scratch/$USER/test_original) <(ls /scratch/$USER/test_backup)  # Should match
fusermount -u /scratch/$USER/test_original /scratch/$USER/test_backup

# Store password securely
echo "[password]" > ~/.gocryptfs_pw && chmod 600 ~/.gocryptfs_pw

# Create inventory
cat > ~/Documents/encrypted_inventory.txt << 'EOF'
ENCRYPTED DIRECTORY INVENTORY | Created: 2024-04-09
Directory: Confidential Results
Path: /bigdata/lab/<labname>/confidential_results_cipher/
Contents: Patient outcome data (95 subjects)
Password: ~/.gocryptfs_pw (chmod 600), Bitwarden
Backups (3-2-1): /rhome/backup/ (Copy 1), USB home safe (Copy 2), Bitwarden cloud (Copy 3)
Last tested: 2024-04-09
EOF
```

**3-2-1 Strategy: 3 copies, 2 media types (network + USB), 1 offsite (home)**

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Create Key Management Inventory Document

**Scenario**: You manage a lab with 4 encrypted datasets: patient_study_a, patient_study_b, confidential_records, collaboration_partner.

**Your task**: Create inventory with path, contents, access, password management, 3 backup locations, compliance notes, succession plan, and disaster recovery contacts.

**Submit**:
- Complete inventory for all 4 directories
- Backup locations (3+ per directory)
- Succession plan with named successors
- Compliance and recovery contacts

:::::::::::::::::::::::::::::::::::: solution

```
LAB ENCRYPTION KEY MANAGEMENT INVENTORY
Lab: Smith Neuroscience Lab | Last Updated: 2024-04-09

DIRECTORY 1: PATIENT STUDY A
Path: /bigdata/lab/<labname>/patient_study_a_cipher/
Contents: fMRI brain imaging (500 subjects, 2500 files)
Access: PI (owner), 2 postdocs (read-only)
Password: Bitwarden "gocryptfs patient_a" (24 chars, annual rotation)
Backups: /rhome/backup/gocryptfs_patient_a.conf, USB home safe, Bitwarden
Compliance: HIPAA PHI, IRB #12345 (expires 2025-12-31), 10-year retention
Last tested: 2024-04-08

DIRECTORY 2: PATIENT STUDY B
Path: /bigdata/lab/<labname>/patient_study_b_cipher/
Contents: Clinical outcomes (300 subjects, 450 files)
Access: PI (owner+write), 1 postdoc (write), 1 student (read)
Password: Bitwarden "gocryptfs patient_b" (20 chars)
Backups: /rhome/backup/, USB, Bitwarden
Compliance: HIPAA, IRB #12346, 10-year retention

DIRECTORY 3: CONFIDENTIAL RECORDS
Path: /bigdata/lab/<labname>/confidential_records_cipher/
Contents: Financial records, grants, personnel (1200 files)
Access: PI (owner only), Dept Chair (by authorization)
Password: 1Password "gocryptfs confidential" (28 chars, higher security)
Backups: /rhome/backup/, separate USB (security segregation), 1Password
Compliance: Restricted/confidential, Finance audit required

DIRECTORY 4: COLLABORATION PARTNER
Path: /bigdata/lab/<labname>/collaboration_partner_cipher/
Contents: External partner data (320 MB, 156 files)
Access: PI, postdoc, Dr. Jane Collaborator (UC Berkeley)
Password: Bitwarden (20 chars, shared in-person, never email)
Backups: /rhome/backup/, USB (Pomona only), Bitwarden
Compliance: Collaboration agreement (UC Berkeley), expires 2025-02-01

SUMMARY TABLE
Directory    | Contents       | Access        | Backups | Compliance
patient_a    | fMRI (500)     | 3 read        | 3 locs  | HIPAA
patient_b    | Records (300)  | 1W, 2R        | 3 locs  | HIPAA
confidential  | Financial      | PI+Chair      | 3 locs  | Audit
partner      | Collab (156)   | 1 external    | 3 locs  | Agree

SUCCESSION PLAN
Primary: Dr. Carol Lee (PostDoc, c.lee@pomona.edu)
  Works with all 4 directories | Training: 2024-02-15 | Access verified: 2024-02-20
Secondary: Dr. John Martinez (Faculty, j.martinez@pomona.edu)
  Works with patient_a only | Basic training completed

Sealed Document: Pomona Bank Deposit Box #2847
  Contents: Written passwords, inventory, USB backup
  Access authorized: Dr. Carol Lee, Dept Chair
  Trigger: If PI unavailable, Chair opens & contacts Lee
  Transition: Change passwords, create new backups, Lee becomes manager

DISASTER RECOVERY CONTACTS
HPC Help Desk (its-hpc@pomona.edu): System resources, permissions, backups
IT Security (security@pomona.edu): Password compromise, data breach
Disaster Recovery (via its-hpc@pomona.edu): /bigdata failure (24-72hr response)
Compliance (via its-hpc@pomona.edu, who will route to the IRB/compliance office): IRB implications
Partner Lab (j.collab@berkeley.edu): Shared data issues

ANNUAL AUDIT CHECKLIST (Next: 2025-04-09)
□ All directories documented  □ Passwords in vault (no plain copies)
□ Backups to 3 locations      □ Backups tested & working
□ Passwords rotated           □ No departed members in access
□ Backups secured             □ Succession plan current

Signed: _________________ Date: ________ (Lab Director)
Reviewed: ________________ Date: ________ (Dept Chair, annually)
```

::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Establish clear procedures for handling encryption when lab members leave (test access, change passwords, backup updates).
- Create succession plans with named successors and sealed documents in safe locations before they're needed.
- Maintain inventory of all encrypted directories with backup locations, access controls, and compliance notes.
- Implement automated backup scripts and add them to cron for monthly execution.
- Test backup restoration regularly to verify recovery procedures actually work.
- Know the five disaster scenarios: (1) have conf + password (recover), (2) lost conf no backup (permanent loss), (3) forgot password (permanent loss), (4) lost conf with backup (recover), (5) storage failure (permanent unless offsite backup).
- Document compliance requirements (NIST SP 800-171, HIPAA, IRB) alongside encryption procedures.
- Keep a recovery contact list for technical, security, and compliance issues.
- Review encryption management and inventory annually with department leadership.
- Never store passwords unencrypted; use password manager or sealed physical safe only.
::::::::::::::::::::::::::::::::::::::::::::::::

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
