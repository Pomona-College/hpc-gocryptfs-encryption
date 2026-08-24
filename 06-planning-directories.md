---
title: "Planning Your Encrypted Directories"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions
- How do you plan and name your cipher and plain directories?
- Where should encrypted directories be stored?
- What storage quota implications does encryption have?
- How do you check storage usage on BeeGFS?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Plan directory structure using naming conventions and storage location guidelines
- Understand quota implications and how to check them
- Create strong master passwords meeting Pomona's 14+ character requirement
- Complete the initialization process and understand what files are created
::::::::::::::::::::::::::::::::::::::::::::::::

## Planning Your Encrypted Directories

Before you create anything, planning is essential. A few minutes now prevents confusion and mistakes later.

### Naming Conventions

Use clear, consistent naming to distinguish cipher and plain directories:

**Recommended suffix approach**:
- Cipher directory: `project_data_cipher`
- Plain directory: `project_data_plain`

**Alternative approach**:
- Cipher directory: `project_data.encrypted`
- Plain directory: `project_data.working`

**Why naming matters**:
- You immediately know which is which (critical!)
- Prevents mounting plain over cipher (data loss)
- Helps with documentation and collaboration
- Scripts can rely on naming patterns

### Storage Location Strategy

**For large datasets (>100 MB)**:
- Use `/bigdata/lab/<labname>/` for both directories
- BeeGFS performance is optimized for large files
- Shares 1TB lab quota with /rhome, so plan quota use
- Better for datasets you'll access frequently

**For small configs or keys (<100 MB)**:
- Can use `/rhome/<myusername>/` for both directories
- Separate from /bigdata quota concerns
- Still shares 1TB lab quota with /bigdata total
- Good for test cases or small research data

**Never use**:
- `/scratch`: Data deleted when job completes (defeats encryption purpose)
- `/tmpfs`: RAM-backed, deleted at session end
- Root directories: You don't have write permission

### Storage Quota Implications

**Important reality**: Encryption adds minimal overhead
- Encrypted files are approximately the same size as originals
- Overhead: ~1% per file for metadata and IV (initialization vector)
- Example: 1 GB dataset becomes ~1.01 GB when encrypted

**Planning your quota**:
- `/rhome` + `/bigdata` together = 1 TB lab quota
- Example breakdown:
  - `/rhome`: 50 GB for config, code, small datasets
  - `/bigdata`: 950 GB for large research datasets
  - Check current usage: `quota_check.sh` (not `du`!)

::::::::::::::::::::::::::::::::::::: callout
## BeeGFS Storage: Always Use quota_check.sh

On BeeGFS (/bigdata), the standard `du` command gives incorrect results. Always use `quota_check.sh` instead:

```bash
quota_check.sh
```

The `-apparent-size` flag is already built in. Running `du --apparent-size` on BeeGFS is still not accurate—use the quota script.
::::::::::::::::::::::::::::::::::::::::::::::::

## Creating Your First Encrypted Directory

### Step 1: Create the Cipher Directory

First, create the directory that will hold encrypted files. This must be empty at initialization time.

```bash
$ mkdir -p /bigdata/lab/<labname>/project_data_cipher
$ ls -ld /bigdata/lab/<labname>/project_data_cipher
drwxr-xr-x 2 username groupname 4096 Apr 09 14:22 /bigdata/lab/<labname>/project_data_cipher
```

Permissions here don't matter yet—initialization will handle them.

### Step 2: Create the Plain Directory (on /scratch, from a compute session)

Create the plain (working) directory where you'll access decrypted files. This must also be empty initially.

::::::::::::::::::::::::::::::::::::::: callout

## The Plain Directory Must Live on /scratch — Not /bigdata

`/bigdata` is a BeeGFS network filesystem, and FUSE refuses to use a network
filesystem as a mountpoint. If you try to mount onto a plain directory under
`/bigdata`, gocryptfs fails with a mount error. The rule:

- **Cipher directory** → `/bigdata/lab/<labname>/...` (persistent, backed up — ordinary files, so BeeGFS is fine)
- **Plain directory (mountpoint)** → `/scratch/$USER/...` (node-local)

Because `/scratch` is node-local and cleaned between jobs, you must be **inside
a compute session** (`srun` or `sbatch`) to create it and mount — not on the
login node — and you re-create the plain directory at the start of each session.
Nothing decrypted is ever written to `/scratch`: the plain view is virtual, and
writes into it land encrypted in the cipher directory on `/bigdata`.

:::::::::::::::::::::::::::::::::::::::::::::::

```bash
# From inside an interactive compute session (srun --pty bash):
$ mkdir -p /scratch/$USER/project_data_plain
$ ls -ld /scratch/$USER/project_data_plain
drwxr-xr-x 2 username groupname 4096 Apr 09 14:22 /scratch/$USER/project_data_plain
```

### Step 3: Create a Strong Master Password

This is the most critical step. Your password protects all your encrypted data.

**Pomona requirement**: 14+ characters minimum (NIST SP 800-63B standard)

**Good password characteristics**:
- 14+ characters long
- Mix of uppercase, lowercase, numbers, symbols
- Not a dictionary word or variation of one
- Not based on personal information
- Unique to this encryption (don't reuse passwords)
- Memorable but not obvious

**Good password examples** (NOT to use—make your own):
- `2Gr1ff0n$Secure&Data` (14 chars: mix of all types)
- `April!Research2026@Pomona` (25 chars: phrase-based with additions)
- `B1geData#Secure$Lab2026` (23 chars: organization-specific)

**Weak password examples** (NEVER use these):
- `mypassword123` (too simple, dictionary word)
- `Pomona2026` (personal information + year)
- `secure` (single word, too short)
- `12345678901234` (numbers only, not memorable)

**Creating a strong passphrase**:

Instead of random characters, use a longer memorable phrase with modifications:

1. Start with a phrase: "My research group studies cryptography at Pomona"
2. Take first letter of each word: `MrgsaP`
3. Add numbers and symbols: `MrgsaP@2026!`
4. Still short—extend it: `MrgsaP@2026!CryptographyLab`

This passphrase is 27 characters and memorable to you while meeting complexity requirements.

::::::::::::::::::::::::::::::::::::: callout
## Warning: Password Cannot Be Recovered

Choose your password carefully. If you forget it, there is no recovery method. Your encrypted data will be permanently inaccessible. Memorize the password or store it in an encrypted password manager. Write it down ONLY in a secure physical location (not email, not digital files on your computer).
::::::::::::::::::::::::::::::::::::::::::::::::

### Step 4: Initialize gocryptfs

::::::::::::::::::::::::::::::::::::::: callout

## Load the gocryptfs Module First

Sagehen provides gocryptfs through Lmod (currently `gocryptfs/2.6.1`). Before
the steps below, load it and confirm the command is available:

```bash
$ module load gocryptfs
$ which gocryptfs
```

If `which gocryptfs` comes back empty even after loading the module, contact
its-hpc@pomona.edu. Remember to load the module in every new session -- and in
any SLURM script -- where you mount or initialize an encrypted directory.

:::::::::::::::::::::::::::::::::::::::::::::::

Now run the initialization command on the cipher directory. gocryptfs will prompt you for the password.

```bash
$ gocryptfs -init /bigdata/lab/<labname>/project_data_cipher
```

Full terminal output:

```
Choose a master password.
Password:
```

Type your password (it won't display as you type):

```
Password: [14+ characters, you type but don't see]
Confirm:
```

Type the same password again to confirm:

```
Confirm: [same password, you type but don't see]
Your master key is protected with scrypt, a memory-hard key derivation function.
Go to the docs folder, chapter "Passwords", to see how to customize the parameters.
gocryptfs filesystem created successfully.
```

::::::::::::::::::::::::::::::::::::: callout
## What Just Happened

gocryptfs executed these steps:
1. Derived a cryptographic key from your password using scrypt (memory-hard, resistant to brute force)
2. Generated a random AES-256-GCM master key
3. Encrypted the master key with the derived password key
4. Created `gocryptfs.conf` in the cipher directory
5. Created `gocryptfs.diriv` (directory IV for authentication)
6. Set directory permissions to 700 (only you can read/write)

Your password was never stored on disk—only the encrypted key is stored.
::::::::::::::::::::::::::::::::::::::::::::::::

## Challenges

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Plan Your Encrypted Directory Structure

Before initializing anything, complete this planning exercise to ensure you choose the right setup.

**Task 1: Decide on naming convention**
- Choose cipher directory name: `project_data_cipher` or `project_data.encrypted`?
- Choose plain directory name: `project_data_plain` or `project_data.working`?
- Write down your choice with reasoning

**Task 2: Decide on storage location**
- What is the expected size of your dataset?
- Should you use `/bigdata/lab/<labname>/` or `/rhome/<myusername>/`?
- Check current quota with `quota_check.sh`
- Will your dataset fit with room to spare?

**Task 3: Create both directories**
- Create cipher directory: `mkdir -p /path/to/cipher_dir`
- Create plain directory on node-local scratch (inside a compute session): `mkdir -p /scratch/$USER/plain_dir`
- Verify both exist and are empty: `ls -la /path/to/`

**Task 4: Create a strong password**
- Generate a 14+ character password using the passphrase method
- Verify it meets all criteria: uppercase, lowercase, numbers, symbols
- Write it down temporarily (you'll need it twice for initialization)

**Record your answers**:
1. Full path to cipher directory:
2. Full path to plain directory:
3. Current quota usage (from quota_check.sh):
4. Space available for your data:
5. Your password (temporarily, for the next exercise):

:::::::::::::::::::::::::::::::::::: solution

**Expected outcomes**:

1. **Naming chosen**:
   - Either suffix approach (\_cipher, \_plain) or alternative approach (.encrypted, .working)
   - Both names chosen and documented

2. **Storage location decided**:
   - /bigdata selected for >100 MB datasets
   - /rhome selected for <100 MB datasets
   - Reasoning documented

3. **Directories created**:
   - Cipher directory exists: `ls -ld /path/cipher_dir` shows it
   - Plain directory exists: `ls -ld /path/plain_dir` shows it
   - Both appear in the parent directory listing
   - Both are empty initially

4. **Strong password created**:
   - 14+ characters verified
   - Contains uppercase, lowercase, numbers, symbols
   - Written down (temporarily, for next exercise)
   - Not a dictionary word or personal information

**Common planning mistakes to avoid**:
- Using /scratch or /tmpfs (data gets deleted!)
- Using /rhome for very large datasets (quota exceeded)
- Weak passwords (too short, dictionary words)
- Similar names for cipher and plain (confusing later)

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Plan directory structure first: use naming conventions to avoid confusion
- Choose storage location based on dataset size: /bigdata for large (>100 MB), /rhome for small (<100 MB)
- Create strong passwords: 14+ characters, mixed types, memorable or in password manager
- Always use quota_check.sh on BeeGFS—du gives wrong results
- Encryption overhead is minimal (~1% per file)
- Never store encrypted directories in /scratch, /tmpfs, or root directories
- Create both cipher and plain directories before initialization
- Password cannot be recovered—choose carefully and store securely
::::::::::::::::::::::::::::::::::::::::::::::::

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
