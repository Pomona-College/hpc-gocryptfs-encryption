---
title: "Why Encrypt? Regulations and Risk"
teaching: 15
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions
- When is data encryption necessary?
- What are the regulatory requirements?
- What happens when research data is exposed?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand regulatory drivers for encryption (FERPA, HIPAA, EAR, NIST)
- Identify the cascading consequences of a data breach
- Know the scale of the problem in higher education
- Learn the regulatory landscape that requires encryption
::::::::::::::::::::::::::::::::::::::::::::::::

## A Real Breach: The Cautionary Tale

Three years ago, a researcher at a peer institution was analyzing student health data for a psychological study approved by IRB. The data included student ID numbers, health conditions, and counseling notes—classic FERPA and HIPAA-protected information. The researcher stored it in an unencrypted directory shared with a graduate student.

During a storage system maintenance window, a misconfiguration briefly exposed the shared directory to the entire campus network. No one noticed immediately. By the time discovered 36 hours later, the directory had been indexed by campus file search tools. The exposure affected 847 students' health records.

The consequences cascaded rapidly:
- **Day 1**: Campus breach notification letter to 847 students
- **Day 3**: FERPA compliance investigation initiated
- **Week 1**: Research suspended; all data access revoked
- **Month 1**: University legal team involved; potential lawsuit from affected students
- **Month 2**: Federal HIPAA investigation launched; daily fines began accumulating
- **Month 3**: Researcher's funding revoked by agency; publication retracted
- **Month 6**: Settlement reached; university paid $2.4M in damages and remediation
- **Ongoing**: Researcher's reputation damaged; unfunded for future grants; removed from advisory committees

This wasn't malicious hacking. It was a preventable storage misconfiguration—precisely what encryption stops.

::::::::::::::::::::::::::::::::::::: callout
## The Domino Effect of Unencrypted Data

A single breach can trigger cascading consequences:

**Immediate (Days)**:
- Breach notification to affected individuals
- Regulatory agency notification
- Media coverage
- Social media backlash

**Short-term (Weeks)**:
- Compliance investigations by multiple agencies
- Suspension of related research
- Funding agency inquiries
- Legal team involvement

**Medium-term (Months)**:
- Potential fines (FERPA: $60,000 per violation; HIPAA: $100–$50,000 per record)
- Settlement negotiations
- Remediation costs (credit monitoring, notifications, legal fees)
- Publication retraction or correction

**Long-term (Years)**:
- Reputational damage
- Lost grant funding
- Career impact
- Difficulty recruiting collaborators
- Institutional reputation damage

Encryption prevents this entire cascade. One encrypted directory avoids millions in potential costs.
::::::::::::::::::::::::::::::::::::::::::::::::

## The Scale of the Problem

Higher education is a persistent target for data breaches:

- **500+ million student records exposed** in breaches at U.S. educational institutions over the past 5 years
- **Average cost per record**: $150–$300 (notification, remediation, legal)
- **FERPA violations at 40+ U.S. colleges** in the past 3 years alone
- **Average HIPAA fine**: $28,000 per violation
- **Time to detect breach**: 287 days average (months of exposure before discovery)

Encryption dramatically reduces the damage even if a breach occurs: encrypted data is inaccessible and unmarketable.

## The Regulatory Landscape

Multiple federal laws and regulations require encryption of research data:

### FERPA (Family Educational Rights and Privacy Act)

**What it covers**: Any information linked to a student that could identify them
- Student ID numbers
- Grades and transcripts
- Course enrollment
- Disciplinary records
- Psychological counseling notes
- Academic performance data

**Why it matters**: Applies to all student data, even if anonymized by name (student ID is identifiable)

**Violation penalties**: $60,000+ per violation; personal liability

**Encryption requirement**: "Encryption or other method of secure authentication" (NIST SP 800-171, Control 3.13.11)

**Pomona relevance**: Any study recruiting Pomona students or using institutional records

### HIPAA Security Rule (45 CFR § 164.312(a)(2))

**What it covers**: Protected health information (ePHI)
- Medical diagnoses and treatment
- Psychological or psychiatric records
- Medication information
- Lab results
- Any health-related data

**Why it matters**: Applies even to small research studies collecting health data

**Violation penalties**: $100–$50,000 per record per violation

**Encryption requirement**: "Encryption of electronic protected health information" (HIPAA mandates encryption of data at rest)

**Pomona relevance**: Biomedical research, psychology studies, health sciences research

### Export Administration Regulations (EAR) and International Traffic in Arms Regulations (ITAR)

**What it covers**: Research with potential military, security, or dual-use applications
- Cryptography research
- Materials science (advanced ceramics, composites)
- Semiconductor manufacturing
- Advanced manufacturing processes
- AI/ML in sensitive domains
- Some chemistry and physics research

**Why it matters**: "Deemed export" rule—sharing with foreign nationals (including postdocs on visa) is treated as exporting controlled technology

**Encryption requirement**: NIST SP 800-171 "Controlled Unclassified Information" (CUI) protection standard

**Pomona relevance**: Physics, chemistry, engineering, and CS research with international collaborators

### NIST SP 800-171 (Controlled Unclassified Information)

**What it covers**: Any federally funded research with restrictions
- CUI data (research sensitive for national security reasons)
- Data with export control implications
- Data with patent/IP protection value

**Why it matters**: Many federal grants (NSF, DARPA, DOD) require CUI compliance

**Key control (3.13.11)**: Encryption of information at rest using FIPS 140-2/3-validated algorithms (AES-256 meets this)

**Pomona relevance**: NSF-funded research, DOD grants, DOE grants

### Pomona ITS Policy 24: Data Classification and Protection

**What it establishes**: Institutional requirement to classify all research data and apply appropriate protections

**Key requirement**: "RESTRICTED data must employ encryption or equivalent controls"

**Who must comply**: All faculty, students, staff conducting research

**Enforcement**: Policy violation can result in account suspension and research termination

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Regulatory Match

For each research scenario below, identify which primary regulation applies: **FERPA**, **HIPAA**, **EAR**, or **NIST SP 800-171**.

1. A psychology professor is analyzing anonymized counseling session notes linked to student ID numbers for a study on academic stress.
2. A biology lab is collecting blood pressure and medication data from volunteer participants in a clinical nutrition study.
3. A physics researcher is collaborating with a visiting scholar from abroad on advanced semiconductor fabrication techniques with potential dual-use applications.
4. A computer science professor has received a DARPA grant to develop new AI algorithms, and the grant terms classify the research output as Controlled Unclassified Information (CUI).

::::::::::::::::::::::::::::::::::::: solution

## Solution

1. **FERPA** — The data is linked to student ID numbers, which are personally identifiable student records. Even though counseling notes may also touch HIPAA, the student ID linkage makes FERPA the primary regulation at an educational institution.
2. **HIPAA** — Blood pressure and medication data are Protected Health Information (ePHI). Any research collecting health-related data from participants falls under HIPAA Security Rule requirements.
3. **EAR** — Semiconductor fabrication techniques are export-controlled technology. Sharing with a foreign national (even on campus) triggers the "deemed export" rule under Export Administration Regulations.
4. **NIST SP 800-171** — DARPA grants that classify output as CUI require compliance with NIST SP 800-171 controls, including encryption of data at rest (Control 3.13.11) using FIPS 140-2/3-validated algorithms like AES-256.

::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Encryption is required for FERPA, HIPAA, EAR/ITAR, and NIST CUI-protected data
- A single breach cascades: notification → investigation → fines → reputation damage → career impact
- Higher education is a persistent breach target; 500+ million records exposed in 5 years
- FERPA violations cost $60,000+ per violation; HIPAA violations $100–$50,000 per record
- Pomona ITS Policy 24 mandates encryption for RESTRICTED data
- Encryption stops preventable breaches like storage misconfigurations
- Back up gocryptfs.conf separately; it cannot be recovered without the file
::::::::::::::::::::::::::::::::::::::::::::::::
