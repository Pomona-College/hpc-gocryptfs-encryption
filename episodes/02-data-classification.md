---
title: "Pomona's Data Classification System"
teaching: 10
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions
- What data requires encryption at Pomona?
- How do I classify my research data?
- What does gocryptfs protect against?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand Pomona's three-tier data classification system
- Learn what data falls into each tier (PUBLIC, PROPRIETARY, RESTRICTED)
- Know what gocryptfs protects and doesn't protect
- Understand why encryption alone is not enough
- Learn what "beyond compliance" means for your research
::::::::::::::::::::::::::::::::::::::::::::::::

## Understanding Pomona's Three-Tier Data Classification System

Pomona College ITS Policy 24 defines three data tiers. Understanding these is essential for knowing when encryption is required.

### Tier 1: PUBLIC Data (Green, Unix permissions 755)

**What it includes**:
- Published research papers
- Anonymized aggregated statistics
- Marketing materials
- Lecture notes
- Published code

**Where to store**: /bigdata (no encryption required)

**Who can access**: Everyone

**If exposed**: No impact

**Encryption**: Not required

### Tier 2: PROPRIETARY Data (Orange, Unix permissions 750)

**What it includes**:
- Unpublished research data (pre-publication)
- Preliminary analysis results
- Code under development
- Industry collaboration data (non-confidential)
- Institutional information (committee records)
- Internal grant proposals

**Where to store**: /bigdata or /rhome with group access controls

**Who can access**: Research group members

**If exposed**: Competitive disadvantage, loss of publication advantage

**Encryption**: Recommended but not mandated

**Example**: Your dataset analyzing crime patterns before publication—valuable because unpublished, but doesn't directly harm individuals if leaked.

### Tier 3: RESTRICTED Data (Red, Unix permissions 700+gocryptfs)

**What it includes**:
- FERPA-protected student records (IDs, grades, transcripts)
- Medical information (health records, psychological counseling notes)
- Personally identifiable information + sensitive data (names + financial info)
- Export-controlled research data (ITAR/EAR)
- Proprietary data under NDA or confidentiality agreement
- Biometric data
- Social Security numbers, passport numbers
- Financial account information

**Where to store**: /bigdata or /rhome with gocryptfs encryption

**Who can access**: Only authorized researchers with passphrase

**If exposed**: Legal liability, regulatory fines, personal harm to subjects

**Encryption**: MANDATORY

**Example**: Student survey responses with ID numbers; health screening data from a clinical study; export-controlled research in materials science or cryptography.

::::::::::::::::::::::::::::::::::::: callout
## Legal Requirement: Not Optional

Encryption for RESTRICTED data is not just policy; it's federal law. FERPA, HIPAA, and EAR violations carry:
- **Personal legal liability** (you, not just the institution)
- **Criminal penalties** (up to $250,000+ per HIPAA violation)
- **Research suspension** (immediate halting of all related work)
- **Career consequences** (difficulty securing future funding)

Failing to encrypt RESTRICTED data can result in disciplinary action, account suspension, and legal consequences. **If you have ANY doubt about whether your data needs encryption, encrypt it.** Better safe than facing federal investigators.
::::::::::::::::::::::::::::::::::::::::::::::::

## What gocryptfs Protects

### What It DOES Protect

- **File contents**: Data at rest encrypted with AES-256-GCM (unbreakable with current technology)
- **Accidental access**: Unauthorized users cannot read encrypted data without passphrase
- **Laptop theft**: If your computer is stolen, encrypted data remains secure
- **Storage media disposal**: Old drives or decommissioned systems can be safely discarded
- **Compliance with regulations**: Satisfies FERPA, HIPAA, EAR, and NIST requirements
- **Filename privacy**: Even filenames are encrypted (only you see the real structure)
- **Storage misconfiguration**: Even if accidentally exposed, encrypted files are unreadable

### What It DOES NOT Protect

- **Active sessions**: While mounted, data is accessible in plaintext to anyone on your account
- **Weak passwords**: gocryptfs is only as secure as your master passphrase (must be 14+ characters)
- **Network traffic**: Data in transit (SSH, SCP, network copies) is a separate concern (use SSH for transport security)
- **Accidental deletion**: Encryption doesn't prevent rm or accidental file deletion
- **System administrator access**: Root user can access mounted data or read memory
- **Malware**: If your system is infected, malware can read mounted plaintext data
- **Metadata**: File sizes, modification times, and directory structure remain visible
- **Physical attacks**: Forensic recovery from RAM during an active session

::::::::::::::::::::::::::::::::::::: callout
## Important: Encryption Alone Is Not Enough

gocryptfs protects data at rest, but when mounted, anyone with access to your account can read files. It's one essential layer in defense-in-depth security:

**Additional controls you must use**:
- Strong, unique passphrases (14+ characters, not dictionary words)
- Unmount encrypted directories when not actively working
- Use SSH keys (Ed25519 recommended) for cluster access
- Enable MFA (DUO) at https://duo.pomona.edu
- Keep your system and software updated
- Use caution with sensitive data in /scratch or /tmpfs (temporary storage)
- Report suspicious activity immediately to its-hpc@pomona.edu

Encryption secures data against theft and misconfiguration, but good security practices secure the system as a whole.
::::::::::::::::::::::::::::::::::::::::::::::::

## Beyond Compliance: Why Encryption Matters for Your Research

While regulatory compliance is the minimum requirement, encryption provides additional benefits:

### Research Integrity and Trust

Encrypted data demonstrates that you take data stewardship seriously. Peer reviewers, collaborators, and funding agencies increasingly expect encryption for sensitive data. It signals institutional rigor.

### Competitive Advantage

For unpublished research, encryption protects your competitive advantage. A storage misconfiguration or a compromised credentials shouldn't expose months of work before publication.

### Publication and Reproducibility

Many top-tier journals (Nature, Science, PNAS) now require evidence of data security. Encrypted data at rest is standard practice.

### Funding Agency Requirements

NSF, NIH, DOE, and other agencies increasingly require data management plans that include encryption. You may not be allowed to submit grants without demonstrating secure data handling.

### Personal Liability Protection

By implementing encryption, you demonstrate due care. In the event of a breach, showing you took reasonable precautions protects you legally.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Classify These Datasets

For each dataset below, classify it as **PUBLIC**, **PROPRIETARY**, or **RESTRICTED** under Pomona's ITS Policy 24. Explain your reasoning.

1. A CSV file of published climate measurements downloaded from NOAA for use in an environmental science course.
2. An unpublished draft manuscript and supporting statistical analysis for a sociology paper under peer review.
3. A spreadsheet containing student names, ID numbers, and final grades for a research study on academic performance.
4. Genomic sequencing data from a clinical trial, linked to participant medical records and diagnoses.
5. Source code for a custom R package your lab developed but has not yet released publicly.

::::::::::::::::::::::::::::::::::::: solution

## Solution

1. **PUBLIC** (Green / 755) — Published government data freely available online. No encryption needed. Store on `/bigdata` with standard permissions.
2. **PROPRIETARY** (Orange / 750) — Unpublished research with competitive value. Exposure would mean loss of publication priority, but no legal liability or personal harm. Encryption recommended but not mandated.
3. **RESTRICTED** (Red / 700 + gocryptfs) — Contains student names and ID numbers, which are FERPA-protected personally identifiable information. Encryption is **mandatory**.
4. **RESTRICTED** (Red / 700 + gocryptfs) — Medical records and diagnoses are HIPAA-protected ePHI. This is the highest-risk category. Encryption is **mandatory**, and additional access controls are required.
5. **PROPRIETARY** (Orange / 750) — Unpublished code has competitive value but does not contain personally identifiable or regulated information. Encryption recommended, especially if the code implements novel algorithms you plan to publish.

::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Pomona's three-tier system: PUBLIC (no encryption), PROPRIETARY (recommended), RESTRICTED (mandatory)
- RESTRICTED data is legally protected under FERPA, HIPAA, EAR, and NIST requirements
- gocryptfs protects data at rest and prevents accidental exposure from misconfiguration
- Encryption is one layer; defense-in-depth requires strong passphrases, SSH keys, and MFA
- Encryption supports research integrity, publication, and funding compliance
- If you have doubt about data classification, encrypt it
::::::::::::::::::::::::::::::::::::::::::::::::
