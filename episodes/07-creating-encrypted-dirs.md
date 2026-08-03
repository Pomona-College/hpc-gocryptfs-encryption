---
title: "Verifying and Managing Encrypted Directories"
teaching: 15
exercises: 15
---

:::::::::::::::::::::::::::::::::::::: questions
- What files does gocryptfs create during initialization?
- How do you verify your encrypted directory is working?
- What are the correct directory permissions?
- What are the common mistakes when creating encrypted directories?
- Why is backing up gocryptfs.conf critical?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand the complete initialization process and files created
- Examine and interpret gocryptfs.conf and gocryptfs.diriv
- Mount and verify encryption with test files
- Verify encryption and decryption works correctly
- Understand directory permissions and why they matter
- Identify common mistakes and how to avoid them
- Back up gocryptfs.conf correctly in multiple locations
::::::::::::::::::::::::::::::::::::::::::::::::

## Examining Initialization Files

gocryptfs creates two critical files during initialization:

```bash
$ ls -la /bigdata/groupname/project_data_cipher/
```

**gocryptfs.conf** (314 bytes):
- Contains encrypted master key and Argon2 parameters (cipher: AES-256-GCM)
- Plain text JSON format, but key is encrypted
- **MUST be backed up immediately**

**gocryptfs.diriv** (16 bytes):
- Directory IV (initialization vector) for file name encryption
- Used alongside the master key for authenticated encryption
- Must be backed up with gocryptfs.conf

**Directory permissions** (700 = drwx------):
- Owner: read, write, execute access
- Group and others: no access
- Enforced for security

## Mounting and Verifying Encryption

Mount the cipher directory to access your files:

```bash
$ gocryptfs /bigdata/groupname/project_data_cipher /bigdata/groupname/project_data_plain
Password: [your 14+ character password]
```

Successful output:
```
Master key decrypted successfully.
Mounting 'gocryptfs' to '/bigdata/groupname/project_data_plain'
Filesystem mounted successfully.
```

**What happened**:
1. gocryptfs read gocryptfs.conf and derived key from your password using Argon2
2. Decrypted the master key stored in gocryptfs.conf
3. Started FUSE daemon and mounted the cipher directory

### Verify the Mount

::::::::::::::::::::::::::::::::::::: callout
## Verification Idiom: `mountpoint -q` for Scripts
For verification logic in scripts, prefer `mountpoint -q $MOUNT_POINT` (exact, no false positives). Use `mount | grep` only for interactive (human) inspection like the example below.
::::::::::::::::::::::::::::::::::::::::::::::::

```bash
$ mount | grep gocryptfs    # human-inspection idiom — for scripts use mountpoint -q
gocryptfs on /bigdata/groupname/project_data_plain type fuse.gocryptfs (...)

$ ls -la /bigdata/groupname/project_data_plain/
total 0
drwx------ 2 username groupname 0 Apr 09 14:25 .
```

The plain directory is empty and ready for your files.

## Testing Encryption

### Create and Verify a Test File

Create a test file in the plain directory:

```bash
$ echo "Research data" > /bigdata/groupname/project_data_plain/test_file.txt
$ cat /bigdata/groupname/project_data_plain/test_file.txt
Research data
```

File is readable when mounted (automatic decryption). Now check how it appears encrypted:

```bash
$ ls -la /bigdata/groupname/project_data_cipher/
```

Output shows:
```
-rw-r--r-- 1 username groupname  314 Apr 09 14:25 gocryptfs.conf
-rw-r--r-- 1 username groupname   16 Apr 09 14:25 gocryptfs.diriv
-rw-r--r-- 1 username groupname   41 Apr 09 14:26 gocryptfs.longfilename.sGzMQ_X8=6P
```

The filename `test_file.txt` is completely encrypted. Try reading it directly:

```bash
$ cat /bigdata/groupname/project_data_cipher/gocryptfs.longfilename.sGzMQ_X8=6P
[Binary garbage - 41 bytes of encrypted AES-256-GCM data, not readable]
```

### Verify Decryption on Remount

Unmount and remount to confirm data persists:

```bash
$ fusermount -u /bigdata/groupname/project_data_plain
$ ls /bigdata/groupname/project_data_plain/
[empty directory]

$ gocryptfs /bigdata/groupname/project_data_cipher /bigdata/groupname/project_data_plain
Password: [your password]
Filesystem mounted successfully.

$ cat /bigdata/groupname/project_data_plain/test_file.txt
Research data
```

**Verification complete**: Your file was encrypted, stored safely, and correctly decrypted on remount.

## Directory Permissions

Both cipher and plain directories must have 700 permissions (drwx------):

```bash
$ ls -ld /bigdata/groupname/project_data_cipher
drwx------ 2 username groupname 4096 Apr 09 14:25 /bigdata/groupname/project_data_cipher
```

**Why 700 is correct**:
- Only you can access the encrypted files
- FUSE daemon (running as you) can read gocryptfs.conf
- Prevents other users from attempting password attacks
- Security best practice

**What if permissions are wrong** (e.g., 755):
- Any user can read gocryptfs.conf and attempt password-cracking offline
- Fix: `chmod 700 /bigdata/groupname/project_data_cipher`

## Common Mistakes

::::::::::::::::::::::::::::::::::::: callout

**Mistake 1: Initializing in the wrong directory**
- Wrong: `gocryptfs -init /bigdata/groupname/project_data_plain` (plain dir should be mount point, not cipher)
- Fix: Initialize cipher dir only: `gocryptfs -init /bigdata/groupname/project_data_cipher`

**Mistake 2: Weak password**
- Falls below Pomona's 14+ character requirement
- Fix: Use 14+ characters with uppercase, lowercase, numbers, symbols

**Mistake 3: Nested directories**
- Wrong: Creating cipher inside plain or vice versa
- Fix: Keep cipher and plain directories as siblings at same level

**Mistake 4: Insecure permissions**
- Wrong: `chmod 755 /bigdata/groupname/project_data_cipher`
- Fix: `chmod 700 /bigdata/groupname/project_data_cipher`

::::::::::::::::::::::::::::::::::::::::::::::::

## Backing Up gocryptfs.conf

**Critical step**: Back up gocryptfs.conf immediately. Without it, encrypted data is unrecoverable.

Create backups in multiple locations:

```bash
# Backup 1: Different storage system (/rhome)
mkdir -p /rhome/username/gocryptfs_backups
cp /bigdata/groupname/project_data_cipher/gocryptfs.conf \
   /rhome/username/gocryptfs_backups/gocryptfs.conf.backup

# Backup 2: Date-stamped for version tracking
cp /bigdata/groupname/project_data_cipher/gocryptfs.conf \
   /rhome/username/gocryptfs_backups/gocryptfs.conf.2026-04-09

# Backup 3: External drive (if available)
cp /bigdata/groupname/project_data_cipher/gocryptfs.conf \
   /mnt/external_drive/gocryptfs.conf.backup
```

Verify:
```bash
$ ls -la /rhome/username/gocryptfs_backups/
-rw-r--r-- 1 username groupname 314 Apr 09 14:30 gocryptfs.conf.backup
-rw-r--r-- 1 username groupname 314 Apr 09 14:30 gocryptfs.conf.2026-04-09
```

::::::::::::::::::::::::::::::::::::: callout
## Do This Right Now: Back Up gocryptfs.conf

Stop here and complete backups before proceeding. This 1-minute action prevents data loss. Multiple backups in different locations are your only safety net if something goes wrong.
::::::::::::::::::::::::::::::::::::::::::::::::

## Challenges

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Guided Initialization and Verification

Complete this step-by-step exercise:

1. Plan your directories:
   - Cipher: `/bigdata/groupname/myresearch_cipher`
   - Plain: `/bigdata/groupname/myresearch_plain`

2. Create both directories and verify they're empty

3. Create a strong password (14+ characters, mixed types, memorable)

4. Initialize gocryptfs: `gocryptfs -init /bigdata/groupname/myresearch_cipher`

5. Verify initialization: gocryptfs.conf and gocryptfs.diriv exist with 700 permissions

6. Mount: `gocryptfs /bigdata/groupname/myresearch_cipher /bigdata/groupname/myresearch_plain`

7. Create a test file with content in the plain directory

8. Verify the file appears encrypted (long hash filename) in the cipher directory

9. Try to read the encrypted file—you should see binary garbage

10. Back up gocryptfs.conf to `/rhome/username/backups/`

11. Unmount and remount; verify your test file reappears with original name

Record any issues and how you resolved them.

:::::::::::::::::::::::::::::::::::: solution

**Expected outcomes**:

- Both directories created, empty with `ls -la`
- Password accepted by gocryptfs
- gocryptfs.conf (314 bytes) and gocryptfs.diriv (16 bytes) with 700 permissions
- Mount succeeds: "Filesystem mounted successfully"
- File created and readable in plain directory
- Cipher directory shows encrypted filename (gocryptfs.longfilename.XXXXX)
- Encrypted file content is unreadable binary data
- Backup exists in /rhome/username/backups/ with same size as original
- After unmount, plain directory empty; after remount, test file reappears

**Common issues**:

- "Permission denied": Check that you own /bigdata/groupname/
- "Plain directory not empty": Must be completely empty; remove hidden files
- "Password rejected": Verify it's 14+ characters with mixed types
- "Mount fails": Check that gocryptfs.conf and gocryptfs.diriv exist
- "Can't find quota_check.sh": Use full path or check module load

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Initialization creates gocryptfs.conf (encrypted key) and gocryptfs.diriv (directory IV)
- Verify encryption: encrypted files unreadable in cipher directory, readable in plain directory
- Directory permissions must be 700 (drwx------) on both cipher and plain directories
- Back up gocryptfs.conf immediately in multiple locations—it's your only access key
- Common mistakes: wrong directory, weak password, no backup, nested directories, insecure permissions
- Encryption is transparent: work in plain directory; encryption/decryption happen automatically
- Test remounting to verify data persists and decrypts correctly
- Keep file permissions at 700—don't make directories world-readable
::::::::::::::::::::::::::::::::::::::::::::::::
