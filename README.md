# gocryptfs Encryption for Research Data

Pomona College HPC Workshop Series

## Overview

This workshop teaches researchers how to encrypt sensitive data using gocryptfs on the Sagehen HPC cluster. Participants learn to protect FERPA-protected student data, medical records, financial information, and export-controlled research with strong AES-256-GCM encryption. The workshop covers both the technical implementation and the compliance requirements that make encryption mandatory for certain data categories.

## Episodes

**Section 1 — Why Encrypt? (1–3)**

1. **Regulatory Landscape**: When encryption is legally required (FERPA, HIPAA, EAR) and what's at stake
2. **Data Classification**: Pomona's three tiers — PUBLIC (755), PROPRIETARY (750), RESTRICTED (700 + gocryptfs)
3. **Scenarios and Responsibilities**: Real research scenarios and who is responsible for what

**Section 2 — How gocryptfs Works (4–5)**

4. **gocryptfs Architecture**: The two-directory model, FUSE, and why gocryptfs beats alternatives
5. **Encryption Internals**: AES-256-GCM, scrypt key derivation, and gocryptfs.conf

**Section 3 — Hands-On Encryption (6–9)**

6. **Planning Directories**: Naming, placement, and password choices before you initialize
7. **Creating Encrypted Directories**: `gocryptfs -init` and verifying your setup
8. **Mounting Workflow**: Daily mount/unmount habits
9. **Non-Interactive Mounting**: `--extpass`, `--passfile`, and troubleshooting

**Section 4 — SLURM Integration (10–12)**

10. **Decision Framework**: When jobs need encrypted data mounted (and when they don't)
11. **Script Templates**: Mount-compute-unmount patterns for batch jobs
12. **Advanced Patterns**: GPU jobs, job arrays, and robust error handling

**Section 5 — Key Management and Maintenance (13–17)**

13. **Key Backup**: gocryptfs.conf, the 3-2-1 rule, and backup locations
14. **Password Management**: Choosing, storing, and changing passphrases
15. **Lab Management**: Team access, offboarding, and disaster recovery
16. **Security Best Practices**: Permissions, hygiene, and troubleshooting
17. **Verification and Maintenance**: Routine checks and recovery drills

## Prerequisites

- Active Sagehen HPC cluster account
- Familiarity with Linux command line and file permissions
- Understanding of FERPA, medical data privacy, and data classification concepts
- Basic knowledge of SLURM job submission (helpful but not required)

## Learning Objectives

After completing this workshop, learners will be able to:
- Identify which data categories require encryption under institutional policy
- Create and manage encrypted directories using gocryptfs
- Mount and unmount encrypted volumes securely
- Integrate encryption into SLURM workflows and batch jobs
- Manage encryption keys using best practices
- Implement encryption as part of research data protection strategies

## Target Audience

Researchers who work with sensitive data including student records, medical information, financial data, or export-controlled research. This includes graduate students, postdocs, faculty, and research staff at Pomona College who need to protect restricted data on shared HPC systems.

## Duration

Approximately 2-3 hours, depending on hands-on practice with encrypted directory setup and SLURM integration.

## Technical Requirements

- Sagehen HPC cluster account with storage allocation
- SSH access to Sagehen login nodes
- Basic Linux command-line tools (mount, mkdir, chmod)
- Text editor for configuration files
- Optional: SLURM job submission scripts to integrate encryption into workflows

## Contact

- **Email**: its-hpc@pomona.edu
- **Workshop Author**: Andrew Wilson, Director of Research Computing

## License

This workshop is licensed under [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).

## Citation

Wilson, A. (2026). *gocryptfs Encryption for Research Data*. Pomona College ITS Research Computing.

## Acknowledgments

**Andrew Wilson** — Director of Research Computing and Digital Scholarship,
Pomona College. Workshop design and development.

**Andrei Motchenko** — testing, editing, cleanup and screenshots across the
Pomona College HPC Workshop Series.
