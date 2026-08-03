---
title: Instructor Notes
---

# gocryptfs Encryption for Research Data - Instructor Notes

## Workshop Overview

**Title:** gocryptfs Encryption for Research Data at Pomona College's Sagehen HPC Cluster

**Duration:** 6.5 hours total
- Teaching: ~230 minutes (across 17 episodes)
- Hands-on exercises: ~155 minutes (across 17 episodes)
- Breaks and discussion: ~35 minutes

**Format:** Interactive hands-on workshop with live demonstrations, individual exercises, and collaborative troubleshooting. Episodes can be delivered as:
- **Two half-day sessions:** Day 1 (Sections 1–3, 3 hours); Day 2 (Sections 4–7, 3.5 hours)
- **Three shorter sessions:** Sessions 1 (Sections 1–2), Session 2 (Sections 3–4), Session 3 (Sections 5–7)
- **Full-day sprint:** All 17 episodes with lunch break in middle

**Audience:** Researchers, graduate students, postdocs, and lab managers working with RESTRICTED or PROPRIETARY data on Sagehen HPC

**Prerequisites:**
- Active Sagehen HPC account
- Completion of Workshop 13 (HPC Security Orientation)
- Completion of Workshop 14 (Data Classification and Handling) *strongly recommended*
- Basic command-line familiarity (Workshop 1: The Unix Shell)
- Understanding of data classification tiers (PUBLIC, PROPRIETARY, RESTRICTED)

**Learning Outcomes:** By the end of this workshop, participants will be able to:
1. Explain the fundamental concepts behind encryption and gocryptfs architecture
2. Assess when encryption is mandatory for their research data
3. Create, mount, and unmount encrypted directories on Sagehen
4. Integrate encrypted vaults into active research workflows and SLURM job scripts
5. Implement secure key management and backup strategies for encrypted data
6. Troubleshoot common gocryptfs issues and performance considerations
7. Apply best practices for sharing encrypted data with collaborators

---

## Pre-Workshop Instructor Preparation

### Technical Setup Checklist

- [ ] **Access verification:** Confirm your own Sagehen account access; test gocryptfs module availability (`module load gocryptfs; gocryptfs --version`)
- [ ] **Demo vaults:** Create 3–4 pre-initialized demo encrypted vaults on `/bigdata/demo` for live demonstrations (use temporary demo-only passwords, document separately)
- [ ] **Test commands:** Verify mounting/unmounting work reliably; test both interactive and non-interactive (password-from-file) mounting
- [ ] **File system layout:** Confirm understanding of Sagehen storage: `/rhome` (home), `/bigdata` (shared lab), `/scratch` (non-persistent), `/tmpfs` (RAM-backed)
- [ ] **Quota checks:** Verify demo account has 5–10 GB free on `/bigdata` for participant exercises
- [ ] **Live demo hardware:** Large monitor or projector; secondary screen ideal for simultaneous terminal + documentation display
- [ ] **SLURM job testing:** Pre-test sample scripts covering job arrays, GPU jobs, and pipeline scenarios
- [ ] **Key backup/recovery:** Test archiving encrypted folders (`tar` with gocryptfs.conf), test restoration workflow
- [ ] **Failure scenarios:** Pre-stage backup screenshots/recordings of all live demos
- [ ] **Compliance reference:** Have quick access to NIST SP 800-171, FERPA, HIPAA, EAR/ITAR standards during teaching

### Materials Preparation

- [ ] **Slides/presentation:** Workflow diagrams, encryption/decryption visualizations, data flow diagrams, architecture diagrams (especially FUSE concept)
- [ ] **Reference card:** Printed/digital quick reference with gocryptfs commands, password requirements, troubleshooting steps
- [ ] **Example data files:** Small CSV with synthetic sensitive data (patient IDs, blood pressure, medications)
- [ ] **Sample SLURM scripts:** Working examples for basic jobs, job arrays, GPU jobs, and pipeline scenarios
- [ ] **Troubleshooting guide:** 1-page reference for common errors and solutions to distribute
- [ ] **Assessment checklist:** Printable checklist for participants to verify progress during/after workshop
- [ ] **Follow-up resources:** List of recommended next workshops, documentation links, contact info (its-hpc@pomona.edu)
- [ ] **Episode-specific handouts:** One-page summaries for Sections 3 (Planning), 5 (SLURM templates), 6 (Key management)

### Knowledge Requirements for Instructor

You should be familiar with:

- **gocryptfs command syntax:** `-init`, mount operation (options, output format), unmount (`fusermount -u`), both interactive and non-interactive modes
- **Sagehen directory structure and quotas:** Storage locations, purpose, quota limits, how to check quota
- **Data classification and compliance:** When RESTRICTED data requires encryption, FERPA/HIPAA/EAR/ITAR basics, Pomona ITS Policy 24
- **Encryption fundamentals:** AES-256-GCM properties, Argon2 memory-hard design, why strong passwords matter
- **FUSE (Filesystem in Userspace):** Basic concept (virtual filesystem layer), `fusermount` command, how mounting works
- **Common gocryptfs errors:** "Bad password," "Already mounted," "Permission denied," "Disk quota exceeded," how to diagnose and fix
- **SLURM integration patterns:** When to mount (job start), when to unmount (job cleanup), trap patterns, error handling
- **Password management best practices:** 14+ character requirements (NIST SP 800-63B), password manager recommendations, secure sharing
- **OnDemand portal:** Navigation, terminal access, file manager usage for live demos
- **Performance benchmarks:** Typical overhead (5–15%), when to optimize, tuning options
- **Job array and GPU considerations:** How encryption impacts job arrays, GPU memory, batched processing
- **Disaster recovery:** Backup restoration, 3-2-1 rule, password recovery impossibility, lab transitions

---

## Episode-by-Episode Teaching Guide

### SECTION 1: Why Encryption Matters (35 min teaching + 20 min exercises)

#### Episode 01: Why Encrypt? Regulations and Risk (15 min teaching + 5 min exercises)

**Learning Outcome:** Participants understand mandatory vs. recommended encryption and recognize when their data needs protection.

**Key Concepts:**
- RESTRICTED data (HIPAA, FERPA, genetic, classified) *must* be encrypted
- Encryption protects against unauthorized access, including by admins and after physical theft
- Real breach scenarios: cost, legal liability, institutional damage
- Password is the only key; losing it makes data permanently inaccessible

**Teaching Approach:**

1. **Real-world breach scenario (5 min):** Use the HIPAA medical research example (150 participants, laptop theft, thief gains access to unencrypted data). Outline consequences: participant notification, lawsuits, institutional costs ($2–5M), reputational damage, loss of active grants. Then: "With encryption, the thief gets gibberish. The breach is contained."

2. **When encryption is mandatory (7 min):** Show classification tier table (PUBLIC: no encryption needed; PROPRIETARY: recommended; RESTRICTED: mandatory). Walk through each mandatory case:
   - HIPAA health records → federal privacy law
   - FERPA student records → educational privacy law
   - Genetic data with IDs → re-identification risk, NIH policy
   - Classified information (CUI, EAR/ITAR) → government contract requirements
   - IRB-approved human subjects research → IRB protocol requirement

3. **Transition (3 min):** "gocryptfs makes encryption practical. You'll leave here with an encrypted vault and confidence to use it."

**Common Pitfalls:**
- Researchers assume permissions (chmod 700) are sufficient—clarify that admins can still read files
- Underestimating breach cost—use real institutional examples
- Confusing recommendation with requirement—be explicit about RESTRICTED = mandatory

**Teaching Tips:**
- Start emotional: breach scenarios resonate more than abstract security
- Make personal: use examples from your institution
- Distinguish mandatory vs. recommended clearly
- End optimistically: "This is doable."

**Timing Guidance:**
- Spend 5 min on scenario (worth it for buy-in)
- Emphasize mandatory cases; brief on optional
- Save Q&A for end of episode

#### Episode 02: Pomona's Data Classification System (10 min teaching + 5 min exercises)

**Learning Outcome:** Participants can classify their own data and determine encryption requirements.

**Key Concepts:**
- Three-tier system: PUBLIC, PROPRIETARY, RESTRICTED
- gocryptfs protections align with RESTRICTED tier
- Classification guides decisions: storage location, encryption, access control, retention
- Re-assessment during project lifecycle

**Teaching Approach:**

1. **Classification framework (4 min):** Define each tier with examples and encryption status:
   - PUBLIC: papers, course materials, open-source code → no encryption
   - PROPRIETARY: pre-publication algorithms, grant proposals, trade secrets → recommended encryption
   - RESTRICTED: health records, student data, genetic IDs, classified → **mandatory encryption**

2. **How gocryptfs fits (3 min):** gocryptfs is the tool you use to meet RESTRICTED encryption requirement. It handles AES-256-GCM and password management transparently.

3. **Practical exercise (3 min):** Show 5 realistic data examples (patient IDs, grant proposals, published data, DNA sequences, student grades). Ask: "What tier? What protection?" Drive discussion toward correct classification.

**Common Pitfalls:**
- Researchers uncertain about their own data → provide a simple self-assessment worksheet
- Conflation of PUBLIC and non-sensitive → clarify PUBLIC includes open-source and published data
- Overly restrictive classification → explain that PROPRIETARY captures most pre-publication work

**Teaching Tips:**
- Use institution-specific examples (your grants office, IRB, compliance office can provide real data examples)
- Emphasize: "When in doubt, ask your PI. But RESTRICTED is always mandatory."
- Leave time for Q&A; this is where misconceptions surface

**Timing Guidance:**
- Spend time on RESTRICTED definition (most important)
- Move quickly through PUBLIC
- Exercise is low-risk practice for classification judgment

#### Episode 03: Scenarios and Responsibilities (10 min teaching + 10 min exercises)

**Learning Outcome:** Participants understand roles, responsibilities, and breach response for encrypted data.

**Key Concepts:**
- Different roles (PI, postdoc, manager, IT) have different responsibilities
- Scenario-based decision-making: Should you encrypt? Who owns the password?
- Incident response: What happens if someone accesses encrypted data?
- Documentation: Who needs to know about encrypted vaults?

**Teaching Approach:**

1. **Four scenario discussion (10 min, interleaved with teaching):**

   - **Scenario A:** Postdoc has 50 patient samples with genetic IDs. Stores on `/bigdata/lab/genetics`. Q: "Does this data need encryption? Why? Who should manage the password—postdoc or PI? What if postdoc leaves?"
     - Answer: Yes, RESTRICTED. Postdoc should hold password; PI should have emergency access. Document succession plan.

   - **Scenario B:** Graduate student working on pre-publication machine-learning model. Wants to back up code to personal laptop. Q: "PROPRIETARY data on personal device—how to protect?"
     - Answer: Encrypt on laptop too. Use same password manager. Keep main copy on Sagehen.

   - **Scenario C:** Lab manager oversees encrypted data for multiple postdocs. One postdoc's encrypted vault gets corrupted (mount fails). Q: "What's the recovery process? Who has backup copies?"
     - Answer: Must have 3-2-1 backup rule (3 copies, 2 locations, 1 offsite). If original vault corrupted, restore from backup. No password recovery possible.

   - **Scenario D:** PI publishes paper; postdoc's contract ends. Q: "What happens to encrypted data? Can postdoc take it?"
     - Answer: RESTRICTED data stays institutional. Postdoc can't take decrypted copy. PI archives encrypted vault per 7-year retention rule. Data remains encrypted indefinitely.

2. **Responsibility matrix (3 min):** Present a simple table:

   | Role | Encryption Decision | Password Management | Monitoring | Backup |
   |------|--------------------|--------------------|-----------|--------|
   | **PI** | Approves; ensures mandate is met | Aware of existence; emergency access | Ensures compliance | Ensures 3-2-1 rule |
   | **Postdoc/Grad Student** | Implements gocryptfs | Day-to-day access | Reports issues | Local backup |
   | **Lab Manager** | Supports setup; documents | Tracks access; succession planning | Monitors usage | Enforces institutional backup |
   | **IT (ITS)** | Provides tools; support | No access; password encrypted | Monitors system security | Storage-level backup |

3. **Breach response (2 min, brief overview):** If encrypted data is breached:
   - **If encrypted:** Assess impact. If password not compromised, breach may be contained. Document and notify compliance office.
   - **If decrypted data accessed:** Follow IRB/compliance protocol; may require participant notification, regulatory reporting.
   - Report to its-hpc@pomona.edu immediately.

**Common Pitfalls:**
- PIs don't realize they need emergency access to postdocs' encrypted data → emphasize succession planning
- Lab managers assume IT manages all passwords → clarify password is researcher's responsibility
- Overconfidence in single backup → stress 3-2-1 rule

**Teaching Tips:**
- Use real scenarios from your institution (anonymized)
- Emphasize: "Encryption doesn't remove responsibility; it changes how you manage responsibility."
- Discuss incident response matter-of-factly, not alarmingly

**Timing Guidance:**
- Scenario discussion is the core; don't rush
- Responsibility matrix is reference material; don't dwell
- Save incident response for Q&A if time is tight

---

### SECTION 2: How gocryptfs Works (30 min teaching + 15 min exercises)

#### Episode 04: gocryptfs Architecture (15 min teaching + 5 min exercises)

**Learning Outcome:** Participants understand the two-directory model and how FUSE makes encryption transparent.

**Key Concepts:**
- gocryptfs creates two directories: ciphertext (encrypted on disk) and plaintext (decrypted in RAM, mounted)
- FUSE (Filesystem in Userspace) acts as intermediary
- Mounting/unmounting controls when decrypted access is available
- Architecture enables on-the-fly encryption/decryption

**Teaching Approach:**

1. **History and context (3 min):** gocryptfs is open-source, maintained, audited. Used by researchers at multiple institutions. Alternatives include VeraCrypt, EncFS (older, less secure), LUKS (block-level). gocryptfs is practical for per-directory encryption without admin privileges.

2. **Two-directory model (7 min):** Draw or display a diagram:
   ```
   User's view (plaintext, mounted):
   /home/alice/projects/data_vault/  <- mount point (decrypted, readable)
      patient_data.csv
      analysis_code.py
   
   On disk (ciphertext, encrypted):
   /bigdata/lab/vault.gocryptfs/      <- physical storage (encrypted)
      gocryptfs.conf
      xyz123abc...                      <- encrypted files (gibberish)
      def456...
   ```
   Explain:
   - User interacts with mounted plaintext directory
   - Underlying physical storage is encrypted
   - FUSE layer handles encryption/decryption on reads/writes
   - Mount is voluntary; unmounting hides decrypted data

3. **FUSE mechanics (4 min):** Simplify:
   - User reads file from `/home/alice/vault/patient_data.csv`
   - FUSE intercepts read → asks gocryptfs to decrypt
   - gocryptfs retrieves encrypted file from `/bigdata/lab/vault.gocryptfs`, decrypts using password, returns plaintext
   - User sees plaintext; disk remains encrypted
   - When unmounted, plaintext access goes away; decrypted data is no longer in RAM

4. **Security implication (1 min):** "Once you unmount, data is encrypted again. Even if someone hacks your account while vault is unmounted, they see gibberish."

**Common Pitfalls:**
- Researchers assume "encrypted vault" means all access is encrypted → clarify that mounted plaintext is not encrypted
- Confusion between file permissions and encryption → emphasize encryption is independent
- Thinking unmounting deletes data → clarify unmounting only stops access, data remains on disk

**Teaching Tips:**
- Use diagram; visual is crucial
- Compare to analogy: password manager (encrypted vault, decrypt on open, re-encrypt on close)
- Emphasize transparency: users don't write encryption code; gocryptfs handles it

**Timing Guidance:**
- Diagram + explanation: 7 min
- History/alternatives: keep brief (3 min)
- FUSE mechanics: explain clearly (4 min); use analogies if audience is non-technical

#### Episode 05: Encryption, Performance, and Comparison (15 min teaching + 10 min exercises)

**Learning Outcome:** Participants understand encryption algorithms, performance trade-offs, and why gocryptfs is the right choice.

**Key Concepts:**
- AES-256-GCM: NIST-approved, strong, authenticated encryption
- Argon2: memory-hard key derivation, resistant to brute-force
- Typical overhead: 5–15% on disk I/O
- Alternatives exist; gocryptfs balances security, usability, performance

**Teaching Approach:**

1. **AES-256-GCM (5 min):**
   - AES: Advanced Encryption Standard, symmetric (same key for encrypt/decrypt)
   - 256-bit key: astronomically large keyspace (2^256 ≈ 1.15 × 10^77 possible keys)
   - GCM (Galois/Counter Mode): authenticated encryption; detects tampering
   - NIST-approved, FIPS 140-2 compliant
   - No known practical attacks on AES-256
   - High-level math: not required for users; emphasis is on "NIST says this is strong"

2. **Argon2 (3 min):**
   - Key derivation function: converts password to cryptographic key
   - Memory-hard design: requires significant RAM, resists GPU/ASIC brute-force
   - Password can be relatively short (e.g., 14 chars); Argon2 stretches it into strong key
   - Tuning: gocryptfs defaults are good; advanced users can adjust time/memory cost

3. **Performance (4 min):** Benchmark results (on typical Sagehen hardware):
   - Unencrypted read: 1000 MB/s
   - Encrypted read (gocryptfs): 850–950 MB/s (5–15% overhead)
   - Overhead is negligible for most research workflows (data analysis, text processing, not real-time video)
   - Encrypted backups: similar overhead
   - Mount/unmount are instant (sub-second)

4. **Alternatives comparison (3 min, table):**

   | Tool | Encryption | Ease of Use | Performance | Best For |
   |------|------------|------------|-------------|----------|
   | **gocryptfs** | AES-256-GCM + Argon2 | High | 90–95% | Per-directory, user-friendly |
   | **VeraCrypt** | AES (multiple modes) | Medium | 85–90% | Full-disk or container encryption |
   | **LUKS** | AES-256 | Low (admin only) | 95%+ | Linux full-disk encryption (system level) |
   | **EncFS** | AES (older) | High | 80–85% | Legacy; not recommended for new projects |

   Conclusion: "gocryptfs is the practical choice for per-directory encryption without admin privileges."

**Common Pitfalls:**
- Researchers worry encryption will make work unusable → performance overhead is negligible
- Misconception that AES can be broken → explain no practical attacks known
- Comparison anxiety ("Is gocryptfs the best?") → clarify it's the best for this use case

**Teaching Tips:**
- Benchmark graphs help: show actual numbers, not abstract percentages
- Admit limitations: "Encryption adds a tiny bit of overhead; security is worth it"
- For non-technical audience: emphasize NIST approval, not mathematical details

**Timing Guidance:**
- AES-256-GCM: 5 min (enough detail for confidence)
- Argon2: 3 min (brief overview sufficient)
- Performance: 4 min (concrete numbers matter)
- Alternatives: 3 min (context, not deep dive)

---

### SECTION 3: Hands-On Setup (30 min teaching + 25 min exercises)

#### Episode 06: Planning Your Encrypted Directories (15 min teaching + 10 min exercises)

**Learning Outcome:** Participants plan vault layout, naming, password strategy, and initialization parameters before creation.

**Key Concepts:**
- Naming conventions: meaningful, consistency across lab
- Storage location: ciphertext on `/bigdata`, plaintext mount point in `/rhome`
- Password requirements: 14+ characters, no dictionary words, password manager storage
- Initialization parameters: reverse mode, plain names vs. encrypted names

**Teaching Approach:**

1. **Naming strategy (4 min):** Guide participants through naming decisions:
   - Ciphertext vault directory (on-disk storage): `/bigdata/lab/vault_PROJECTNAME_YEAR`
     - Example: `/bigdata/lab/vault_cardio_2026`
     - Include project name + year; easy to organize and archive
   - Mount point (decrypted access): `/rhome/alice/PROJECTNAME_data`
     - Example: `/rhome/alice/cardio_data`
     - Shorter, obvious purpose, in home directory for quick access
   - Consistent naming across lab enables handoff and documentation

2. **Storage location principles (4 min):**
   - Ciphertext vault (encrypted on-disk): `/bigdata/lab/` (backed up, shared lab storage)
   - Mount point (plaintext access): personal directory (e.g., `/rhome/alice/` or `/scratch`)
   - Never mount on shared paths (e.g., `/bigdata/lab/`) → would expose decrypted data to all lab members
   - Mount point can be temporary (`/scratch` project directory) if vault is short-lived
   - Home directory is best default (private, backed up, persistent)

3. **Password strategy (4 min):**
   - Requirement: 14+ characters, uppercase, lowercase, numbers, symbols
   - Generation: Use password manager (Bitwarden, 1Password, LastPass) to generate and store
   - Never use: birthdays, names, dictionary words, keyboard patterns (qwerty)
   - Never share: Keep in password manager; only PI knows for emergency access
   - Never write down: Exception for emergency backup (encrypted storage)
   - Example strong password: `Sg42!mK9@xQp2v` (random, 14 chars, mixed case)

4. **Initialization parameters (3 min):**
   - **Reverse mode:** `-reverse` flag mounts plaintext side, keeps ciphertext in RAM. Rare use case; not recommended for beginners.
   - **Plain names vs. encrypted names:** Default (`-plaintextnames=false`) encrypts filenames. Advanced users can use `-plaintextnames=true` if filename privacy not needed (small performance gain).
   - For this workshop: use defaults (plain encrypted filenames, reverse=false)

**Hands-On Exercise (10 min):** Participants plan their own vault:
1. Decide on project/data purpose
2. Choose ciphertext path: `/bigdata/lab/vault_PROJECTNAME_YEAR`
3. Choose mount point: `/rhome/alice/PROJECTNAME_data`
4. Generate strong password in Bitwarden/1Password
5. Write down plan on worksheet (to be reviewed before initialization)
6. Instructor circulates, validates naming, offers feedback

**Common Pitfalls:**
- Mount point on `/bigdata` (would expose plaintext data) → enforce home directory default
- Weak passwords (dictionary words, names) → show password manager generation
- Unclear project naming → provide template

**Teaching Tips:**
- Worksheet helps formalize the plan; reduces mistakes later
- Password manager demo saves time (show generating password in Bitwarden)
- Validate before initialization; prevents re-initialization

**Timing Guidance:**
- Planning is crucial; don't rush (15 min teaching justified)
- Exercise is real work; expect 10 min of active thinking
- Collect worksheets; use for next episode (initialization)

#### Episode 07: Verifying and Managing Encrypted Directories (15 min teaching + 15 min exercises)

**Learning Outcome:** Participants initialize vaults, verify setup, manage basic operations, and recover from common mistakes.

**Key Concepts:**
- Initialization command and output interpretation
- Verification: mount, check contents, unmount
- Permissions: vault ownership, mount point permissions, data file access
- Common mistakes: re-initialization, wrong password, permission errors
- Backup of gocryptfs.conf: essential for recovery

**Teaching Approach:**

1. **Initialization command (4 min):** Live demo with participants following along:
   ```bash
   gocryptfs -init /bigdata/lab/vault_cardio_2026
   ```
   - Prompts for password (twice for confirmation)
   - Creates gocryptfs.conf (encrypted config file)
   - Creates master key (also in gocryptfs.conf, encrypted)
   - Output: "The filesystem has been successfully created."
   - Time: ~5 seconds (Argon2 derivation is intentionally slow)

2. **First mount (3 min):**
   ```bash
   gocryptfs /bigdata/lab/vault_cardio_2026 /rhome/alice/cardio_data
   ```
   - Prompts for password
   - Mount succeeds silently (good sign)
   - Verify with `ls /rhome/alice/cardio_data/` (shows empty directory)
   - Check with `df` to confirm mount point is active

3. **Adding test file (2 min):**
   ```bash
   echo "test data" > /rhome/alice/cardio_data/test.txt
   cat /rhome/alice/cardio_data/test.txt
   ```
   - File is readable from plaintext side
   - Show corresponding encrypted file in ciphertext directory:
   ```bash
   ls /bigdata/lab/vault_cardio_2026/ (shows encrypted gibberish)
   ```

4. **Unmounting (2 min):**
   ```bash
   fusermount -u /rhome/alice/cardio_data
   ```
   - Unmount succeeds silently
   - Verify with `ls /rhome/alice/cardio_data/` (directory is empty again; mounted filesystem gone)
   - Emphasis: "Data is still encrypted on disk; mounting made it temporarily accessible."

5. **Permissions and security (4 min):**
   - Vault ownership: `ls -ld /bigdata/lab/vault_cardio_2026/` should show researcher as owner
   - Mount point permissions: `ls -ld /rhome/alice/cardio_data/` should show `700` (only owner can access)
   - Data files in mounted vault: user-readable (because owner mounted it)
   - Anti-pattern: never mount on world-readable path (e.g., `/tmp`)
   - Anti-pattern: never `chmod 777` the mount point after mounting

6. **Common mistakes and recovery (4 min):**

   - **Mistake 1: Wrong password on mount**
     - Error: "Bad password"
     - Recovery: Check password in password manager; try again. If lost, vault is permanently inaccessible.
   
   - **Mistake 2: Attempting to re-initialize**
     - Error: "gocryptfs.conf already exists"
     - Recovery: Don't re-initialize. If vault is corrupted, delete `/bigdata/lab/vault_PROJECTNAME_YEAR` and start over (losing data unless backed up).
   
   - **Mistake 3: Mount point doesn't exist**
     - Error: "mount point doesn't exist"
     - Recovery: `mkdir -p /rhome/alice/cardio_data` first, then mount.
   
   - **Mistake 4: Permission denied when accessing mounted files**
     - Error: "Permission denied"
     - Cause: Mount point owned by different user
     - Recovery: Ensure researcher mounted vault (not sudo), verify `ls -ld` shows correct owner.
   
   - **Mistake 5: Vault already mounted**
     - Error: "Device or resource busy"
     - Recovery: `fusermount -u /path/to/mount` to unmount; retry initialization or mount.

7. **Backup of gocryptfs.conf (2 min):** Critical step often forgotten:
   - gocryptfs.conf is encrypted; contains master key (encrypted with password)
   - If vault.gocryptfs directory is deleted/corrupted, you need gocryptfs.conf to restore
   - **Action:** Copy gocryptfs.conf to secure location (password manager attachment, external drive, institutional backup)
   - Emphasize: "If you lose gocryptfs.conf AND forget password, data is gone forever."

**Hands-On Exercise (15 min):** Participants initialize their own vault, test mount/unmount, verify setup:
1. Open terminal on Sagehen (OnDemand)
2. Initialize vault using plan from Episode 06
3. Mount vault
4. Add test file
5. Verify on disk that file is encrypted
6. Unmount
7. Verify unmounted (mount point empty)
8. **Backup gocryptfs.conf to external location** (password manager or personal USB drive)
9. Instructor checks each participant's setup; validates successful mount/unmount

**Common Pitfalls:**
- Forgetting to create mount point → catch early (step 1 of exercise)
- Mount on world-readable path → emphasize home directory in planning
- Lost password before backup → reinforce backup step
- Re-initialization attempts → clarify this destroys data

**Teaching Tips:**
- Live demo first (10 min); participants feel more confident
- Exercise is hands-on repetition; essential for retention
- Instructor circulates during exercise; catch mistakes before vault is full of data
- Celebrate first successful mount/unmount; builds confidence

**Timing Guidance:**
- Teaching is detailed but necessary (15 min justified)
- Exercise is active learning (15 min justified)
- Expect some troubleshooting during exercise; leave buffer

---

### SECTION 4: Daily Operations (25 min teaching + 20 min exercises)

#### Episode 08: Daily Mount and Unmount Workflow (15 min teaching + 10 min exercises)

**Learning Outcome:** Participants develop secure mount/unmount habits and understand security implications of long-lived mounts.

**Key Concepts:**
- Typical workflow: mount when working, unmount when done
- Mount anatomy: password prompt, verification, successful access
- Security best practice: unmount when leaving workstation, at end of day
- Session management: SSH sessions, long-running processes

**Teaching Approach:**

1. **Daily workflow (5 min):** Typical researcher's day:
   - Morning: Arrive at office, open terminal, mount vault
   - Work: Read/write encrypted data (via mounted plaintext directory)
   - Break/Lunch: Unmount to prevent casual access during absence
   - Afternoon: Mount again, continue work
   - Evening: Unmount before leaving office (security best practice)

2. **Mount anatomy (5 min):** Step through mount command and output:
   ```bash
   gocryptfs /bigdata/lab/vault_cardio_2026 /rhome/alice/cardio_data
   Password: [hidden input]
   Mounted successfully. Use 'fusermount -u /rhome/alice/cardio_data' to unmount.
   ```
   - Password is not echoed (good security practice)
   - Prompt appears in terminal; user inputs password
   - Successful mount returns to prompt immediately (no error = success)
   - Command is not interactive after mounting (can background with `&` if desired)

3. **When NOT to leave mounted (4 min):**
   - Leaving workstation unattended: unmount before stepping away
   - After research session: unmount to minimize exposure window
   - During breaks/lunch: unmount (reduces risk if someone accesses your account)
   - Overnight: unmount (no need for persistent access; reduces attack surface)
   - When sharing workstation: unmount before handing to another user

4. **SSH and remote mounts (2 min):** (Brief overview; detailed in Section 4)
   - If SSHing into Sagehen from laptop, mounting a vault is possible but requires password entry
   - For persistent SLURM jobs, password is stored in file (see Episode 09)
   - SSH best practice: use key-based auth, not password auth

5. **Unmounting (1 min):**
   ```bash
   fusermount -u /rhome/alice/cardio_data
   ```
   - Command succeeds silently (good sign)
   - Verify: `ls /rhome/alice/cardio_data/` shows empty directory
   - If files are open in that directory, unmount may fail with "Device or resource busy"

**Hands-On Exercise (10 min):** Participants practice mount/unmount cycle multiple times:
1. Mount vault (using password from password manager)
2. Create a few files (e.g., `data1.txt`, `data2.txt`)
3. Unmount
4. Verify mounted directory is empty
5. Mount again (password from password manager)
6. Verify files are still there (encryption works across mount cycles)
7. Unmount
8. Repeat cycle 2–3 times (builds muscle memory, demystifies process)

**Common Pitfalls:**
- Passwords typed incorrectly (especially if copy-paste fails) → emphasize careful password entry
- Forgetting unmount command → provide reference card
- "Device busy" error during unmount (files still open) → explain cause, show how to find open processes (`lsof`)

**Teaching Tips:**
- Repetition builds confidence; multiple mount/unmount cycles are worth time
- Emphasize the ease: password, access, unmount. Simple routine.
- Security is about habits, not features; daily unmounting is a habit to build

**Timing Guidance:**
- Workflow context: 5 min
- Mount anatomy: 5 min (detailed; participants need to understand prompts and outputs)
- When to unmount: 4 min (security emphasis)
- Exercise: 10 min (repetition valuable)

#### Episode 09: Non-Interactive Mounting and Troubleshooting (10 min teaching + 10 min exercises)

**Learning Outcome:** Participants understand password-from-file mounting, common errors, and how to troubleshoot.

**Key Concepts:**
- Interactive mounting: password prompt in terminal (default, secure for manual use)
- Non-interactive mounting: password from file (necessary for SLURM jobs, automated workflows)
- Security of password file: restricted permissions (400), stored in secure location
- Common gocryptfs errors: "Bad password," "Already mounted," "Permission denied," "No space left on device"
- Troubleshooting workflow: error interpretation, checking filesystem state, recovery

**Teaching Approach:**

1. **Non-interactive mounting (4 min):** Use case: SLURM job script needs to mount vault without user interaction.
   ```bash
   gocryptfs -extpass "cat /rhome/alice/.vault_password" \
     /bigdata/lab/vault_cardio_2026 /rhome/alice/cardio_data
   ```
   - `-extpass` flag: specify command that outputs password
   - `cat /rhome/alice/.vault_password`: password stored in file
   - Vault mounts without prompt (job script continues)
   - Security: file must have restrictive permissions (`chmod 400`)
   - Warning: storing plaintext password on disk is a trade-off; acceptable for SLURM jobs if file permissions are strict

2. **Password file security (3 min):**
   - Create password file:
     ```bash
     echo "your_strong_password_here" > /rhome/alice/.vault_password
     chmod 400 /rhome/alice/.vault_password
     ```
   - Permissions: `400` means only owner can read, no write
   - Location: home directory (private), not `/tmp` (world-readable)
   - Backup: password manager also holds copy; file is ephemeral

3. **Common errors and recovery (3 min):**

   - **"Bad password":**
     - Cause: Incorrect password entered (or password file contents wrong)
     - Recovery: Verify password in password manager; retype carefully; check file contents with `cat`

   - **"Already mounted":**
     - Cause: Vault already mounted at target mount point
     - Recovery: `fusermount -u /path` to unmount first; then retry mount

   - **"Permission denied":**
     - Cause: Mount point owned by different user, or wrong permissions on gocryptfs.conf
     - Recovery: Verify ownership (`ls -ld /path`); re-create mount point if needed

   - **"No space left on device":**
     - Cause: Disk quota exceeded or `/bigdata` full
     - Recovery: Check quota (`quota`); clean up old files; request quota increase from ITS

   - **"Stale NFS file handle":**
     - Cause: (Rare) NFS issue on mounted vault
     - Recovery: Unmount and remount vault

4. **Troubleshooting workflow (1 min):**
   - Error message → read carefully; identify issue (password, permissions, space, already mounted)
   - Check state: `df` (mounted filesystems), `ls -ld` (permissions), `quota` (disk usage)
   - Attempt fix (remount, fix permissions, clean up space)
   - Retry operation
   - If stuck: contact its-hpc@pomona.edu with error message and context

**Hands-On Exercise (10 min):** Participants test non-interactive mounting and encounter/resolve an error:
1. Create password file: `echo "password_here" > /rhome/alice/.vault_password`
2. Set permissions: `chmod 400 /rhome/alice/.vault_password`
3. Unmount vault (if currently mounted): `fusermount -u /rhome/alice/cardio_data`
4. Mount using password file: `gocryptfs -extpass "cat /rhome/alice/.vault_password" /bigdata/lab/vault_cardio_2026 /rhome/alice/cardio_data`
5. Verify mount succeeded: `ls /rhome/alice/cardio_data/` (shows files)
6. Deliberately cause an error (e.g., change password file to wrong value, try to remount)
7. Interpret error message
8. Recover (fix password file, unmount, remount correctly)
9. Cleanup: remove password file (`rm /rhome/alice/.vault_password`)

**Common Pitfalls:**
- Password file with weak permissions (chmod 644) → catch early, explain 400 requirement
- Password file left in place after testing → cleanup step is important
- Over-reliance on password file for manual use → clarify this is for automation only

**Teaching Tips:**
- Introduce non-interactive mounting as a bridge to SLURM (coming in Section 5)
- Error scenario is not punitive; it's learning
- Troubleshooting workflow is a general skill applicable to all gocryptfs issues

**Timing Guidance:**
- Non-interactive mounting explanation: 4 min (detailed; participants need clarity)
- Password file security: 3 min (critical for SLURM safety)
- Common errors: 3 min (overview; detailed coverage in troubleshooting section)
- Exercise: 10 min (intentional error + recovery builds confidence)

---

### SECTION 5: SLURM Integration (45 min teaching + 30 min exercises)

#### Episode 10: SLURM Integration: Decision Framework (15 min teaching + 5 min exercises)

**Learning Outcome:** Participants understand when and how to integrate encryption into HPC workflows and decide between pre-mount vs. mount-in-script strategies.

**Key Concepts:**
- Pre-mount strategy: Researcher mounts vault before submitting SLURM job
- Mount-in-script strategy: SLURM job mounts vault at start, unmounts at end
- Decision factors: Job duration, concurrent jobs, data persistence, security
- Password handling in scripts: secure file storage, restricted permissions, cleanup

**Teaching Approach:**

1. **Two strategies overview (5 min):**

   **Strategy A: Pre-mount (Simple)**
   - Researcher: `gocryptfs /bigdata/lab/vault /rhome/alice/vault` (interactive, password prompt)
   - SLURM job: Uses mounted directory (e.g., reads from `/rhome/alice/vault/data.csv`)
   - Unmount: Researcher does manually after job completes
   - Pros: Easy, secure (no password in script), works for single short jobs
   - Cons: Requires manual action; not practical for long-running or many jobs

   **Strategy B: Mount-in-script (Automated)**
   - SLURM job: Mounts vault at start using password from file
   - Job script: Runs analysis, reads/writes encrypted data
   - SLURM job: Unmounts vault at end (cleanup)
   - Pros: Fully automated; suitable for batch processing, job arrays
   - Cons: Password stored in file (if permissioned correctly, acceptable trade-off); slightly more complex script

2. **Decision framework (5 min):** Table to guide choice:

   | Scenario | Strategy | Why |
   |----------|----------|-----|
   | One job, I'll wait for output | Pre-mount | Simplest; no script complexity |
   | Job runs 30 min, I'm away | Pre-mount (leave mounted) | Convenient; I don't need to interact |
   | Many jobs, same encrypted data | Mount-in-script | Automation; avoid manual mounting |
   | Long-running background job (hours/days) | Mount-in-script | Clean separation; auto-unmount |
   | GPU job with encrypted data | Mount-in-script | Ensures cleanup; GPU resources persist across mounts |
   | Job array (100 jobs) | Mount-in-script | Must be automated |

   Bottom line: "Start with pre-mount (simpler). As jobs get more complex, move to mount-in-script."

3. **Password handling in scripts (3 min):**
   - Never hard-code password in script (version control, logs expose it)
   - Store password in file: `/rhome/alice/.vault_pw` (400 permissions, home directory only)
   - Script: `gocryptfs -extpass "cat /rhome/alice/.vault_pw" /bigdata/vault /rhome/alice/vault`
   - Cleanup: Script deletes password file after unmount (if using temporary file), OR password file persists (acceptable if 400 permissions)
   - Never commit password file to Git (add to `.gitignore`)

4. **Cleanup strategy (2 min):** Ensure vault unmounts even if job fails:
   - Use bash trap: `trap "fusermount -u /path" EXIT` at start of script
   - Trap ensures unmount runs even if job crashes or exits early
   - Prevents data leakage or resource exhaustion (mounted filesystem left dangling)

**Hands-On Exercise (5 min):** Participants fill out a worksheet:
- Scenario 1: "I have 20 patient CSV files to analyze. Each analysis takes 5 minutes. Should I pre-mount or mount-in-script?" (Answer: mount-in-script for job array)
- Scenario 2: "I'm debugging a script interactively on command line. Should I pre-mount or mount-in-script?" (Answer: pre-mount)
- Scenario 3: "I'm submitting 500 image processing jobs to Sagehen. Each job takes 30 minutes." (Answer: mount-in-script with job array)

**Common Pitfalls:**
- Hard-coding password in script → catch in review
- Using temporary password file without cleanup → explain trap pattern
- Uncertainty about which strategy to use → refer to decision table

**Teaching Tips:**
- Decision framework is the key takeaway; not every script needs both strategies
- Password handling is a security concern; reinforce 400 permissions and file location
- Trap pattern is worth emphasizing; it's a general bash pattern applicable beyond gocryptfs

**Timing Guidance:**
- Strategies overview: 5 min
- Decision framework: 5 min (participants need time to internalize the table)
- Password handling: 3 min
- Exercise: 5 min (quick worksheet application)

#### Episode 11: SLURM Script Templates and Patterns (15 min teaching + 10 min exercises)

**Learning Outcome:** Participants have working templates for common SLURM scenarios and understand the trap pattern for error handling.

**Key Concepts:**
- Full template: Initialize, mount, run job, unmount, cleanup
- Trap pattern: Ensures cleanup runs even on error
- Storage location for password file: home directory, restricted permissions
- Logging and error handling within script
- Portability: Script works for different researchers with different vault paths

**Teaching Approach:**

1. **Template structure (7 min):** Walk through complete template:

   ```bash
   #!/bin/bash
   #SBATCH --job-name=cardio-analysis
   #SBATCH --time=00:30:00
   #SBATCH --ntasks=1
   #SBATCH --cpus-per-task=4
   #SBATCH --mem=8G

   # Configuration
   VAULT_CIPHERTEXT="/bigdata/lab/vault_cardio_2026"
   VAULT_PLAINTEXT="/rhome/alice/cardio_data"
   PASSWORD_FILE="/rhome/alice/.vault_pw"

   # Cleanup function
   cleanup() {
       echo "[$(date)] Unmounting vault..."
       fusermount -u "$VAULT_PLAINTEXT" 2>/dev/null || echo "[$(date)] Warning: unmount failed"
   }
   trap cleanup EXIT

   # Mount vault
   echo "[$(date)] Mounting encrypted vault..."
   gocryptfs -extpass "cat $PASSWORD_FILE" "$VAULT_CIPHERTEXT" "$VAULT_PLAINTEXT"
   if [ $? -ne 0 ]; then
       echo "[$(date)] ERROR: Mount failed"
       exit 1
   fi
   echo "[$(date)] Vault mounted successfully"

   # Run analysis
   echo "[$(date)] Starting analysis..."
   python3 /home/alice/analysis.py "$VAULT_PLAINTEXT/data.csv"
   ANALYSIS_STATUS=$?
   echo "[$(date)] Analysis completed with status $ANALYSIS_STATUS"

   # Implicit unmount via trap
   exit $ANALYSIS_STATUS
   ```

   Key features:
   - SBATCH directives: job configuration (time, CPUs, memory)
   - Configuration variables: easy to adapt for different researchers/vaults
   - Cleanup function + trap: ensures unmount on success, error, timeout
   - Error checking: `if [ $? -ne 0 ]` validates mount succeeded
   - Logging: timestamps and status messages help debugging
   - Exit status: script returns analysis exit code (0 = success)

2. **Trap pattern explained (4 min):**
   ```bash
   cleanup() {
       fusermount -u "$VAULT_PLAINTEXT" 2>/dev/null
   }
   trap cleanup EXIT
   ```
   - `trap cleanup EXIT`: Bash executes cleanup function before script exits (success, error, timeout)
   - Ensures vault unmounts even if:
     - Script crashes (e.g., Python error)
     - Job times out (SLURM sends SIGTERM)
     - Researcher cancels job (Ctrl+C, `scancel`)
   - `2>/dev/null`: Suppress "already unmounted" error if cleanup runs twice

3. **Customization for different scenarios (4 min):**
   - Single job: change job-name, vault paths, analysis command
   - Job array: add `#SBATCH --array=1-100` (covered in Episode 12)
   - GPU job: add `#SBATCH --gres=gpu:1`
   - Multi-node job: change `--ntasks` and `--cpus-per-task`
   - Long-running job: increase `--time`
   - Password file location: `/rhome/alice/.vault_pw` (home dir, secure)

4. **Common modifications (2 min):**
   - **Logging to file:** Redirect output: `#SBATCH --output=job_%j.log`
   - **Email on completion:** `#SBATCH --mail-type=END --mail-user=alice@pomona.edu`
   - **Multiple vaults:** Add second `gocryptfs` mount command and cleanup for second vault
   - **Data staging:** Copy data from vault to `/scratch` for speed (if performance-critical)

**Hands-On Exercise (10 min):** Participants create a SLURM script using template:
1. Copy template from provided file (or write from scratch)
2. Customize for their vault path (VAULT_CIPHERTEXT, VAULT_PLAINTEXT)
3. Customize for their password file location
4. Replace analysis command with a simple example (e.g., `wc -l "$VAULT_PLAINTEXT/data.txt"`)
5. Save script as `my_job.sh`
6. Syntax check: `bash -n my_job.sh`
7. Submit to SLURM: `sbatch my_job.sh`
8. Monitor: `squeue -u $(whoami)`
9. Wait for completion; check output log
10. Verify vault unmounted: `ls "$VAULT_PLAINTEXT/"` (should be empty)

**Common Pitfalls:**
- Forgetting trap pattern → emphasize it's essential
- Password file with wrong permissions → verify 400 before running
- Incorrect vault path → validate paths with `ls` before submitting
- Script doesn't unmount on error → explain trap catches all exits

**Teaching Tips:**
- Template is immediately useful; participants can copy and adapt
- Walk through template line-by-line first; then let participants customize
- Syntax check (`bash -n`) is quick validation; use it
- Actual job submission and monitoring is valuable (shows real SLURM integration)

**Timing Guidance:**
- Template walkthrough: 7 min (detailed; don't rush)
- Trap pattern: 4 min (core concept)
- Customization: 4 min (context and examples)
- Exercise: 10 min (hands-on, real job submission)

#### Episode 12: Advanced SLURM Patterns (15 min teaching + 15 min exercises)

**Learning Outcome:** Participants adapt gocryptfs to complex HPC scenarios: job arrays, pipelines, GPU jobs, error recovery, logging.

**Key Concepts:**
- Job arrays: mount once, run many jobs with same vault access
- Pipelines: multi-stage jobs sharing encrypted data
- GPU jobs: vault persists across GPU memory allocations
- Error recovery: Handling mount failures, quota errors, password issues
- Logging and monitoring: Debug slow jobs, track vault usage

**Teaching Approach:**

1. **Job arrays (4 min):**
   Problem: 100 analysis jobs, all need same encrypted vault. Mounting/unmounting 100 times is inefficient.
   
   Solution: Single job array script
   ```bash
   #SBATCH --array=1-100  # Run 100 jobs in parallel (subject to queue limits)
   
   # Mount once (each job in array mounts independently; SLURM manages concurrency)
   gocryptfs -extpass "cat $PASSWORD_FILE" "$VAULT_CIPHERTEXT" "$VAULT_PLAINTEXT"
   
   # Job-specific processing
   INPUT_FILE=$(printf "$VAULT_PLAINTEXT/input_%03d.csv" $SLURM_ARRAY_TASK_ID)
   python3 analyze.py "$INPUT_FILE"
   ```
   - `$SLURM_ARRAY_TASK_ID`: identifies which job in array (1–100)
   - Each job mounts independently (filesystem handles concurrency)
   - Each job unmounts via trap (cleanup function runs for each)
   - Efficient: SLURM schedules 100 jobs; each mounts/unmounts once

2. **Pipelines (3 min):**
   Problem: Multi-stage pipeline (extract → process → visualize) sharing encrypted data. Want to run as single job with multiple stages.
   
   Solution: Single job, multiple commands, single mount
   ```bash
   # Mount once
   gocryptfs -extpass "cat $PASSWORD_FILE" "$VAULT_CIPHERTEXT" "$VAULT_PLAINTEXT"
   trap cleanup EXIT
   
   # Stage 1: Extract
   python3 extract.py "$VAULT_PLAINTEXT/raw_data.csv" "$VAULT_PLAINTEXT/extracted.csv"
   [ $? -eq 0 ] || { echo "Extract failed"; exit 1; }
   
   # Stage 2: Process
   python3 process.py "$VAULT_PLAINTEXT/extracted.csv" "$VAULT_PLAINTEXT/processed.csv"
   [ $? -eq 0 ] || { echo "Process failed"; exit 1; }
   
   # Stage 3: Visualize
   python3 visualize.py "$VAULT_PLAINTEXT/processed.csv" "$VAULT_PLAINTEXT/report.pdf"
   [ $? -eq 0 ] || { echo "Visualize failed"; exit 1; }
   ```
   - Vault mounted once; all stages use same plaintext access
   - Error checking after each stage; fails fast if intermediate stage fails
   - Cleaner than separate jobs (shared intermediate data encrypted, not copied to `/scratch`)

3. **GPU jobs (2 min):**
   Problem: GPU job needs encrypted data; GPU occupies limited resources.
   
   Solution: Same script as basic template; gocryptfs works fine with GPU allocation
   ```bash
   #SBATCH --gres=gpu:1  # Request 1 GPU
   
   # Mount before GPU work
   gocryptfs -extpass "cat $PASSWORD_FILE" "$VAULT_CIPHERTEXT" "$VAULT_PLAINTEXT"
   trap cleanup EXIT
   
   # GPU work
   python3 ml_analysis.py --gpu "$VAULT_PLAINTEXT/training_data.csv"
   ```
   - No special handling needed; encryption overhead is small
   - Typical GPU job (hours) justifies automated mounting

4. **Error recovery and debugging (4 min):**
   
   - **Mount failure (bad password):**
     ```bash
     if ! gocryptfs -extpass "cat $PASSWORD_FILE" "$VAULT_CIPHERTEXT" "$VAULT_PLAINTEXT"; then
         echo "[$(date)] ERROR: Mount failed. Check password file."
         echo "[$(date)] Disk usage: $(df -h $VAULT_CIPHERTEXT | tail -1)"
         exit 1
     fi
     ```
   
   - **Quota exceeded during job:**
     ```bash
     python3 analyze.py "$VAULT_PLAINTEXT/data.csv" || {
         echo "[$(date)] Job failed. Check quota:"
         quota
         exit 1
     }
     ```
   
   - **Slow vault performance:**
     Add monitoring to detect I/O bottlenecks:
     ```bash
     echo "[$(date)] Vault mount performance: $(df "$VAULT_PLAINTEXT")"
     du -sh "$VAULT_PLAINTEXT"  # Size of vault contents
     ```

5. **Logging strategy (2 min):**
   Capture script output for debugging:
   ```bash
   #SBATCH --output=/rhome/alice/logs/job_%j.log
   #SBATCH --error=/rhome/alice/logs/job_%j.err
   exec > >(tee -a "/rhome/alice/logs/job_%j.log")
   exec 2>&1
   ```
   - Logs go to file and stdout
   - `job_%j` includes SLURM job ID (e.g., `job_12345.log`)
   - Later review: inspect logs to debug vault mount issues

**Hands-On Exercise (15 min):** Participants implement an advanced scenario:

**Option A (Job Array):**
1. Create 5 small input files in vault: `input_001.csv`, `input_002.csv`, etc.
2. Write simple Python script that reads input file, does minimal processing, writes output
3. Adapt job array template to process all 5 inputs
4. Submit: `sbatch --array=1-5 my_array_job.sh`
5. Monitor: `squeue -u $(whoami)`
6. Verify: all 5 jobs completed successfully, outputs in vault

**Option B (Pipeline):**
1. Write 3 simple Python scripts: stage1.py (create sample data), stage2.py (transform data), stage3.py (summarize results)
2. Create SLURM script with all 3 stages
3. Submit: `sbatch my_pipeline.sh`
4. Monitor and verify all stages completed

**Option C (Error Recovery):**
1. Create SLURM script with intentional error (e.g., access non-existent file)
2. Submit: `sbatch my_error_job.sh`
3. Monitor: observe job fails
4. Inspect output log: `cat /rhome/alice/logs/job_*.log`
5. Modify script to catch error, add recovery logic
6. Resubmit: `sbatch my_error_job.sh`
7. Verify: job handles error gracefully

**Common Pitfalls:**
- Job array with 100 jobs all mounting vault simultaneously (filesystem thrashing) → clarify SLURM serializes with respect to mount point
- Forgetting trap in complex scripts → emphasize it's always needed
- Logging output lost → use `#SBATCH --output` directive
- GPU resource starvation (vault on compute-bound job taking GPU time) → performance overhead is negligible; don't over-optimize prematurely

**Teaching Tips:**
- Job arrays and pipelines are real-world patterns; participants will recognize use cases
- Error recovery is practical debugging; valuable skill
- Logging strategy is underrated; helps troubleshooting significantly

**Timing Guidance:**
- Job arrays: 4 min (common pattern)
- Pipelines: 3 min (advanced but elegant)
- GPU jobs: 2 min (brief; mostly reassurance that it works)
- Error recovery: 4 min (important but not all scenarios needed)
- Logging: 2 min (quick but high-value)
- Exercise: 15 min (hands-on; expect some debugging)

---

### SECTION 6: Key Management (35 min teaching + 25 min exercises)

#### Episode 13: Key Backup Strategy (10 min teaching + 5 min exercises)

**Learning Outcome:** Participants implement 3-2-1 backup rule for encrypted vaults and understand why password recovery is impossible.

**Key Concepts:**
- Three critical components: gocryptfs.conf (encrypted master key), plaintext data, password (single copy)
- 3-2-1 backup rule: 3 copies, 2 different media, 1 offsite
- Backup of gocryptfs.conf enables vault recovery if ciphertext is lost
- Password cannot be recovered; must be stored safely (password manager)
- Disaster scenarios: drive failure, accidental deletion, hardware theft

**Teaching Approach:**

1. **Three critical components (3 min):**
   - **gocryptfs.conf:** Encrypted config file in vault directory; contains encrypted master key; lost if vault ciphertext deleted
   - **Plaintext data:** Files stored in encrypted form on disk; encrypted by AES-256-GCM; unreadable without password
   - **Password:** Human-memorable secret; only thing not backed up; if lost, data is inaccessible even to experts

   "You have two things to back up (gocryptfs.conf and plaintext data if valuable). You have one thing you can't lose (password)."

2. **3-2-1 backup rule (4 min):**
   Apply to encrypted vault:
   - **3 copies:** Original vault on `/bigdata/lab/`, backup copy in cloud (institutional Google Drive, Box), backup copy on external drive
   - **2 different media:** Shared storage (NFS `/bigdata`), cloud (Google Drive), local drive (external USB)
   - **1 offsite:** Cloud or external drive stored away from office

   Example backup strategy:
   ```
   Original: /bigdata/lab/vault_cardio_2026/
   Backup 1: Google Drive folder (institution-managed cloud)
   Backup 2: External USB drive stored in safe (not at desk)
   ```

3. **Backing up gocryptfs.conf specifically (2 min):**
   Most important single file:
   ```bash
   cp /bigdata/lab/vault_cardio_2026/gocryptfs.conf /rhome/alice/backup_gocryptfs.conf
   ```
   Store backup in:
   - Password manager (as attachment or plaintext)
   - Email (to self, with subject "VAULT BACKUP - DO NOT SHARE")
   - Encrypted USB drive
   - Cloud (institutional storage; encrypted by provider)

   Key point: "If vault ciphertext directory is deleted, you can restore using backed-up gocryptfs.conf + password. But the password itself can't be recovered."

4. **Password recovery is impossible (1 min):** Emphasize:
   - gocryptfs.conf is encrypted with password
   - If password is lost AND gocryptfs.conf is lost, vault is permanently inaccessible
   - No backdoor, no recovery key
   - "This is by design: only you control access."

**Hands-On Exercise (5 min):** Participants implement backup for their vault:
1. Copy gocryptfs.conf to backup location: `cp /bigdata/lab/vault_PROJECTNAME_2026/gocryptfs.conf /rhome/alice/backup_gocryptfs.conf`
2. Upload backup to Google Drive (or institutional cloud)
3. Copy vault ciphertext directory to external drive (if available): `rsync -av /bigdata/lab/vault_PROJECTNAME_2026 /mnt/external_drive/backup/`
4. Verify backup integrity: `ls -l /mnt/external_drive/backup/vault_PROJECTNAME_2026/gocryptfs.conf` (should exist)
5. Document backup location in password manager (note: "Backup of vault_cardio_2026 stored on external drive and Google Drive")

**Common Pitfalls:**
- Relying on single backup (violates 3-2-1 rule) → enforce 3 copies minimum
- Storing password in plaintext file on computer (if computer stolen, backup is lost) → emphasize password manager
- Backup stored in same location as vault (e.g., both on `/bigdata`) → clarify "offsite" means different physical location

**Teaching Tips:**
- Backup is boring but essential; make it concrete with examples
- Password recovery impossibility is scary but reassuring (no backdoor = more secure)
- 3-2-1 rule is industry standard; applicable beyond gocryptfs

**Timing Guidance:**
- Components overview: 3 min
- 3-2-1 rule: 4 min (detailed; worth the time)
- gocryptfs.conf backup: 2 min
- Password recovery impossibility: 1 min
- Exercise: 5 min (hands-on backup implementation)

#### Episode 14: Password Management (10 min teaching + 5 min exercises)

**Learning Outcome:** Participants use password manager to generate, store, and securely share passwords.

**Key Concepts:**
- NIST SP 800-63B: 14+ character passwords, no dictionary words
- Password managers: Bitwarden (free), 1Password, LastPass (institution may provide)
- Secure password generation: random, not memorable
- Secure sharing: never email plaintext; use password manager sharing or secure vault
- Password change: Argon2 stretching makes password changes slow but secure
- Emergency access: PI should have recovery key option

**Teaching Approach:**

1. **Password requirements (2 min):**
   - Length: 14+ characters (NIST recommends 15+ for sensitive data)
   - Composition: uppercase, lowercase, numbers, symbols
   - Randomness: avoid dictionary words, names, birthdays, keyboard patterns (qwerty)
   - Example: `Bx7#mQ2$pK9@wRt` (14 chars, mixed case, symbols, random)

2. **Password manager setup (4 min):**
   - Recommendation: Bitwarden (free, open-source) or institution-provided (1Password, LastPass)
   - Setup: Create Bitwarden account with strong master password
   - Generation: In Bitwarden, "Generate Password" → 14+ chars, include symbols → copy to clipboard
   - Storage: Save in Bitwarden vault with vault path and username info
   - Access: Search Bitwarden for "cardio_vault" → retrieve password when mounting
   - Portability: Bitwarden accessible from any device (phone, laptop, tablet); encrypted with master password

3. **Secure password sharing (2 min):**
   Scenario: "PI needs emergency access to my encrypted vault. How do I share the password?"
   - **Wrong:** Email plaintext password (email is logged, insecure)
   - **Right:** Add PI to shared collection in Bitwarden (Bitwarden → Collections → Share → Add colleague)
   - Alternative: Use password manager's emergency access feature (e.g., 1Password Emergency Contact) → PI gets access only if you don't reset the timer
   - Never: Post password in shared document, Slack, or lab notebook

4. **Password change (1 min):**
   Scenario: "I want to change my vault password (or suspect it's compromised)."
   - Old password: required to decrypt vault
   - New password: required to encrypt new master key
   - Process: `gocryptfs -passwd /bigdata/lab/vault_cardio_2026` → prompts for old password → prompts for new password
   - Time: Argon2 stretching takes ~3 seconds (by design; slows brute-force attacks)
   - Update password manager: Change stored password in Bitwarden and inform PI if shared

5. **Emergency/succession planning (1 min):**
   - PI should know vaults exist (inventory)
   - At least PI + postdoc should have access (shared in password manager, or PI has emergency recovery key)
   - Document: "Lab has encrypted vaults with following purposes; PI has emergency access"
   - Annual review: Update access list if lab members change

**Hands-On Exercise (5 min):** Participants practice password management:
1. Open password manager (Bitwarden or institution-provided)
2. Generate new password (14+ chars, symbols)
3. Store in vault with entry "gocryptfs_vault_cardio_2026"
4. Note related info: vault path, mount point, purpose
5. If sharing with PI: Create shared collection, add PI's email
6. Change vault password: `gocryptfs -passwd /bigdata/lab/vault_cardio_2026`
   - Old password: current password (from password manager)
   - New password: use password manager to generate new random password
   - Confirm change: update password manager with new password
7. Test: Unmount vault, remount with new password (should succeed)

**Common Pitfalls:**
- Weak password (dictionary word) → use password manager to generate
- Password stored in plaintext file on computer → password manager only
- PI doesn't have emergency access → emphasize succession planning
- Password change forgotten in password manager → update both simultaneously

**Teaching Tips:**
- Password manager demo is worth 5 minutes; makes password management concrete
- Shared collections in Bitwarden are elegant; show the feature
- Password change is slow (Argon2) but safe; users should understand the trade-off
- Succession planning is often overlooked; emphasize it during this episode

**Timing Guidance:**
- Requirements: 2 min
- Password manager setup: 4 min (demo valuable)
- Secure sharing: 2 min
- Password change: 1 min
- Emergency access: 1 min
- Exercise: 5 min (hands-on; password change is realistic)

#### Episode 15: Lab Management and Disaster Recovery (15 min teaching + 15 min exercises)

**Learning Outcome:** Participants develop lab-level practices: vault inventory, succession planning, disaster recovery, compliance documentation.

**Key Concepts:**
- Vault inventory: Lab maintains list of all encrypted vaults, purposes, owners
- Succession planning: What happens when postdoc leaves or PI retires
- Disaster recovery: Recover vault from backup after data loss
- Compliance documentation: Records for audit (HIPAA, FERPA, IRB)
- Archival: Vaults must be retained per legal/regulatory hold periods

**Teaching Approach:**

1. **Lab-level vault inventory (3 min):**
   Lab manager maintains spreadsheet:
   ```
   | Vault Name | Purpose | Owner | PI | Encryption Status | Backup Location | Retention Until |
   |------------|---------|-------|----|--------------------|-----------------|-----------------|
   | vault_cardio_2026 | HIPAA patient data | Alice | Bob | Encrypted | Google Drive + USB | 2031-06-30 |
   | vault_genetics_2024 | DNA samples with IDs | Charlie | Bob | Encrypted | Google Drive | 2032-01-15 |
   ```
   - Purpose: audit trail; enables handoff
   - Owner: who manages vault day-to-day
   - PI: who approves usage and retention
   - Encryption status: confirms data is protected
   - Backup location: enables recovery
   - Retention until: legal/regulatory hold

2. **Succession planning (4 min):**
   Scenario: Postdoc Alice (vault owner) is leaving. What happens to cardio_2026 vault?
   
   - **Before departure:**
     1. Alice provides vault password to PI (or shared collection in Bitwarden)
     2. PI confirms access (test mount with shared password)
     3. Alice documents vault contents, structure, any quirks (e.g., symbolic links)
     4. Verify backup is up-to-date and accessible
   
   - **Transition plan:**
     1. New postdoc Charlie takes over vault
     2. Alice changes password (if needed for security)
     3. Charlie's credentials added to shared access
     4. Lab inventory updated: Alice → Charlie
   
   - **Archive phase (if project ended):**
     1. Alice stops active use
     2. Vault remains encrypted on `/bigdata/lab/` (not deleted)
     3. Backup verified and stored offsite
     4. Retention period enforced (e.g., 7 years per NIH record retention)

3. **Disaster recovery scenario (4 min):**
   Scenario: `/bigdata/lab/vault_cardio_2026` directory is accidentally deleted.
   
   - **Discovery:** Charlie tries to mount vault: "directory doesn't exist"
   - **Recovery steps:**
     1. Restore from backup: `cp -r /mnt/external_drive/backup/vault_cardio_2026 /bigdata/lab/vault_cardio_2026`
     2. Verify restore: `ls /bigdata/lab/vault_cardio_2026/gocryptfs.conf` (should exist)
     3. Mount: `gocryptfs /bigdata/lab/vault_cardio_2026 /rhome/charlie/cardio_data` (with password)
     4. Verify contents: `ls /rhome/charlie/cardio_data/` (should see files)
     5. Time to recovery: minutes (not hours)
   
   - **Lesson:** Backup was critical; without it, data would be permanently lost.

4. **Compliance documentation (2 min):**
   For regulated data (HIPAA, FERPA):
   - Document: Vault exists, encryption is AES-256-GCM, access is logged (via Sagehen account access)
   - Annual audit: Confirm vaults are encrypted, backups are tested
   - Incident response: If vault is breached, document in compliance system
   - Retention: Comply with data retention laws (HIPAA 6 years, FERPA 5+ years, IRB per protocol)

5. **Archival and retention (2 min):**
   - Active project: Vault stored on `/bigdata`, regularly backed up, actively used
   - Completed project (data retention required): Vault remains encrypted, moved to archive storage, backup maintained
   - Example: Cardio study completed 2023; data must be kept until 2031 (HIPAA 6-year rule + 1 year for archive). Vault remains encrypted the entire time.
   - Never delete encrypted vault until retention period expires.

**Hands-On Exercise (15 min):** Participants design lab-level management plan for their vault:

**Part 1: Create Vault Inventory (5 min)**
1. Create spreadsheet (or use provided template) with columns: Vault Name, Purpose, Owner, PI, Status, Backup Location, Retention
2. Fill in details for their vault
3. Estimate retention period (if HIPAA data: 2031 minimum; if FERPA: 2030+; if NIH-funded: 7 years)

**Part 2: Succession Plan (5 min)**
1. Write brief document: "What happens if current owner leaves?"
2. Identify who would take over (if any)
3. Document password access strategy (shared in Bitwarden? PI has recovery key?)
4. Document onboarding steps for new owner (mount, verify access, update inventory)

**Part 3: Disaster Recovery Test (5 min)**
1. Simulate disaster: Rename vault directory to test backup
   ```bash
   mv /bigdata/lab/vault_cardio_2026 /bigdata/lab/vault_cardio_2026.bak
   ```
2. Restore from backup (if available) or use the .bak as mock backup
   ```bash
   cp -r /bigdata/lab/vault_cardio_2026.bak /bigdata/lab/vault_cardio_2026
   ```
3. Test mount: `gocryptfs /bigdata/lab/vault_cardio_2026 /rhome/alice/cardio_data`
4. Verify contents: `ls /rhome/alice/cardio_data/`
5. Document time to recovery

**Common Pitfalls:**
- No lab inventory (vault knowledge lost when person leaves) → emphasize spreadsheet
- No succession plan (vault becomes orphaned) → make it explicit
- Backup tested only once initially (real disaster reveals backup is broken) → emphasize regular testing
- Retention period misunderstood (data deleted too early) → provide clear retention rules

**Teaching Tips:**
- Succession planning is often overlooked but critical; emphasize it
- Disaster recovery test should be low-stakes (just renaming directory) but educational
- Lab-level management is different from individual skill; frame as "protect your lab"
- Compliance documentation is often required by auditors; make it concrete

**Timing Guidance:**
- Inventory: 3 min
- Succession: 4 min
- Disaster recovery: 4 min
- Compliance: 2 min
- Archival: 2 min
- Exercise: 15 min (hands-on; worthwhile)

---

### SECTION 7: Best Practices (30 min teaching + 20 min exercises)

#### Episode 16: Security Best Practices and Troubleshooting (15 min teaching + 5 min exercises)

**Learning Outcome:** Participants apply 10 security habits, avoid 6 common mistakes, and troubleshoot performance/access issues.

**Key Concepts:**
- 10 habits: unmount when away, strong passwords, backup, no password sharing, monitor access, regular testing
- 6 mistakes: mounting on shared paths, weak passwords, no backup, password in version control, leaving mounted overnight, ignoring errors
- Troubleshooting: slow performance, permission errors, mount failures
- Performance tuning: when to optimize, what not to optimize

**Teaching Approach:**

1. **10 security habits (6 min):**
   1. **Unmount when leaving workstation:** Don't leave vault mounted while away. Reduces window of exposure.
   2. **Use password manager:** Generate strong passwords, store securely, no plaintext files.
   3. **Implement 3-2-1 backup:** Three copies, two media, one offsite. Test backup annually.
   4. **Share passwords securely:** Password manager sharing or emergency access feature. Never email plaintext.
   5. **Monitor access logs:** Periodically check who accessed vault (SLURM job logs, file timestamps).
   6. **Test backup recovery:** Annual practice restore from backup ensures backup is valid.
   7. **Document vault purpose:** Inventory spreadsheet helps succession planning.
   8. **Update passwords annually:** Especially for long-term sensitive projects.
   9. **Verify unmount:** After working with encrypted data, confirm vault is unmounted (`ls /mountpoint` should be empty).
   10. **Report security concerns:** If vault access seems wrong or suspicious, notify ITS immediately.

2. **6 common mistakes (4 min):**
   1. **Mounting on shared paths (e.g., `/bigdata/lab/myvault_mount`):** Decrypted data visible to all lab members. Fix: Always mount in personal home directory.
   2. **Using weak passwords:** "12345678", "password123". Vulnerable to brute-force. Fix: Use password manager; minimum 14 characters, random.
   3. **No backup strategy:** Relying only on original vault. If hard drive fails, data is lost. Fix: Implement 3-2-1 rule; test annually.
   4. **Hardcoding password in scripts:** Password in version control, logs, email. Fix: Use password file with 400 permissions or password manager.
   5. **Leaving vault mounted overnight:** Reduces security if office is shared. Fix: Unmount before leaving; mount only when working.
   6. **Ignoring mount errors:** "Bad password" error; researcher tries again without checking. Fix: Pause, verify password in password manager, then retry.

3. **Troubleshooting workflow (3 min):**
   - **Slow vault performance (< 50% expected throughput):**
     1. Check disk space: `df -h /bigdata` (quota exceeded?)
     2. Check load: `top` (CPU bottleneck?)
     3. Check I/O: `iostat` (disk bottleneck?)
     4. Check encryption overhead: Run `dd` test to benchmark (`dd if=/dev/zero of=/path/test bs=1M count=100`)
     5. If overhead > 50%, contact ITS for guidance
   
   - **Permission denied when accessing mounted files:**
     1. Verify mount point owner: `ls -ld /rhome/alice/cardio_data/` (should show alice as owner)
     2. Verify file permissions in vault: `ls -l /rhome/alice/cardio_data/` (should be readable)
     3. Verify gocryptfs.conf permissions: `ls -l /bigdata/lab/vault_cardio_2026/gocryptfs.conf` (should be 600)
     4. If still denied: unmount and remount (`fusermount -u /path && gocryptfs ...`)
   
   - **Mount fails with "Already mounted":**
     1. Check current mounts: `df | grep cardio_data`
     2. If mounted: unmount with `fusermount -u /path`
     3. If not listed but error persists: remove stale mount point with `rm -rf /rhome/alice/cardio_data` and recreate directory
   
   - **Password prompt not responding (SSH session):**
     1. Check if terminal is in raw mode: `stty sane`
     2. Try Ctrl+C to cancel, then retry
     3. If stuck: close SSH session and reconnect

4. **Performance tuning (2 min):**
   - gocryptfs performance is adequate for most workflows (overhead 5-15%)
   - Don't over-optimize prematurely
   - Acceptable slowdown: typical research workflows don't require real-time performance
   - If optimization needed:
     - Move frequently-accessed files to `/scratch` (unencrypted, fast) — copy at end of job back to vault
     - Use compression on vault if available (`-nonempty` flag; see manual)
     - Avoid small files (prefer larger consolidated files); encryption has per-file overhead

**Hands-On Exercise (5 min):** Participants practice troubleshooting:
1. Deliberately cause an error (e.g., try to mount with wrong password): observe error message
2. Interpret error: "Bad password" → password is wrong, not account locked
3. Verify password in password manager
4. Retry mount: should succeed
5. Simulate "already mounted": attempt second mount to same point
6. Use troubleshooting workflow: check mount status with `df`
7. Fix: unmount first, then retry

**Common Pitfalls:**
- Researchers assume some security habits are "optional" → emphasize they're all best practice
- Confusing "slow performance" with "vault is broken" → teach diagnostic steps
- Forgetting to unmount before restart → mention automatic unmount on reboot (SLURM cleans up mounts)

**Teaching Tips:**
- 10 habits are concrete and actionable; reference card helps
- 6 mistakes framework (what not to do) is memorable
- Troubleshooting workflow is practical; use real examples from your institution
- Performance conversation is about expectations; set realistic overhead

**Timing Guidance:**
- 10 habits: 6 min (list them; spend 30 sec on each)
- 6 mistakes: 4 min (parallel structure to habits; easier to remember)
- Troubleshooting: 3 min (overview; details in exercise)
- Performance tuning: 2 min (brief; don't over-optimize)
- Exercise: 5 min (low-stakes error scenarios)

#### Episode 17: Verification, Sharing, and Maintenance (15 min teaching + 15 min exercises)

**Learning Outcome:** Participants verify vault integrity, share encrypted data with collaborators, and maintain vaults over time.

**Key Concepts:**
- Verification: Testing vault and password, checking encryption integrity
- Sharing encrypted data: Options for collaborators (shared vault, encrypted zip, cloud sync)
- Lifecycle maintenance: Cleanup, optimization, migration to new storage
- Long-term use: Annual password testing, backup refresh, compliance checks

**Teaching Approach:**

1. **Verification protocol (4 min):**
   After creation or major change, verify vault is working:
   
   **Basic verification (5 minutes):**
   ```bash
   # 1. Mount vault
   gocryptfs /bigdata/lab/vault_cardio_2026 /rhome/alice/cardio_data
   
   # 2. Create test file
   echo "test content $(date)" > /rhome/alice/cardio_data/verification_test.txt
   
   # 3. Verify file readable
   cat /rhome/alice/cardio_data/verification_test.txt
   
   # 4. Unmount and verify can't access plaintext
   fusermount -u /rhome/alice/cardio_data
   ls /rhome/alice/cardio_data/ # Should be empty
   
   # 5. Remount and verify test file still there
   gocryptfs /bigdata/lab/vault_cardio_2026 /rhome/alice/cardio_data
   cat /rhome/alice/cardio_data/verification_test.txt
   
   # 6. Cleanup
   rm /rhome/alice/cardio_data/verification_test.txt
   fusermount -u /rhome/alice/cardio_data
   ```
   
   **Annual verification (30 minutes):**
   - Restore vault from backup (test location)
   - Mount restored vault with stored password
   - Verify all critical files are present and readable
   - Calculate checksums: `find /rhome/alice/cardio_data -type f -exec md5sum {} \; > /tmp/checksums.txt`
   - Document: "Backup verified 2026-04-09; all files intact, 42 files, 2.1 GB"

2. **Sharing encrypted data (5 min):**
   Scenario: Collaborator at different institution needs access to encrypted cardiology data.
   
   **Option 1: Shared vault (recommended for team)**
   - Both researchers in lab with access to Sagehen
   - Share vault on `/bigdata/lab`; both have access to password (via Bitwarden shared collection)
   - Both mount same vault to their own directories
   - Efficient: single copy, both access directly
   
   **Option 2: Encrypted zip file**
   - Researcher exports vault to encrypted zip: `gocryptfs /vault_source /tmp/vault_mount && zip -r vault_backup.zip /tmp/vault_mount && fusermount -u /tmp/vault_mount`
   - Send zip to collaborator (unencrypted transmission is now safe; zip is encrypted)
   - Collaborator extracts and mounts (needs vault password)
   - Less efficient: duplicate copy, slower for large data
   
   **Option 3: Cloud sync (Google Drive, Box)**
   - Archive vault as tar: `tar czf cardio_vault_backup.tar.gz /bigdata/lab/vault_cardio_2026`
   - Upload to Google Drive or institutional cloud
   - Collaborator downloads and restores: `tar xzf cardio_vault_backup.tar.gz`
   - Both encrypt and decrypt independently; good for offsite backup
   - Caveat: Cloud provider has encrypted data; password remains secret

   Bottom line: "If sharing real-time encrypted data with team, shared vault is best. If sending data to external collaborator, encrypted zip or archive."

3. **Lifecycle maintenance (3 min):**
   
   **Quarterly:**
   - Check disk usage: `du -sh /bigdata/lab/vault_cardio_2026`
   - Monitor for unexpected growth
   - Clean up old analysis outputs if needed
   
   **Annually:**
   - Test password (verify can still mount)
   - Test backup recovery (restore from backup, mount, verify)
   - Update lab inventory (confirm vault is still in use)
   - Review access logs if sensitive data
   
   **End of project:**
   - Archive vault (move to archive storage if available)
   - Update inventory: project complete, retention until DATE
   - Confirm backup is offsite
   - NO deletion unless retention period has expired and compliance approves

4. **Migration to new storage (2 min):**
   Scenario: Lab is moving to different HPC cluster with better storage.
   
   - Create vault on new cluster (same name, new storage)
   - Copy ciphertext directory: `rsync -av /bigdata/lab/vault_cardio_2026 /new_cluster/storage/vault_cardio_2026`
   - Test mount on new cluster with password
   - Delete from old cluster only after verifying success on new cluster
   - Update scripts, paths, documentation

5. **Password testing and lifecycle (3 min):**
   - Annual password test: `gocryptfs /bigdata/lab/vault_cardio_2026 /tmp/test_mount` (from password manager password)
   - Confirms password is still accessible and correct
   - If password has been changed (emergency), update password manager immediately
   - Inform PI if password was changed

**Hands-On Exercise (15 min):** Participants complete verification, sharing simulation, and maintenance planning:

**Part 1: Verification Protocol (5 min)**
1. Mount vault
2. Create test file with timestamp
3. Unmount and verify inaccessible
4. Remount and verify test file present
5. Document verification result: "Vault cardio_2026 verified 2026-04-09; test file confirmed readable after unmount/remount cycle"

**Part 2: Sharing Scenario (5 min)**
1. Create encrypted zip of vault contents:
   ```bash
   gocryptfs /bigdata/lab/vault_cardio_2026 /tmp/mount_temp
   zip -r /rhome/alice/vault_backup.zip /tmp/mount_temp
   fusermount -u /tmp/mount_temp
   ```
2. Note: encrypted zip is secure for transmission (no password needed, recipient uses vault password)
3. Imagine sending to collaborator: how would they restore it?

**Part 3: Maintenance Planning (5 min)**
1. Create maintenance calendar:
   - Quarterly: Check disk usage
   - Annually: Test password, verify backup, update inventory
   - End of project: Archive vault, confirm retention period
2. Document: "Next verification date: April 2027"
3. Add reminder to calendar (or use scheduled-tasks feature)

**Common Pitfalls:**
- Verification skipped (vault seems fine but actual test may fail) → emphasize annual test
- Sharing decision poorly made (sent plaintext, or encrypted zip to external with no clear process) → teach options
- Maintenance forgotten (vault grows unbounded, backup gets stale) → calendar reminders help

**Teaching Tips:**
- Verification is concrete and reassuring; participants feel confident after successful test
- Sharing scenario is realistic (collaborations happen); multiple options provide flexibility
- Maintenance planning is prevention (avoids crises); calendar-based approach is practical
- End-of-life planning (archival, retention) is often overlooked; emphasize compliance

**Timing Guidance:**
- Verification protocol: 4 min (concrete steps)
- Sharing encrypted data: 5 min (multiple options; tradeoffs important)
- Lifecycle maintenance: 3 min (calendar-based approach)
- Password testing: 3 min (simple but important)
- Exercise: 15 min (hands-on; three parts allow participants to choose focus)

---

## Delivery Recommendations

### Two Half-Day Format (Recommended for 6.5-hour workshop)

**Day 1: Foundations and Setup (3 hours)**
- Section 1: Why Encryption Matters (55 min)
- Section 2: How gocryptfs Works (45 min)
- Section 3: Hands-On Setup (55 min)
- Break: 15 min
- Q&A and troubleshooting: 15 min

**Day 2: Integration and Best Practices (3.5 hours)**
- Section 4: Daily Operations (45 min)
- Section 5: SLURM Integration (75 min)
- Break: 15 min
- Section 6: Key Management (60 min)
- Section 7: Best Practices (45 min)
- Final Q&A and wrap-up: 15 min

### Three-Session Format (Shorter sessions, distributed over weeks)

- **Session 1:** Sections 1–2 (1.5 hours) — Why and how
- **Session 2:** Sections 3–4 (1.5 hours) — Setup and daily use
- **Session 3:** Sections 5–7 (3.5 hours) — Integration and best practices

### Full-Day Intensive (6.5 hours with lunch)

All 17 episodes in sequence with lunch break at midpoint.

---

## Materials and Resources

- **Instructor slides:** Download from workshop portal
- **Reference card:** 1-page gocryptfs commands (provided in course materials)
- **Troubleshooting guide:** Common errors and fixes (provided)
- **SLURM templates:** Copy-paste ready scripts for participants
- **Vault inventory spreadsheet:** Template for lab management
- **Backup verification checklist:** Annual testing protocol
- **Contact:** its-hpc@pomona.edu for technical support

---

## Post-Workshop Follow-Up

- Email participants: reference links, contact info, announcement of future advanced workshops
- Schedule office hours: 2–3 weeks after workshop for questions
- Document common questions and update troubleshooting guide
- Measure success: participant vault creation rate, follow-up adoption (SLURM integration, backups)
