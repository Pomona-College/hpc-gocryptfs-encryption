# gocryptfs Encryption for Research Data

Pomona College HPC Workshop Series

## Overview

This workshop teaches researchers how to encrypt sensitive data using gocryptfs on the Sagehen HPC cluster. Participants learn to protect FERPA-protected student data, medical records, financial information, and export-controlled research with strong AES-256-GCM encryption. The workshop covers both the technical implementation and the compliance requirements that make encryption mandatory for certain data categories.

## Episodes

1. **Why Encrypt Your Research Data**: Understand when encryption is legally required, what gocryptfs protects and does not protect, and the tradeoffs between security and performance.
2. **gocryptfs Overview and Architecture**: Learn how gocryptfs works, the role of FUSE (Filesystem in Userspace), AES-256-GCM encryption, and the cipher/plain directory model.
3. **Creating Encrypted Directories**: Set up encrypted directories on Sagehen, initialize gocryptfs volumes, and manage encryption keys securely.
4. **Mounting and Unmounting**: Learn proper mounting and unmounting procedures, including how to handle sessions and prevent accidental data access.
5. **SLURM Integration**: Integrate encrypted directories into SLURM job scripts, mount encrypted data within batch jobs, and manage encryption in automated workflows.
6. **Key Management**: Manage encryption keys safely, use password best practices, create backups, and handle key recovery scenarios.
7. **Best Practices for Encrypted Research Data**: Follow guidelines for secure data handling, access controls, and integrating encryption into research workflows.

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
