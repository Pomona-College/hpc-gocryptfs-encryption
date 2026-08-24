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
$ ls -la /bigdata/lab/<labname>/project_data_cipher/
```

**gocryptfs.conf** (314 bytes):
- Contains encrypted master key and scrypt parameters (cipher: AES-256-GCM)
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
$ gocryptfs /bigdata/lab/<labname>/project_data_cipher /scratch/$USER/project_data_plain
Password: [your 14+ character password]
```

Successful output:
```
Master key decrypted successfully.
Mounting 'gocryptfs' to '/scratch/$USER/project_data_plain'
Filesystem mounted successfully.
```

**What happened**:
1. gocryptfs read gocryptfs.conf and derived a key from your password using scrypt
2. Decrypted the master key stored in gocryptfs.conf
3. Started FUSE daemon and mounted the cipher directory

### Verify the Mount

::::::::::::::::::::::::::::::::::::: callout
## Verification Idiom: `mountpoint -q` for Scripts
For verification logic in scripts, prefer `mountpoint -q $MOUNT_POINT` (exact, no false positives). Use `mount | grep` only for interactive (human) inspection like the example below.
::::::::::::::::::::::::::::::::::::::::::::::::

```bash
$ mount | grep gocryptfs    # human-inspection idiom — for scripts use mountpoint -q
gocryptfs on /scratch/$USER/project_data_plain type fuse.gocryptfs (...)

$ ls -la /scratch/$USER/project_data_plain/
total 0
drwx------ 2 username groupname 0 Apr 09 14:25 .
```

The plain directory is empty and ready for your files.

## Testing Encryption

### Create and Verify a Test File

Create a test file in the plain directory:

```bash
$ echo "Research data" > /scratch/$USER/project_data_plain/test_file.txt
$ cat /scratch/$USER/project_data_plain/test_file.txt
Research data
```

File is readable when mounted (automatic decryption). Now check how it appears encrypted:

```bash
$ ls -la /bigdata/lab/<labname>/project_data_cipher/
```

Output shows:
```
-rw-r--r-- 1 username groupname  314 Apr 09 14:25 gocryptfs.conf
-rw-r--r-- 1 username groupname   16 Apr 09 14:25 gocryptfs.diriv
-rw-r--r-- 1 username groupname   41 Apr 09 14:26 gocryptfs.longfilename.sGzMQ_X8=6P
```

The filename `test_file.txt` is completely encrypted. Try reading it directly:

```bash
$ cat /bigdata/lab/<labname>/project_data_cipher/gocryptfs.longfilename.sGzMQ_X8=6P
[Binary garbage - 41 bytes of encrypted AES-256-GCM data, not readable]
```

### Verify Decryption on Remount

Unmount and remount to confirm data persists:

```bash
$ fusermount -u /scratch/$USER/project_data_plain
$ ls /scratch/$USER/project_data_plain/
[empty directory]

$ gocryptfs /bigdata/lab/<labname>/project_data_cipher /scratch/$USER/project_data_plain
Password: [your password]
Filesystem mounted successfully.

$ cat /scratch/$USER/project_data_plain/test_file.txt
Research data
```

**Verification complete**: Your file was encrypted, stored safely, and correctly decrypted on remount.

## Directory Permissions

Both cipher and plain directories must have 700 permissions (drwx------):

```bash
$ ls -ld /bigdata/lab/<labname>/project_data_cipher
drwx------ 2 username groupname 4096 Apr 09 14:25 /bigdata/lab/<labname>/project_data_cipher
```

**Why 700 is correct**:
- Only you can access the encrypted files
- FUSE daemon (running as you) can read gocryptfs.conf
- Prevents other users from attempting password attacks
- Security best practice

**What if permissions are wrong** (e.g., 755):
- Any user can read gocryptfs.conf and attempt password-cracking offline
- Fix: `chmod 700 /bigdata/lab/<labname>/project_data_cipher`

## Common Mistakes

::::::::::::::::::::::::::::::::::::: callout

**Mistake 1: Initializing in the wrong directory**
- Wrong: `gocryptfs -init /scratch/$USER/project_data_plain` (plain dir should be mount point, not cipher)
- Fix: Initialize cipher dir only: `gocryptfs -init /bigdata/lab/<labname>/project_data_cipher`

**Mistake 2: Weak password**
- Falls below Pomona's 14+ character requirement
- Fix: Use 14+ characters with uppercase, lowercase, numbers, symbols

**Mistake 3: Nested directories**
- Wrong: Creating cipher inside plain or vice versa
- Fix: Keep cipher and plain directories as siblings at same level

**Mistake 4: Insecure permissions**
- Wrong: `chmod 755 /bigdata/lab/<labname>/project_data_cipher`
- Fix: `chmod 700 /bigdata/lab/<labname>/project_data_cipher`

::::::::::::::::::::::::::::::::::::::::::::::::

## Backing Up gocryptfs.conf

**Critical step**: Back up gocryptfs.conf immediately. Without it, encrypted data is unrecoverable.

Create backups in multiple locations:

```bash
# Backup 1: Different storage system (/rhome)
mkdir -p /rhome/<myusername>/gocryptfs_backups
cp /bigdata/lab/<labname>/project_data_cipher/gocryptfs.conf \
   /rhome/<myusername>/gocryptfs_backups/gocryptfs.conf.backup

# Backup 2: Date-stamped for version tracking
cp /bigdata/lab/<labname>/project_data_cipher/gocryptfs.conf \
   /rhome/<myusername>/gocryptfs_backups/gocryptfs.conf.2026-04-09

# Backup 3: External drive (if available)
cp /bigdata/lab/<labname>/project_data_cipher/gocryptfs.conf \
   /mnt/external_drive/gocryptfs.conf.backup
```

Verify:
```bash
$ ls -la /rhome/<myusername>/gocryptfs_backups/
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
   - Cipher: `/bigdata/lab/<labname>/myresearch_cipher`
   - Plain: `/scratch/$USER/myresearch_plain`

2. Create both directories and verify they're empty

3. Create a strong password (14+ characters, mixed types, memorable)

4. Initialize gocryptfs: `gocryptfs -init /bigdata/lab/<labname>/myresearch_cipher`

5. Verify initialization: gocryptfs.conf and gocryptfs.diriv exist with 700 permissions

6. Mount: `gocryptfs /bigdata/lab/<labname>/myresearch_cipher /scratch/$USER/myresearch_plain`

7. Create a test file with content in the plain directory

8. Verify the file appears encrypted (long hash filename) in the cipher directory

9. Try to read the encrypted file—you should see binary garbage

10. Back up gocryptfs.conf to `/rhome/<myusername>/backups/`

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
- Backup exists in `/rhome/<myusername>/backups/` with same size as original
- After unmount, plain directory empty; after remount, test file reappears

**Common issues**:

- "Permission denied": Check that you own `/bigdata/lab/<labname>/`
- "Plain directory not empty": Must be completely empty; remove hidden files
- "Password rejected": Verify it's 14+ characters with mixed types
- "Mount fails": Check that gocryptfs.conf and gocryptfs.diriv exist
- "Can't find quota_check.sh": Use the full path, or contact its-hpc@pomona.edu

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

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
