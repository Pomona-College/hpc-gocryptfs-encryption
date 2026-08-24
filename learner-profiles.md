---
title: Learner Profiles
---

## Overview

The following learner profiles represent typical backgrounds, data contexts, and motivations for the gocryptfs Encryption for Research Data workshop at Pomona College's Sagehen HPC cluster. These personas reflect the diverse needs of researchers working with HIPAA, GDPR, export-controlled, and other sensitive data. Use these to tailor examples, scenarios, and the pace of discussion during the workshop.

---

## Profile 1: Dr. Sarah Chen - Biology Researcher with HIPAA-Regulated Data

**Name:** Dr. Sarah Chen  
**Role:** Associate Professor, Molecular Biology Lab  
**Background:** 12 years research experience, managing lab with 5 graduate students and 3 undergraduate researchers. Works with human subject data for genetic research.  
**Technical Level:** Medium (Linux command line familiar from HPC work; no file-level encryption experience)  
**Motivation for Workshop:** Lab is expanding human subjects research and needs to implement secure data handling on Sagehen.

### Current Challenges

- Lab is collecting genomic data linked to phenotype information on human participants
- Data includes participant IDs, genetic sequences, and health history
- Under HIPAA regulations and IRB protocol, this data must be encrypted and access-controlled
- Currently storing in shared lab directory with only basic file permissions (world-readable)
- Must support 5-8 concurrent lab members accessing different subsets of data
- No documented procedure for secure password sharing with lab members
- Needs backup and recovery procedures for critical datasets
- Lab lacks compliance audit trail for access

### Specific Needs from Workshop

- Understand regulatory requirements (HIPAA, IRB) for data protection on Sagehen
- Set up gocryptfs for genomic dataset (~50 GB) on shared /bigdata storage
- Create secure mount workflow that lab members can follow without deep technical knowledge
- Implement tiered access: lead researchers, analysts, student assistants have different permissions
- Document procedures for lab handbook (training for new lab members)
- Verify data remains encrypted when unmounted (evidence for compliance)
- Learn recovery procedures if gocryptfs password is lost
- Establish process for revoking access when students graduate or leave

### Expected Outcomes

- Can classify lab genomic data as RESTRICTED tier (encrypted)
- Has designed gocryptfs vault structure for shared lab use on /bigdata
- Can create lab-specific mount procedures and scripts
- Can set up password rotation procedure for lab members
- Has documented compliance plan to show IRB and auditors
- Can train lab members on secure data handling
- Understands how gocryptfs protects HIPAA requirements
- Has checklist for onboarding/offboarding lab members

### Teaching Adjustments

- Use "genomic data with participant IDs" as running example throughout
- Emphasize HIPAA and IRB requirements (not just technical security)
- Demonstrate practical setup: Create encrypted vault on /bigdata, mount on /rhome, organize participant data inside
- Discuss lab governance: How to give new team members access, revoke for departing students
- Show scenario: "A grad student wants to take genomic data home on laptop. How do you respond?"
- Provide HIPAA checklist and lab data security policy template
- Discuss password management: How to share vault password securely with lab (use password manager? NDA with key phrase?)
- Include exercise: Practice mounting/unmounting encrypted vault
- Provide IRB contact and resources for documentation

---

## Profile 2: Dr. Marcus Johnson - Postdoc with Export-Controlled Proprietary Data

**Name:** Dr. Marcus Johnson  
**Role:** Postdoctoral Researcher, Materials Science Group  
**Background:** 4 years postdoc experience, specializing in novel synthetic materials. Previous position at university; current work involves international collaborations.  
**Technical Level:** Medium-High (familiar with GPG encryption for email; has used encrypted thumb drives; comfortable with command-line)  
**Motivation for Workshop:** Developing proprietary catalytic compounds with potential commercial applications and export control implications.

### Current Challenges

- Developing new catalytic compounds with potential commercial applications
- Research includes proprietary synthesis procedures, computational models, and preliminary performance data
- Export Control Regulations (EAR) apply because technology could be controlled for national security
- Must protect from both external theft and accidental disclosure to international collaborators
- Works with faculty advisor who has licensing/export control expertise but not always available
- Data spans multiple project phases with different sensitivity levels
- Needs ability to revoke access to old collaborators quickly
- Must track who accesses what data and when (for export control compliance)
- Potential for commercialization; uncertain about IP ownership vs. data security

### Specific Needs from Workshop

- Encrypt proprietary synthesis data and computational models on Sagehen
- Create separate encrypted vaults for different project stages (early/confidential vs. publishable)
- Understand which data tiers apply (PROPRIETARY with potential export control restrictions)
- Establish workflow for sharing encrypted directories with select collaborators
- Learn gocryptfs password recovery in case key is lost (critical for research continuity)
- Implement backup strategy for research-critical data across project phases
- Document encryption approach for export control compliance documentation
- Understand access logs and audit trails for regulatory purposes
- Learn how gocryptfs compares to other encryption (GPG, encrypted USB)

### Expected Outcomes

- Can classify synthesis data as PROPRIETARY (encrypted on Sagehen)
- Has designed gocryptfs vault structure with separate project phase folders
- Understands which collaborators get access to which phases
- Can document encryption setup for export control report
- Has backup and recovery strategy for /rhome and /bigdata storage
- Knows how to revoke collaborator access when project phase ends
- Can explain to advisors why encryption is necessary for EAR compliance
- Understands password management for multi-user proprietary data

### Teaching Adjustments

- Use "export-controlled proprietary chemistry" as running example
- Emphasize legal/regulatory aspects (EAR, IP) alongside technical security
- Show practical: Create PROPRIETARY vault on /bigdata, organize by project phase, set permissions
- Discuss access control: "How do you give a collaborator access to one phase only?"
- Include scenario: "Postdoc moves to industry job. How do you ensure she doesn't take proprietary data?"
- Provide export control compliance checklist and documentation template
- Discuss password management: separate passwords for different project phases?
- Connect to patent/licensing: "Encryption isn't just for security; it's also for IP protection"
- Provide faculty advisor contact info for export control questions
- Include advanced topic: Audit logging and access tracking for compliance

---

## Profile 3: Professor Jennifer Williams - Faculty Managing Survey Research with PII

**Name:** Professor Jennifer Williams  
**Role:** Associate Professor, Economics Department; Director of Survey Research Lab  
**Background:** 10 years faculty experience, runs survey research involving sensitive economic data. Lab has grown from 2 to 8 staff members.  
**Technical Level:** Low-Medium (uses password managers personally; limited Linux experience; tends to work through graphical interfaces; willing to learn structured procedures)  
**Motivation for Workshop:** Lab is expanding and needs to implement data governance while maintaining research team productivity.

### Current Challenges

- Survey responses contain PII (personally identifiable information): names, addresses, SSNs, bank account information, income details
- Currently storing data in shared folder on lab computer with inconsistent access controls
- Lab has grown from 2 to 8 staff members but no formal data security policy
- Staff includes both graduate students and undergraduate interns with varying technical skills
- Data retention schedules vary by funding source (some must be deleted after study, others archival)
- Must comply with institutional IRB and federal data protection regulations
- Must support onboarding/offboarding workflows as staff changes frequently
- No clear documentation for staff on how to handle sensitive economic data
- Worried about liability if data is breached

### Specific Needs from Workshop

- Encrypt survey datasets containing PII on Sagehen /rhome and /bigdata storage
- Create mount/unmount procedures that lab staff can follow without deep technical knowledge
- Understand best practices for handling sensitive economic data (RESTRICTED tier)
- Set up tiered access: lead researcher, analysts, student assistants have different permissions
- Develop disaster recovery plan for encrypted data
- Create clear documentation for non-technical staff members
- Establish data retention and deletion procedures per funding source
- Learn how to revoke access when interns graduate or staff leave
- Understand compliance requirements (what documentation to keep for auditors)

### Expected Outcomes

- Can classify survey data as RESTRICTED (encrypted with gocryptfs)
- Has designed secure folder structure for survey datasets on /bigdata
- Can create simple mount/unmount procedures for lab staff
- Has documented data security policy for lab handbook
- Understands IRB and federal compliance requirements for survey data
- Can explain to staff why data must be encrypted (not just "security best practices")
- Can train new staff on data handling procedures
- Has checklist for onboarding/offboarding with data access management
- Can provide compliance documentation to auditors and IRB
- Understands password management and sharing with lab staff

### Teaching Adjustments

- Start with "survey research with PII" as running example
- Emphasize compliance (IRB, federal regulations) over technical details
- Use simple language; avoid Linux jargon when possible
- Demonstrate practical setup: Create encrypted vault on /bigdata, mount on /rhome, organize datasets inside
- Show step-by-step: How do you give a new intern access? How do you revoke access?
- Include scenario: "A grad student asks to email survey data to a collaborator. What do you do?"
- Provide IRB checklist and lab data security policy template
- Show visual diagram of lab folder structure
- Discuss password management: How to share vault password securely with staff (password manager + NDA?)
- Include hands-on exercise: Mount and unmount encrypted vault
- Provide resources for compliance documentation
- Suggest follow-up: Help setting up encryption with real lab data

---

## Profile 4: Alex Kumar - Graduate Student with Thesis Research Data

**Name:** Alex Kumar  
**Role:** Graduate Student, Machine Learning thesis track  
**Background:** 2 years into Master's program, thesis on privacy-preserving machine learning. Strong programming skills but limited systems administration experience.  
**Technical Level:** High (comfortable with command line and Python scripting; studied cryptography in coursework; understands encryption theory but no practical experience)  
**Motivation for Workshop:** Thesis involves building ML models on sensitive healthcare datasets; learning to work with real sensitive data for first time.

### Current Challenges

- Thesis data includes de-identified patient records, diagnostic codes, and treatment outcomes from research hospital
- Data includes HIPAA-regulated information even though de-identified
- First time working with sensitive data in practice; has studied theory but not implementation
- Learning to balance encryption security with analysis workflow (need to mount for computation)
- Must document encryption approach for thesis methodology section
- Plans to publish research; must understand how encryption affects reproducibility
- Works independently; needs self-sufficient troubleshooting skills
- High learning curve tolerance; enjoys technical depth
- Wants to understand not just gocryptfs but also broader data security architecture
- May need to explain encryption approach to thesis committee

### Specific Needs from Workshop

- Understand practical encryption implementation beyond cryptography theory
- Set up gocryptfs for thesis dataset (~10 GB) on Sagehen storage
- Create automated mount/unmount scripts for analysis workflows (Python + bash)
- Learn recovery procedures: what happens if password is lost? (critical for thesis data)
- Understand how gocryptfs integrates with analysis tools (Python notebooks, ML frameworks)
- Document encryption approach for thesis methodology section
- Learn how to share encrypted data with thesis advisor or collaborators
- Understand data classification: RESTRICTED vs. PROPRIETARY for thesis data
- Explore encryption options and trade-offs (gocryptfs vs. encrypted USB vs. GPG)
- Understand backup and version control with encrypted data

### Expected Outcomes

- Can classify thesis data as RESTRICTED (de-identified but HIPAA-regulated)
- Has set up gocryptfs vault for thesis dataset on /bigdata
- Has created automated mount/unmount scripts for analysis workflows
- Understands encryption for thesis write-up and defense
- Can explain encryption approach to thesis committee and in methodology section
- Knows recovery procedures and what to do if password is lost
- Can share encrypted dataset with advisor or collaborators safely
- Understands how encryption affects data reproducibility and archiving
- Has strong troubleshooting skills for encryption issues
- Can reason about encryption design trade-offs

### Teaching Adjustments

- Don't belabor basics; fast-track through conceptual material (Alex studied cryptography)
- Focus on practical implementation: gocryptfs internals, system integration, automation
- Use "healthcare data for ML thesis" as running example
- Demonstrate: Create encrypted vault, mount programmatically, write automation scripts
- Discuss integration: "How does encryption work with Jupyter notebooks? TensorFlow? PyTorch?"
- Include scenario: "Advisor wants to review your data. How do you securely share it?"
- Provide advanced topics: gocryptfs architecture, performance tuning, security trade-offs
- Discuss reproducibility: "How do you document encryption approach in thesis?"
- Provide resources: gocryptfs source code, cryptography papers, security best practices
- Suggest hands-on exercise: Write automated mount script for thesis workflow
- Discuss backup strategy for thesis data (version control + encryption)
- Provide contact: Research & Sponsored Programs for questions about data ownership

---

## Profile 5: Dr. Robert Okafor - Political Science Faculty with Export-Controlled Data and GDPR Implications

**Name:** Dr. Robert Okafor  
**Role:** Visiting Scholar from EU Institution; Conducting International Election Survey Research  
**Background:** Political science faculty on 1-year visiting appointment from European university. Brings survey data from international election study with EU participants.  
**Technical Level:** Medium (familiar with Linux from prior work; has used basic encryption; less familiar with US regulatory systems)  
**Motivation for Workshop:** Survey data includes responses from EU participants under GDPR; also involves sensitive political/election information (potentially export-controlled analysis); must ensure compliance with both EU and US regulations.

### Current Challenges

- Survey data collected in EU from 200+ election study participants (sensitive political information)
- Data is under GDPR (EU regulation), different from US HIPAA
- Some analysis topics could touch export control issues (international politics, sensitive election data)
- Needs to transfer data from Europe to Sagehen; worried about legal compliance
- Doesn't fully understand US IRB process or HIPAA (different from EU ethics committees)
- Data is currently encrypted on home institution's server; needs similar setup on Sagehen
- Concerned about data leaving EU (GDPR restrictions on data transfer)
- Will be leaving Pomona in 1 year; needs clear plan for data handling at end of visit
- Collaborating with US lab; unclear who owns data, who can publish what
- Questions about whether analysis is export-controlled (sensitive political data)

### Specific Needs from Workshop

- Understand GDPR vs. HIPAA: which rules apply to her survey data in the US?
- Learn how to classify election survey data (likely PROPRIETARY or RESTRICTED)
- Set up gocryptfs for election data on Sagehen to match EU protection level
- Understand IRB approval requirements for human subjects research in US (different from EU)
- Learn legal process for transferring data from EU to US
- Implement encryption approach that complies with GDPR even on US servers
- Understand data ownership: who owns the international survey data? Publication rights?
- Plan for data handling after visit ends (transfer back to Europe? Archive at Pomona? Delete?)
- Understand export control for sensitive political analysis
- Document compliance with both GDPR and US regulations

### Expected Outcomes

- Understands how GDPR and HIPAA/US regulations apply to election survey data
- Has engaged with Pomona's IRB and Research & Sponsored Programs for international compliance
- Can transfer data securely (encrypted) following legal requirements
- Has set up encrypted storage on Sagehen at GDPR-compliant level (likely RESTRICTED)
- Understands data ownership and publication restrictions in US collaboration
- Has clear, documented plan for transferring data back to Europe or archiving at Pomona
- Can navigate US data protection requirements and international data transfer regulations
- Has documented approach for both GDPR and US compliance
- Can explain encryption approach to EU and US institutions

### Teaching Adjustments

- Acknowledge GDPR/EU experience; explain how US system is different (regulatory, not legal framework)
- Clarify: HIPAA is health-specific; Election data is social science; may fall under broader FERPA/IRB
- Emphasize: IRB is similar to European ethics committees but different process and scope
- Discuss data transfer: GDPR restrictions, legal agreements, encryption in transit
- Show: gocryptfs works internationally; same encryption tools, different regulations
- Use "international election survey with GDPR constraints" as running example
- Include scenario: "You want to share data with US collaborator. What steps? What documents?"
- Provide contacts: IRB, Research & Sponsored Programs, legal office for international questions
- Discuss end-of-visit planning: "Can you take data home? What's the GDPR process? Archive timeline?"
- Provide resources: GDPR vs. HIPAA comparison, international data transfer checklist
- Acknowledge: "This is complex. It's okay to involve legal and IRB."
- Suggest follow-up: Check-in with IRB and legal before transferring data across borders

---

## Profile 6: Priya Patel - Undergraduate Research Assistant, First Exposure to Data Security

**Name:** Priya Patel  
**Role:** Undergraduate Research Assistant, Neuroscience Lab  
**Background:** Junior biology student, first research experience in neuroscience lab. Working with EEG data from study participants.  
**Technical Level:** Low (knows computers; no Linux experience; no research or data security background)  
**Motivation for Workshop:** Lab is implementing new data security procedures; Priya was asked to attend and help with data organization.

### Current Challenges

- First research experience; doesn't understand what "data classification" or "encryption" means
- Lab advisor mentioned data security but Priya isn't sure what her role should be
- Worried about accidentally mishandling sensitive participant data
- Doesn't understand difference between data that can be shared vs. kept private
- Has only basic computer skills (some web use, no Linux or command line)
- Nervous about asking "dumb questions" about data security
- Sees older grad students handling data; doesn't understand why some steps are necessary
- Worried she might do something wrong and cause a compliance problem
- Doesn't know when to ask for help vs. when to figure it out herself

### Specific Needs from Workshop

- Very basic introduction to data classification (what it is, why it matters)
- Understanding her specific role and responsibility as lab assistant
- How to recognize sensitive data (participant IDs, EEG data with identifiers)
- Practical guidance: "What do I do with data files in my lab?"
- Clear, simple rules for her lab (from PI or senior grad students)
- Understanding encryption without technical jargon (analogies, simple language)
- Knowing when to ask for help (better to ask than guess)
- Confidence that she's doing the right thing

### Expected Outcomes

- Understands data tiers in simple terms (PUBLIC can be shared widely, PROPRIETARY restricted to collaborators, RESTRICTED needs encryption)
- Knows what types of data her lab works with and their classification
- Can follow her lab's data handling procedures correctly
- Knows what NOT to do with sensitive data (email it, copy to personal laptop, leave on public folder, etc.)
- Can recognize when something seems wrong and ask for help
- Feels confident in her role without feeling overwhelmed
- Knows who to ask when confused (lab PI, senior grad student, HPC support)

### Teaching Adjustments

- Start very basic: use analogies ("Data tiers like locks on doors: PUBLIC = everyone, PROPRIETARY = team only, RESTRICTED = encrypted vault")
- Avoid all technical jargon; use plain language consistently
- Pair with more experienced lab member during workshop for context and support
- Use "neuroscience lab with participant EEG data" as example
- Provide very simple, visual decision tree: "If you have data, ask: Are participant IDs in it? Then it's RESTRICTED."
- Provide printed checklist: "Where does your lab's data go? Who can see it? When do you ask PI?"
- Emphasize frequently: "It's okay to ask questions. Better to ask than to guess and make a mistake."
- Share simple, real scenario: "You're organizing data files. You see a folder called 'raw_EEG_with_IDs'. What do you do?"
- Make clear: "Your PI is responsible for overall compliance. You're learning and helping."
- Provide "Who to Ask" list with names and emails
- Include exercise: "Here are 5 data files. Which ones are RESTRICTED? Which ones are PUBLIC?"
- Reassure: Compliance mistakes happen; learning how to prevent them is the point

---

## Profile 7: Dr. James Liu - Physics Faculty with DOD-Funded NIST SP 800-171 Compliance

**Name:** Dr. James Liu  
**Role:** Associate Professor, Physics Department  
**Background:** Physics research with DOD funding. Works with Controlled Unclassified Information (CUI). Strong systems-oriented thinking; comfortable with complex technical requirements.  
**Technical Level:** High (very comfortable with Linux, systems thinking, technical specifications; has worked with NIST standards before)  
**Motivation for Workshop:** DOD-funded research requires NIST SP 800-171 compliance for CUI data; needs to set up secure encryption approach on Sagehen that meets federal standards.

### Current Challenges

- DOD-funded research contract requires protection of Controlled Unclassified Information (CUI)
- Must comply with NIST SP 800-171 (federal security standard for CUI)
- Specific requirement: 14+ character passwords (NIST SP 800-63B), encryption at rest and in transit
- Research data on Sagehen /rhome and /bigdata shares 1TB lab quota (BeeGFS)
- Must maintain compliance documentation for DOD auditors
- Team includes grad students and postdocs; must ensure all follow security procedures
- Concerned about whether gocryptfs alone meets NIST requirements or if additional controls needed
- Needs to understand encryption architecture: Does gocryptfs meet CUI protection requirements?
- Must document security controls for DOD compliance report
- Concerned about password management at scale (team size growing)

### Specific Needs from Workshop

- Understand gocryptfs from a NIST SP 800-171 compliance perspective
- Set up encrypted storage for CUI data that demonstrates federal compliance
- Implement 14+ character password requirements (NIST SP 800-63B standard)
- Document security controls for DOD audit purposes
- Understand encryption at rest (gocryptfs) and encryption in transit (SFTP on Sagehen)
- Set up access controls and audit logging for CUI data
- Learn backup and recovery procedures compliant with NIST standards
- Implement team-wide password management process (secure sharing, rotation, recovery)
- Understand what additional controls may be needed beyond gocryptfs
- Document compliance approach to share with DOD program officer

### Expected Outcomes

- Can architect encryption solution for CUI data on Sagehen that meets NIST SP 800-171
- Understands gocryptfs capabilities and limitations for federal compliance
- Has implemented 14+ character passwords per NIST SP 800-63B
- Has designed CUI folder structure with appropriate access controls and audit logging
- Can document security approach for DOD compliance report
- Can train team on NIST-compliant password management and procedures
- Understands encryption at rest (gocryptfs) and in transit (secure communication) requirements
- Has planned backup/recovery process compliant with NIST standards
- Can explain NIST compliance approach to DOD auditors
- Knows when to escalate questions to HPC support or legal/compliance office

### Teaching Adjustments

- Fast-track through basics; focus on NIST compliance architecture and requirements
- Use "DOD CUI research with NIST SP 800-171 requirements" as running example
- Provide depth on federal standards and compliance documentation
- Show practical: Set up CUI vault, implement 14+ char passwords, document access controls
- Discuss: Does gocryptfs alone meet NIST, or are additional controls needed?
- Discuss encryption in transit: SFTP, SSH key management, secure communication with collaborators
- Include scenario: "DOD auditor asks about your encryption approach. How do you explain?"
- Provide resources: NIST SP 800-171 standard, CUI compliance checklist, audit documentation template
- Discuss advanced topics: access logging, audit trails, compliance monitoring
- Provide government contacts: DOD program officer, Pomona Research & Sponsored Programs for compliance questions
- Suggest: Work with HPC support and Pomona's compliance office to verify setup
- Include advanced exercise: Review gocryptfs architecture against NIST requirements, identify gaps

---

## Using These Profiles

### During Workshop Planning

- Anticipate which profiles will attend by asking at registration: "What's your role? What type of data do you work with? What are your compliance requirements?"
- Prepare examples matching the group's interests (HIPAA for biomedics, GDPR for international researchers, NIST for federal funded research, etc.)
- Plan breakout discussions for different contexts (human subjects research, proprietary/export-controlled, international compliance, federal CUI)
- Identify which advanced topics to emphasize (password management, audit logging, export control, international data transfer)

### During Teaching

- Reference profiles: "If you're like Sarah managing genomic data, here's how to set up lab-wide encryption..."
- Adjust pace: Speed up for Alex and James (high tech skills), slow down for Priya
- Use relevant examples: Genomic data for Sarah, proprietary chemistry for Marcus, surveys for Jennifer, election data for Robert, thesis for Alex, participant EEG for Priya, CUI for James
- Create peer learning: Pair Priya with more experienced learner; pair Marcus/Robert with James for discussion of compliance issues

### For Differentiated Exercises

- **Beginner (Priya):** Recognize sensitive data, follow mount/unmount procedures, identify when to ask for help
- **Intermediate (Sarah, Marcus, Jennifer, Robert):** Design gocryptfs setup for their own data, plan access control, draft security documentation
- **Advanced (Alex, James):** Automate encryption workflows, integrate with analysis/development tools, design compliance architecture, write audit procedures

---

## Demographic Mix and Pacing

If your workshop has a mix of these profiles:

- **Mostly faculty/senior researchers (Sarah, Marcus, Jennifer, Robert, James):** Spend 15% on basics, 35% on compliance/architecture, 50% on implementation and problem-solving
- **Mostly students/postdocs (Alex, Priya):** Spend 40% on concepts and examples, 40% on hands-on setup, 20% on advanced topics
- **Mixed group with international researchers (Robert):** Include segment on international data transfer, GDPR, and multi-jurisdiction compliance
- **Group with federal funding (James, potentially Marcus for export control):** Offer optional "advanced compliance" session on NIST/export control standards
- **Large group with diverse compliance needs:** Use breakout exercises by compliance type (HIPAA, GDPR, export control, NIST); invite compliance office (IRB, legal, Research & Sponsored Programs) for Q&A

### When Discussing Data Tiers

Use the three-tier framework consistently for Pomona/Sagehen storage:
- **PUBLIC (755):** Data shared openly; no encryption needed (on /rhome or /bigdata)
- **PROPRIETARY (750):** Data shared within team; encryption recommended; includes export-controlled data, proprietary research, draft publications
- **RESTRICTED (700+gocryptfs):** Sensitive human subjects data, PII, health information, participant data; encryption required on Sagehen

---

## Accommodations and Support Needed

- **For Priya (novice with low tech skills):** Provide very simple, visual decision trees. Pair with experienced learner. Reassure frequently. Written checklist ("What to do with data files"). Emphasize: asking for help is good.
- **For Robert (international perspective):** Acknowledge GDPR background and EU regulatory experience. Provide comparison chart (GDPR vs. HIPAA vs. US IRB). Discuss international data transfer and legal considerations. Provide contacts (IRB, legal office) for complex questions.
- **For Alex (high technical skills):** Don't belabor basics. Offer advanced topics: automation, scripting, performance, cryptography depth. Hands-on exercise with real thesis workflow.
- **For Marcus (export control complexity):** Provide export control compliance checklist. Discuss IP/commercialization alongside security. Provide faculty advisor contact. Offer scenario on access revocation and collaborator management.
- **For Jennifer (managing non-technical team):** Provide templates and checklists she can print and share. Emphasize simple procedures. Offer optional follow-up session for her lab staff (onboarding/offboarding). Suggest she bring lab manager or senior staff.
- **For Sarah (multi-user lab complexity):** Provide lab data policy template. Discuss access control at scale. Offer lab-specific follow-up session. Suggest she work with HPC support to implement encryption.
- **For James (federal compliance):** Provide NIST checklists and compliance templates. Discuss architecture. Offer review of setup against federal standards. Suggest ongoing collaboration with HPC and compliance office.

---

## Follow-Up and Support by Profile

### Dr. Sarah Chen (HIPAA Multi-User Lab)

- **Provide:** HIPAA checklist, lab data security policy template, access control guide, IRB contact, sample onboarding/offboarding procedure
- **Follow-up:** "Help me set up encrypted vault for genomic data and train my lab members"
- **Suggest:** Monthly lab meetings on data security, quarterly access audits, annual compliance review with IRB
- **Contact:** its-hpc@pomona.edu for encryption setup help; IRB for policy questions

### Dr. Marcus Johnson (Export-Controlled Proprietary Data)

- **Provide:** Export control compliance checklist, NDA template, access revocation procedure, IP/commercialization guide
- **Follow-up:** "Help me design separate vaults for different project phases and set up access revocation"
- **Suggest:** Quarterly access reviews, export control documentation updates, commercialization planning
- **Contact:** its-hpc@pomona.edu for encryption; Faculty advisor or Research & Sponsored Programs for export control

### Professor Jennifer Williams (Survey Research with Team)

- **Provide:** Lab data security policy template, sample mount/unmount procedures (documented for non-tech staff), IRB compliance checklist, onboarding/offboarding script
- **Follow-up:** "Help me implement encryption with my team; train staff on procedures"
- **Suggest:** One training session for full lab staff; monthly data security updates; annual IRB compliance review
- **Contact:** its-hpc@pomona.edu for technical help; IRB for policy questions

### Alex Kumar (Thesis with Automation)

- **Provide:** Automation script templates, reproducibility guide, backup/archiving procedures, thesis documentation examples
- **Follow-up:** "Help me automate mount/unmount in my thesis workflow and document encryption approach"
- **Suggest:** Monthly office hours for troubleshooting; explore automated backup; work with advisor on reproducibility plan
- **Contact:** its-hpc@pomona.edu for technical issues; Research & Sponsored Programs for data ownership questions

### Dr. Robert Okafor (International/GDPR Compliance)

- **Provide:** GDPR vs. HIPAA comparison chart, international data transfer guide, legal contacts, IRB process explanation
- **Follow-up:** "Help me ensure my data transfer complies with both GDPR and US rules"
- **Suggest:** Check-in with IRB before data transfer; legal review of collaboration agreements; plan for end-of-visit data handling
- **Contact:** its-hpc@pomona.edu for encryption; IRB for international compliance; Legal office for data transfer; Research & Sponsored Programs

### Priya Patel (Undergraduate Assistant)

- **Provide:** Simple data classification checklist ("Is there a participant ID? Then it's RESTRICTED"), lab-specific rules from PI, "Who to Ask" list, visual decision tree
- **Follow-up:** One-on-one guidance as she takes on more data responsibility; brief check-in after first month
- **Suggest:** Monthly lab meetings where procedures are reinforced; optional follow-up workshop as she progresses
- **Contact:** Her lab PI; senior grad student mentor; its-hpc@pomona.edu for technical questions (with PI's help)

### Dr. James Liu (NIST SP 800-171 Compliance)

- **Provide:** NIST SP 800-171 compliance checklist, CUI handling guide, password management procedure (14+ chars), audit logging template, DOD documentation examples
- **Follow-up:** "Help me verify my encryption setup meets NIST requirements and set up compliance documentation"
- **Suggest:** Review with HPC support and Pomona's compliance office; quarterly compliance audits; DOD audit preparation
- **Contact:** its-hpc@pomona.edu for encryption and Sagehen setup; Research & Sponsored Programs for DOD compliance; Pomona Legal/Compliance for federal standards

---

## Building Community and Continuity

After the workshop:

- **Create online community** (forum, Slack, email list) for ongoing questions and resource sharing across all profiles
- **Offer monthly office hours** for hands-on help: encryption setup, access control, password management, compliance documentation
- **Share case study library** of real (anonymized) data security scenarios from Pomona research: "Lab with growing team," "International collaboration," "DOD-funded project," etc.
- **Create role-specific resource guides:**
  - Faculty guide (Sarah, Marcus, Jennifer, Robert, James)
  - Graduate student guide (Alex)
  - Undergraduate/assistant guide (Priya)
  - Lab manager guide (for Sarahs and Jennifers with large teams)
  - International researcher guide (Robert and visiting scholars)
- **Host quarterly refresher workshops** on focused topics (Password Management, Access Control, Compliance Documentation, Automation with Encryption)
- **Build referral network:** Connect Sarah (multi-user lab) with Jennifer (team management) for peer learning; connect Marcus (export control) and James (federal compliance) for advanced compliance discussion; pair Priya with experienced lab member mentor
- **Maintain resource library on Sagehen HPC system:**
  - Password requirements (NIST SP 800-63B: 14+ characters)
  - Data tier definitions (PUBLIC 755, PROPRIETARY 750, RESTRICTED 700+gocryptfs)
  - Storage information (/rhome and /bigdata on BeeGFS, 1TB lab quota)
  - Sample scripts and templates
  - Contact list: its-hpc@pomona.edu, IRB, Research & Sponsored Programs, Legal
