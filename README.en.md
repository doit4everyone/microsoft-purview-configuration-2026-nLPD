🇫🇷 [Lire ce README en français](README.md)

# Microsoft Purview Configuration (2026)
## Compliance & AI Architecture Guide for SMBs

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

This repository provides a **complete test and learning environment**, including:

### 1️⃣ Core configuration procedure
- **`Configuration Microsoft Purview (2026).docx`**  
  The main technical guide, explaining how to configure:
  - sensitivity labels,
  - auto-labeling,
  - DLP policies,
  - Copilot protection,
  - audit and DSPM for AI,
  - SharePoint access governance.

---

### 2️⃣ Test tenant preparation (from scratch)
- **`Préparation_Tenant_POC_Purview.docx`**  
  A dedicated document explaining **how to deploy a Microsoft 365 test tenant from scratch**, specifically prepared for this Purview POC:
  - tenant creation assumptions,
  - required licenses,
  - base configuration before applying Purview policies.

This document ensures you start from a **clean, reproducible test environment**.

---

### 3️⃣ Demo data for realistic testing
- **`FICHIERS DEMO AXIOS.zip`**  
  A ZIP archive containing **demo files from a fictional company (AXIOS)**, used throughout the procedure:
  - HR documents,
  - finance files,
  - internal and confidential content.

These files are referenced in:
- `Préparation_Tenant_POC_Purview.docx`,
- and the main Purview configuration guide,

to allow **end‑to‑end testing** (auto‑labelling, DLP, Copilot, audit).

---

## 🧠 Why Purview matters for Copilot

Microsoft 365 Copilot:
- only sees what the user is authorized to access,
- but **will amplify any over-permission or misconfiguration**.

This repository explains how to:
- classify data correctly,
- restrict risky usage,
- and maintain auditability — **before deploying Copilot widely**.

---

## 🏷️ Licensing model used

| Component | License |
|--------|--------|
| Microsoft 365 Business Premium | Required |
| Microsoft Purview Suite | Required |
| Microsoft 365 Copilot | Optional (but supported) |

💡 **Cost reference** (indicative):
- Purview Suite: ~CHF 11 / user / month

---

## 🇨🇭 Swiss compliance (nLPD)

The proposed configuration aligns with:
- nLPD Art. 6 (purpose limitation & proportionality)
- nLPD Art. 8 (technical and organizational measures)
- nLPD Art. 24 (incident traceability and notification)

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
the maintenance of this documentation and its future updates may do so via Ko‑fi:

👉 https://ko-fi.com/doit4everyone

This support does not imply any obligation, any associated service,
or any difference in access to the content.

🌐 Guide landing page
https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/
``
