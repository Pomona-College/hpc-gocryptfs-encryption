---
title: "Scenarios and Responsibilities"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions
- What do real-world research scenarios look like?
- Who is responsible for what in data security?
- What do I do if I suspect a breach?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand real-world encryption scenarios at Pomona
- Know roles and responsibilities (PI, students, ITS, Compliance)
- Learn incident response procedures
- Practice data classification and risk assessment
::::::::::::::::::::::::::::::::::::::::::::::::

## Real-World Scenarios at Pomona

### Scenario 1: Biology Lab with IACUC-Approved Animal Study

Your lab is conducting behavioral research on mice with IACUC approval. Your data includes:
- Animal ID numbers
- Health observations
- Behavioral measurements
- Lab identifiers

**Classification**: RESTRICTED (IACUC data is sensitive and has regulatory requirements)

**Encryption**: YES, mandatory

**Why**: IACUC approval is conditional on data security; loss of approval halts all animal research

**Storage**: Create `/bigdata/iacuc-behavior/encrypted` with gocryptfs

**Passphrase**: 14+ characters (NIST SP 800-63B); 20+ recommended for long-term keys. Not shared with lab members (only you hold the key)

**Access**: Other lab members cannot directly access encrypted files; you export analysis results to shared folder

### Scenario 2: Economics Study with FERPA Student Data

You're conducting a longitudinal study of student financial aid and academic success. Your data includes:
- Pomona student ID numbers
- Grant/loan amounts
- GPA and course history
- Family income (reported in survey)

**Classification**: RESTRICTED (FERPA: student ID + academic data; PII: family income)

**Encryption**: YES, mandatory

**Why**: FERPA violation penalties apply; students' privacy is at stake

**Storage**: Create `/bigdata/econ-financial-study/encrypted` with gocryptfs

**Key backup**: Store passphrase in your personal secure password manager; back up gocryptfs.conf to separate location

**Incident response**: If you suspect the encrypted directory was accessed, contact its-hpc@pomona.edu immediately

### Scenario 3: Chemistry Lab with Export-Controlled Synthesis Data

Your lab synthesizes advanced materials for energy applications. The synthesis procedures are export-controlled under EAR as "advanced ceramics." Your data includes:
- Detailed synthesis procedures
- Characterization results
- Raw experimental data
- Lab notes with specific temperatures, pressures, times

**Classification**: RESTRICTED (export-controlled, NIST CUI)

**Encryption**: YES, mandatory

**Why**: EAR deemed export rule; sharing with foreign nationals is controlled

**Storage**: Create `/bigdata/materials-synthesis/encrypted` with gocryptfs

**Access control**: Only authorize U.S. citizens and permanent residents; do not mount directory if visiting researchers with visa access are present

**Collaboration**: If collaborating with international researchers, provide only redacted/processed results, not raw data

### Scenario 4: CS Student with Industry Collaboration Under NDA

You're working on software development under a research contract with a tech company. The NDA specifies data protection requirements. Your data includes:
- Source code (company intellectual property)
- Design documents
- Benchmark results
- Implementation specifications

**Classification**: RESTRICTED (Proprietary under NDA; confidentiality agreement)

**Encryption**: YES, required by contract

**Why**: NDA breach can trigger legal action by company

**Storage**: Create `/bigdata/company-project/encrypted` with gocryptfs

**Passphrase**: Treat as company confidential; don't use a password you share elsewhere

**Backup**: Notify your advisor of passphrase location (in case of emergency); don't store with unencrypted data

**Incident response**: If encryption is compromised, notify both your advisor and the company immediately

## Who Is Responsible for What?

Understanding roles and responsibilities prevents confusion:

### Your Responsibility (Principal Investigator / Student Researcher)

- **Classify your data**: Determine whether data is PUBLIC, PROPRIETARY, or RESTRICTED
- **Request encryption**: Contact its-hpc@pomona.edu if you need gocryptfs setup help
- **Manage passphrases**: Create, memorize, and secure your passphrase (do not share)
- **Unmount when done**: Always unmount encrypted directories when leaving your workstation
- **Incident reporting**: If you suspect unauthorized access, report immediately

### ITS Research Computing Responsibility

- **Provide tools**: Ensure gocryptfs is available and supported on Sagehen HPC
- **Provide guidance**: Offer documentation and training on encryption setup
- **Verify setup**: Can review encryption configuration upon request
- **Incident support**: Assist with incident response if breach is suspected
- **Monitoring**: Log access to encrypted directories; alert you to suspicious activity

### ITS Compliance Responsibility

- **Policy enforcement**: Verify that RESTRICTED data is encrypted
- **Audit**: Periodic audits of data classifications and encryption status
- **Incident investigation**: If breach occurs, lead investigation
- **Regulatory liaison**: Communicate with FERPA, HIPAA, EAR agencies if needed

## What To Do If You Suspect a Data Breach

**If you suspect your encrypted directory or passphrase has been compromised:**

1. **Stop work immediately** (do not mount the encrypted directory further)
2. **Preserve evidence**: Note the time, system, and any suspicious activity
3. **Contact ITS**: Email its-hpc@pomona.edu with details
   - What makes you suspect a breach?
   - When did you last know the directory was secure?
   - What system or account was involved?
4. **Inform your advisor** (if student researcher) or your department chair (if faculty)
5. **Do NOT change your passphrase yet** (let ITS preserve audit logs first)
6. **Cooperate with investigation**: ITS will review logs and determine scope

**Timeline**:
- ITS will acknowledge within 4 hours (business hours)
- Initial assessment within 24 hours
- If confirmed breach: Pomona legal and compliance teams notified
- Student/subject notification process begins

The key: **Report early and often.** A suspected breach that turns out false is far better than a confirmed breach discovered months later.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Data Classification

For each type of data, decide:
- What tier (PUBLIC, PROPRIETARY, or RESTRICTED)?
- Encryption required (Y/N)?
- Why?

1. Anonymized survey responses from your study (no identifiers retained)
2. Student course grades and student ID numbers
3. Your published paper (already in journals and on your website)
4. Colleague's research code (shared with you via email; they wrote it)
5. Financial data on grant expenses and invoices (showing your institution's costs)
6. Temperature and humidity readings from lab sensors (no identifying information)
7. Raw genomic sequences from study participants (no names, linked by participant ID only)
8. Preliminary analysis results from unpublished work (never shared; competitive advantage)

:::::::::::::::::::::::::::::::::::: solution

1. **Anonymized survey responses (no identifiers)**: PUBLIC—no identifiers means no PII risk
2. **Student grades and ID**: RESTRICTED—FERPA data, must encrypt
3. **Published paper**: PUBLIC—already public; no encryption needed
4. **Colleague's code**: PROPRIETARY—unpublished work with competitive value; encryption recommended but not mandatory
5. **Grant expenses**: PROPRIETARY—institutional data; encryption recommended but not mandatory
6. **Sensor readings**: PUBLIC—non-human data with no identifying information
7. **Genomic sequences with participant ID**: RESTRICTED—if participants can be identified from ID, this is sensitive health data; HIPAA applies; must encrypt
8. **Preliminary unpublished results**: PROPRIETARY—competitive advantage; encryption recommended but not mandatory

**Key principle**: If data contains identifiers (names, IDs, emails) + anything sensitive (grades, health, financial), it's RESTRICTED. When in doubt, encrypt.

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Risk Assessment and Scenario Analysis

You're starting a new research project. For each scenario, evaluate:
- Is this data FERPA, HIPAA, EAR/ITAR, or other restricted category?
- What encryption approach makes sense?
- What are the consequences if data is exposed?

**Your scenario**: You're conducting a psychology study on stress and academic performance in first-year students at Pomona. You collect:
- Pre-screening survey (qualtrics form asking about anxiety, depression symptoms)
- Weekly check-in surveys (psychological stress scale scores)
- Access to anonymized GPA data (linked to survey via numeric ID only, no name)
- Audio recordings of optional interviews

**Questions**:
1. Which data elements are FERPA-protected?
2. Which are regulated under HIPAA (if any)?
3. Should all of this be encrypted together, or can you separate it?
4. What happens if recordings are exposed? If GPA data is exposed?
5. Who should have access to the encryption passphrase?

:::::::::::::::::::::::::::::::::::: solution

**Analysis**:

1. **FERPA-protected elements**: GPA data and any link to student ID (even numeric ID linking to Pomona records). Student IDs are identifiers under FERPA.

2. **HIPAA-regulated**: Pre-screening (psychiatric symptoms) and weekly check-ins (stress/depression measures)—these constitute health information. If collected in context of health research, HIPAA may apply even for research-only (depends on IRB determination).

3. **Separation strategy**: Yes, you should separate:
   - **Encrypted (RESTRICTED)**: GPA data + survey link (FERPA)
   - **Encrypted (RESTRICTED)**: Psychological symptoms + stress measures (HIPAA)
   - **Encrypted (RESTRICTED)**: Audio recordings (linked to identifiable individuals)
   - **Unencrypted (PROPRIETARY)**: Aggregate results and anonymized findings (no identifiers)

4. **Exposure consequences**:
   - GPA exposed: FERPA violation; affected students; university fined; your research suspended
   - Audio recordings exposed: Identity revealed; privacy violation; potential lawsuits from participants; research retraction
   - Combined data exposed: Full identity + health history + academic performance; severe harm to participants

5. **Passphrase access**: Only you and your primary advisor (for emergency access); not shared with graduate students or lab members. Others access data via you (exports) or via separate, monitored accounts.

**Conclusion**: This project requires comprehensive encryption. Separate encrypted directories for identifiable data; only share aggregate anonymized results with collaborators.

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Real-world scenarios: IACUC, FERPA, EAR, and NDA data all require encryption
- You classify and manage data; ITS provides tools; Compliance enforces policy
- Breach response: Stop, preserve evidence, contact ITS, inform advisor
- Data classification rule: Identifiers + sensitivity = RESTRICTED = encrypt
- Passphrase access: Only you (and advisor for emergency backup)
- Report suspected breaches immediately; early reporting is safer than delayed discovery
::::::::::::::::::::::::::::::::::::::::::::::::
