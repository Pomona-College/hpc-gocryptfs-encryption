---
site: sandpaper::sandpaper_site
---

This workshop teaches you how to use **gocryptfs** — a transparent, file-level encryption tool — to protect restricted research data on the **Sagehen HPC cluster**. You will learn to create encrypted directories, integrate encryption into SLURM workflows, manage encryption keys safely, and maintain compliance with federal and institutional data protection requirements.

::::::::::::::::::::::::::::::::::::: callout

## Pomona College HPC Context

This workshop is part of the **Pomona College Research Computing Workshop Series**
for researchers using the **Sagehen HPC cluster**.

**This workshop is MANDATORY** for all researchers storing or processing
Restricted-tier data on Sagehen. Pomona's [ITS Training and Awareness Policy][policy-24]
and [NIST SP 800-171][nist-800-171] control 3.13.11 require encryption for
Controlled Unclassified Information (CUI) at rest.

**Prerequisites:**

- [Workshop 13: HPC Security Orientation][ws-13] (required)
- [Workshop 14: Data Classification and Handling][ws-14] (strongly recommended)
- Active Sagehen HPC account ([its-hpc@pomona.edu](mailto:its-hpc@pomona.edu))
- Comfortable with the Unix command line (Workshops 1–2)

For questions or support, contact [its-hpc@pomona.edu](mailto:its-hpc@pomona.edu).

:::::::::::::::::::::::::::::::::::::

## Workshop Structure

The workshop is organized into 17 episodes across five sections:

**Section 1 — Why Encrypt? (Episodes 1–3)**
Regulatory landscape, Pomona's data classification system, and real-world scenarios.

**Section 2 — How gocryptfs Works (Episodes 4–5)**
Architecture, AES-256-GCM encryption, FUSE filesystem model, and key derivation.

**Section 3 — Hands-On Encryption (Episodes 6–9)**
Planning directories, creating encrypted volumes, mounting workflows, and non-interactive mounting.

**Section 4 — SLURM Integration (Episodes 10–12)**
Decision framework for pre-mount vs. mount-in-script, job script templates, and advanced patterns (arrays, pipelines, GPU jobs).

**Section 5 — Key Management and Maintenance (Episodes 13–17)**
Key backup, password management, lab group management, security best practices, and verification/maintenance routines.

**Total estimated time:** 4–5 hours (teaching + exercises)

## Key Facts

- **Encryption:** AES-256-GCM (hardware-accelerated on Sagehen)
- **Key derivation:** Argon2 (memory-hard, brute-force resistant)
- **Performance overhead:** 5–10% CPU, 10–20% I/O typical
- **Password requirement:** 14+ characters ([NIST SP 800-63B][nist-800-63b])
- **No password recovery:** Forgotten passwords mean permanently lost data
- **Critical file:** `gocryptfs.conf` must be backed up — losing it means losing all encrypted data
