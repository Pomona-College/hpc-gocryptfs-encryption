---
title: "Encryption, Performance, and Comparison"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions
- What is AES-256-GCM and why is it secure?
- How does Argon2 prevent brute-force attacks on your password?
- How does gocryptfs.conf store the encryption key?
- What is the performance impact of encryption?
- Why is gocryptfs better than alternatives?
- What are the limitations and threat model?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand AES-256-GCM encryption with analogy and technical detail
- Learn Argon2 key derivation and why it matters for security
- Understand gocryptfs.conf and its critical importance
- Understand file-level vs. block-level encryption trade-offs
- Understand filename encryption and metadata protection
- Know gocryptfs performance on Sagehen HPC
- Compare gocryptfs to alternatives and understand why it's optimal
- Know gocryptfs limitations and threat model
::::::::::::::::::::::::::::::::::::::::::::::::

## Encryption Algorithm: AES-256-GCM

**AES-256-GCM** is the symmetric encryption cipher used by gocryptfs:

- **AES (Advanced Encryption Standard)**: NSA-approved, mathematically proven secure, hardware-accelerated by modern CPUs
- **256-bit key**: 2^256 possible keys (10^77)—brute-force would take longer than the age of the universe even with a trillion guesses/second
- **GCM (Galois/Counter Mode)**: Provides both secrecy and tamper-detection. Any modification of encrypted data causes decryption to fail.

**Bottom line**: AES-256-GCM is unbreakable with current technology and won't be broken in your lifetime. Modern CPUs have AES-NI hardware instructions, making encryption virtually free in terms of performance.

## Key Derivation: Argon2

Your password is NOT the encryption key. gocryptfs uses **Argon2**, a memory-hard key derivation function that converts your password into a cryptographic key.

**Why memory-hard?** Argon2 uses CPU, memory, and time resources—making it slow for attackers to guess passwords. GPU and ASIC attackers can't parallelize efficiently because they can't fit many copies in GPU memory. Result: deriving a key takes 1-2 seconds on your machine, but an attacker trying to crack a password waits 1-2 seconds per guess—reducing attack speed from billions of guesses/second to just one. A weak 8-character password with Argon2 is safer than the same password used directly as a key.

## gocryptfs.conf: Critical Configuration File

When you initialize gocryptfs, it creates **gocryptfs.conf**—a JSON file containing:
- **EncryptionKey**: Your encrypted master key (encrypted with your passphrase via Argon2)
- **ScryptObject**: Key derivation parameters (N, R, P, Salt)

**Why it's irreplaceable**: gocryptfs.conf holds the encrypted master key. Your passphrase + Argon2 derives a key to decrypt it. If you lose this file, data is permanently unrecoverable—even with the correct passphrase. The connection between your passphrase and the master key is gone.

**This is a feature, not a bug.** If your system is compromised, attackers cannot recover the master key without your passphrase. But YOU must back it up immediately.

**Backup procedure**:
```bash
# Back up gocryptfs.conf to secure location
cp /bigdata/group/mydata_cipher/gocryptfs.conf /rhome/myusername/backup/gocryptfs_mydata.backup
chmod 600 /rhome/myusername/backup/gocryptfs_mydata.backup
```

Losing gocryptfs.conf = losing all data. There is no recovery, no matter how strong your password is.

::::::::::::::::::::::::::::::::::::: callout
## Critical: Back Up gocryptfs.conf Immediately

Before accessing your encrypted directory for any real work, back up gocryptfs.conf to a separate, safe location:

```bash
# Backup to your home directory
cp /bigdata/yourgroup/cipher/gocryptfs.conf ~/backup_gocryptfs_cipher.conf

# Or to external drive
cp /bigdata/yourgroup/cipher/gocryptfs.conf /path/to/external/backup/
```

Losing this file means losing all encrypted data permanently, even if you remember your passphrase. There is no recovery. Back it up now, before you begin any actual work with encrypted data. Backup is not optional.
::::::::::::::::::::::::::::::::::::::::::::::::

## File-Level Encryption: Why gocryptfs Uses It

gocryptfs uses **file-level encryption** (not block-level):

**Advantages**:
- No administrator access required (users create their own encrypted directories)
- Selective encryption (only protect sensitive data)
- Compatible with HPC shared filesystems like BeeGFS
- Individual file corruption is isolated
- File structure preserved for backups and archival

**Trade-offs**:
- File sizes are visible (but content is encrypted)
- Directory structure is visible (folder names separate from file encryption)
- Timestamps not encrypted

For HPC clusters, file-level is the only practical option. You can't require all users to encrypt entire directories, and block-level encryption (LUKS) requires admin access.

## Filename Encryption

gocryptfs encrypts filenames in addition to file contents. Original files like `data.csv` become cryptic names like `gocryptfs.longfilename.4A3Lx7K9mQ2B_L5xP8uVw==` in the cipher directory. This prevents information leakage—an attacker cannot guess what files you have or correlate filenames with sizes. Filename encryption uses EME (Enciphering Mode for Encrypted Filenames), ensuring same filenames always encrypt identically while different filenames produce different ciphertexts.

## Performance: Encryption Overhead is Minimal

Modern CPUs have AES-NI hardware instructions, so encryption is fast. Typical overhead: ~5-10% for file processing. Example: on Sagehen, sequential reads show ~7% overhead (450 MB/sec unencrypted → 420 MB/sec encrypted). For most research workloads, encryption overhead is negligible compared to your algorithm's computational cost. If your job takes 24 hours with 1 hour of I/O, encryption adds ~6 minutes—acceptable.

## Why gocryptfs for HPC

gocryptfs is optimal because it's:
- **Secure**: AES-256-GCM encryption, memory-hard key derivation, no known vulnerabilities
- **Accessible**: No administrator access required
- **Practical**: Transparent mounting, selective encryption, filename encryption
- **Maintained**: Active development and security updates
- **HPC-friendly**: FUSE-based, works with shared filesystems like BeeGFS
- **Audited**: Modern implementation with community review

## Threat Model: What gocryptfs Protects Against

**gocryptfs DOES protect**:
- Disk theft: Encrypted data useless without passphrase
- Storage misconfiguration: Exposed encrypted data is unreadable
- Decommissioned hardware: Data unrecoverable
- Backup breach: Encrypted backup still encrypted
- Casual unauthorized access: Cannot read files without passphrase
- Compliance: Satisfies RESTRICTED data encryption requirements

**gocryptfs DOES NOT protect**:
- File sizes (visible in cipher directory)
- Directory structure (folder names visible)
- Timestamps (modification times not encrypted)
- Root access (system admin can read mounted plaintext)
- Malware on your machine (can read mounted plaintext)
- Physical attacks (plaintext in RAM during active session)
- Network traffic (use SSH separately)

**Design principle**: Use defense-in-depth. Encryption protects stored data, but combine with strong passphrases, SSH keys, MFA, and safe practices.

## Data Flow

When you use gocryptfs:
1. **Initialize**: `gocryptfs --init /cipher` → creates gocryptfs.conf, generates master key, prompts for passphrase
2. **Mount**: `gocryptfs /cipher /plain` → derives key from passphrase, verifies master key, mounts FUSE filesystem
3. **Write**: Data flows plaintext → FUSE → gocryptfs → AES-256-GCM → encrypted disk storage
4. **Read**: Data flows encrypted disk → AES-256-GCM → gocryptfs → FUSE → plaintext to your command
5. **Unmount**: `fusermount -u /plain` → plain directory becomes empty, encrypted data remains on disk

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Architecture Quiz

Match each concept to its role. For each, explain in 1-2 sentences.

1. **AES-256-GCM** ← What is this component?
2. **Argon2** ← What is this component?
3. **gocryptfs.conf** ← What is this component?
4. **Cipher directory** ← What is this component?
5. **Plain directory** ← What is this component?

**Options** (use each once):
- A. Configuration file containing encrypted master key and key derivation parameters
- B. Symmetric encryption algorithm providing confidentiality and authenticity
- C. Key derivation function that converts your passphrase into a cryptographic key
- E. On-disk encrypted storage containing encrypted files with encrypted names
- F. In-memory decrypted view of files that appears when cipher directory is mounted

:::::::::::::::::::::::::::::::::::: solution

1. **AES-256-GCM** ← B. Symmetric encryption algorithm providing confidentiality and authenticity
2. **Argon2** ← C. Key derivation function that converts your passphrase into a cryptographic key
3. **gocryptfs.conf** ← A. Configuration file containing encrypted master key and key derivation parameters
4. **Cipher directory** ← E. On-disk encrypted storage containing encrypted files with encrypted names
5. **Plain directory** ← F. In-memory decrypted view of files that appears when cipher directory is mounted

**Explanation**: gocryptfs uses AES-256-GCM (B) to encrypt/decrypt file contents, uses Argon2 (C) to derive encryption keys from your passphrase, stores metadata in gocryptfs.conf (A), and maintains encrypted data on disk (E) while presenting a decrypted view in memory (F).

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Design a Threat Model

For your research project, identify what gocryptfs protects against and what it doesn't:

**Your scenario**: You're a CS student working on a capstone project with a local tech company. The company has provided you with proprietary source code and design documents under an NDA. You store this on Sagehen in `/bigdata/capstone/encrypted` using gocryptfs. Your Sagehen account is accessed via SSH from your laptop.

**Questions**:
1. **Disk theft**: If someone steals your laptop hard drive, can they read your encrypted capstone code?
2. **SSH compromise**: If an attacker gains SSH access to your Sagehen account, can they read your encrypted data?
3. **Credential theft**: If someone steals your SSH private key and logs in as you, can they read encrypted files?
4. **Network eavesdropping**: If an attacker intercepts your SCP transfer, do they learn your capstone code?
5. **Memory attack**: While you're working with encrypted files, can an attacker extract plaintext from your RAM?
6. **Backup breach**: If Sagehen's backup system is breached, is your encrypted data at risk?
7. **FUSE bug**: If a FUSE bug is discovered, could it leak plaintext?

**For each, explain**:
- Is it a risk (Yes/No)?
- Does gocryptfs protect against it (Yes/No/Partial)?
- What additional controls help mitigate it?

:::::::::::::::::::::::::::::::::::: solution

1. **Disk theft of laptop hard drive**: 
   - Risk: Yes (if your laptop is stolen)
   - gocryptfs protection: Yes (encrypted data on disk is useless without passphrase)
   - Additional controls: LUKS/BitLocker on laptop; strong passphrase

2. **SSH compromise (attacker logs in as you)**:
   - Risk: Yes (if credentials are compromised)
   - gocryptfs protection: Partial (encrypted files remain encrypted, but if mounted, plaintext is accessible to logged-in account)
   - Additional controls: Unmount encrypted directories when leaving; use MFA (DUO); SSH keys instead of passwords; monitor for unauthorized logins; disable password access

3. **Stolen SSH private key**:
   - Risk: Yes (SSH key grants full access)
   - gocryptfs protection: Partial (encrypted data protected, but mounting requires passphrase; if mounted, plaintext accessible)
   - Additional controls: Passphrase-protect SSH private key; keep SSH key separate from encrypted passphrase; rotate SSH keys if leaked

4. **Network eavesdropping on SCP transfer**:
   - Risk: No (SSH encrypts transport)
   - gocryptfs protection: Not applicable (SSH handles transport)
   - Additional controls: Always use SSH/SCP (never FTP); verify host keys

5. **Memory attack during active session**:
   - Risk: Yes (if attacker has physical access during work)
   - gocryptfs protection: No (plaintext is in RAM while you're working)
   - Additional controls: Physical security; power off machine when leaving; don't work in public spaces with sensitive data

6. **Backup breach**:
   - Risk: Unlikely but possible
   - gocryptfs protection: Yes (backup of cipher directory is still encrypted)
   - Additional controls: Assume backups may be breached; encrypt even backup data

7. **FUSE bug leaking plaintext**:
   - Risk: Extremely unlikely (FUSE is mature, audited)
   - gocryptfs protection: By design, but a bug could break it
   - Additional controls: Keep gocryptfs updated; report any suspicious behavior to its-hpc@pomona.edu; use standard Unix file permissions as backup layer

**Conclusion**: gocryptfs protects against disk theft, misconfiguration, and backup breaches. It does NOT protect against active compromise (malware, SSH breach with mount active) or physical attacks on powered-on machines. Use defense-in-depth: encryption + strong credentials + secure practices.

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- AES-256-GCM is unbreakable: 2^256 possible keys, authenticated encryption, hardware-accelerated
- Argon2 is memory-hard: slows down password cracking by making it time-consuming per guess
- gocryptfs.conf is irreplaceable: losing it means losing all data, even with passphrase
- File-level encryption (not block-level) is necessary for HPC shared storage (no admin required)
- Filename encryption prevents information leakage even to users with filesystem access
- Performance impact is negligible: ~5-10% overhead, hidden by computation in typical research workloads
- gocryptfs is optimal for HPC clusters: secure, accessible, practical, maintained, cluster-friendly
- gocryptfs DOES protect against: disk theft, misconfiguration, backup breach, casual access
- gocryptfs DOES NOT protect against: active malware, SSH compromise with mount active, memory attacks, root access
- Defense-in-depth: encryption + strong passphrases + SSH keys + MFA + safe practices
::::::::::::::::::::::::::::::::::::::::::::::::
