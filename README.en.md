🇫🇷 [Lire ce README en français](README.md)

# Microsoft Purview Configuration (2026)
## Compliance & AI Architecture Guide for SMBs

## 📥 Download

👉 [Download the Word/PDF version (v1.0)](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0)

This repository contains a **step-by-step technical procedure** to configure **Microsoft Purview** for **Microsoft 365 Business Premium** environments, with a strong focus on:

- data protection,
- Microsoft 365 Copilot governance,
- and compliance with **Swiss data protection law (nLPD)**.

> **Key message**  
> Securing Copilot and sensitive data is **not a licensing problem**.  
> It is an **architecture and configuration problem**.

---

## 🎯 Scope of this guide

✅ **Included**
- Microsoft 365 Business Premium  
- Microsoft Purview Suite (add-on)
- SharePoint Online & OneDrive
- Microsoft 365 Copilot
- Swiss nLPD compliance requirements

❌ **Not included**
- Microsoft 365 E5
- Advanced Entra ID P2 scenarios
- Full Shadow AI governance (covered in a separate upcoming guide)

---

## 🧩 What this repository contains

This repository provides a **complete test and learning environment**, published as a GitHub Pages site.

### 1️⃣ Online documentation (GitHub Pages)

The guide is available online at:  
🌐 https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/

It includes **23 pages** organized in two parts:

**Tenant preparation** (8 chapters): creating the Microsoft 365 tenant, Entra ID groups, user accounts, SharePoint sites, enabling audit and deploying demo files.

**Purview configuration** (14 chapters): sensitivity labels, custom Swiss SITs (AVS, IBAN, medical data), auto-labelling, DLP policies, DSPM for AI, Endpoint DLP, Insider Risk Management, eDiscovery Premium, Communication Compliance, SharePoint and Power Platform governance.

The site is built with Jekyll using the `jekyll-theme-minimal` theme.

---

### 2️⃣ Demo data for realistic testing
- **`FICHIERS_DEMO_AXONIX.zip`**  
  A ZIP archive containing **demo files from a fictional company (Axonix SA)**, used throughout the procedure:
  - HR documents,
  - finance files,
  - internal and confidential content.

These files allow **end-to-end testing** (auto-labelling, DLP, Copilot, audit).

---

### 3️⃣ Word and PDF versions (v1.0 release)

Word (.docx) and PDF versions of the main guide and the tenant preparation guide are available for download on the [v1.0 release page](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0).

> ⚠️ The online version (GitHub Pages) is the reference version, kept up to date. The Word/PDF files from the v1.0 release are no longer updated.

---

## 🧠 Why Purview matters for Copilot

Microsoft 365 Copilot:
- only sees what the user is authorized to access,
- but **will amplify any over-permission or misconfiguration**.

This repository explains how to:
- classify data correctly,
- restrict risky usage,
- and maintain auditability, **before deploying Copilot widely**.

---

## 🏷️ Licensing model used

| Component | License |
|--------|--------|
| Microsoft 365 Business Premium | Required |
| Microsoft Purview Suite | Required |
| Microsoft 365 Copilot | Optional (but supported) |

💡 **Cost reference** (indicative):
- Purview Suite: approx. CHF 11 / user / month

---

## 🇨🇭 Swiss compliance (nLPD)

The proposed configuration aligns with:
- nLPD Art. 6: purpose limitation and proportionality
- nLPD Art. 8: technical and organizational measures
- nLPD Art. 24: incident traceability and notification

Auditability and evidence collection are **first-class design goals**.

---

## ⚠️ Disclaimer

This repository is provided for **educational and architectural guidance**.

- Always validate configurations in a test tenant first.
- Adapt retention and monitoring settings to your legal context.
- For production deployment, involve legal and compliance stakeholders.

---

## 📌 Upcoming

A **dedicated Shadow AI & Entra ID P2 guide** is planned separately.

## ☕ Support This Project (Optional)

This guide is provided free of charge.

It is the result of several weeks of work, including:
- hands-on technical testing on Microsoft 365 / Purview,
- a methodology specifically adapted to SMB environments,
- consideration of Swiss nLPD (Data Protection Act) requirements.

On a strictly optional basis, individuals or organizations who wish to support
the maintenance of this documentation and its future updates may do so via Ko-fi:

👉 https://ko-fi.com/doit4everyone

This support does not imply any obligation, any associated service,
or any difference in access to the content.


## 🧭 Quick access

🌐 Guide landing page:  
https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/

🏠 Central documentation hub:  
https://doit4everyone.github.io/
