🇺🇸 [Read this README in English](README.en.md)

# Configuration Microsoft Purview (2026)
## Guide conformité et IA pour PME

Ce dépôt contient une **procédure technique complète**, pas à pas, pour configurer **Microsoft Purview** dans un environnement **Microsoft 365 Business Premium**, avec un focus particulier sur :

- la protection des données,
- la gouvernance de Microsoft 365 Copilot,
- la conformité à la **nLPD suisse**.

> **Message clé**  
> La sécurité de Copilot et des données sensibles n’est **pas un problème de licence**.  
> C’est un **problème d’architecture et de configuration**.

---

## 🎯 Périmètre du guide

✅ **Inclus**
- Microsoft 365 Business Premium  
- Microsoft Purview Suite (add-on)
- SharePoint Online et OneDrive
- Microsoft 365 Copilot
- Conformité nLPD (Suisse)

❌ **Non inclus**
- Microsoft 365 E5
- Scénarios avancés Entra ID P2
- Gouvernance complète du Shadow AI  
  *(traitée dans un guide séparé à venir)*

---

## 🧩 Contenu du dépôt

Ce dépôt fournit un **environnement de test complet et reproductible**, comprenant :

### 1️⃣ Procédure principale de configuration
- **`Configuration Microsoft Purview (2026).docx`**  
  Le guide technique principal, décrivant la configuration de :
  - étiquettes de sensibilité,
  - auto‑labelling,
  - politiques DLP,
  - protection des usages Copilot,
  - audit et DSPM for AI,
  - gouvernance des accès SharePoint.

---

### 2️⃣ Préparation du tenant de test (from scratch)
- **`Préparation_Tenant_POC_Purview.docx`**  
  Un document dédié expliquant **comment déployer un tenant Microsoft 365 de test depuis zéro**, spécifiquement conçu pour ce POC Purview :
  - hypothèses de création du tenant,
  - licences nécessaires,
  - configuration de base avant l’application des politiques Purview.

Ce document garantit un **environnement de test propre et reproductible**.

---

### 3️⃣ Données de démonstration pour les tests
- **`FICHIERS DEMO AXIOS.zip`**  
  Une archive ZIP contenant des **fichiers de démonstration d’une société fictive (AXIOS)**, utilisés tout au long de la procédure :
  - documents RH,
  - fichiers financiers,
  - contenus internes et confidentiels.

Ces fichiers sont référencés dans :
- `Préparation_Tenant_POC_Purview.docx`,
- et la procédure principale,

afin de permettre des **tests complets de bout en bout**  
(auto‑labelling, DLP, Copilot, audit).

---

## 🧠 Pourquoi Purview est indispensable avec Copilot

Microsoft 365 Copilot :
- respecte strictement les permissions utilisateur,
- mais **amplifie toute erreur de sur‑partage ou de gouvernance**.

Ce dépôt montre comment :
- classifier correctement les données,
- encadrer les usages à risque,
- garantir la traçabilité,
**avant** un déploiement Copilot à grande échelle.

---

## 🏷️ Modèle de licence utilisé

| Composant | Licence |
|--------|--------|
| Microsoft 365 Business Premium | Requis |
| Microsoft Purview Suite | Requis |
| Microsoft 365 Copilot | Optionnel (pris en charge) |

💡 **Ordre de grandeur des coûts** :
- Purview Suite : ~CHF 11 / utilisateur / mois

---

## 🇨🇭 Conformité nLPD

La configuration proposée répond notamment aux exigences suivantes :
- art. 6 nLPD — finalité et proportionnalité,
- art. 8 nLPD — mesures techniques et organisationnelles,
- art. 24 nLPD — traçabilité et notification des violations.

L’auditabilité et la preuve de conformité sont des **objectifs centraux**.

---

## ⚠️ Avertissement

Ce dépôt est fourni à des fins **pédagogiques et architecturales**.

- Testez toujours dans un tenant de démonstration.
- Adaptez les durées de rétention et les contrôles à votre contexte légal.
- En production, impliquez les parties juridiques et conformité.

---

## 📌 À venir

Un **guide dédié au Shadow AI et à Entra ID P2** sera publié séparément.
``
---

## ☕ Soutenir ce projet (optionnel)

Ce guide est mis à disposition gratuitement.

Il est le fruit de plusieurs semaines de travail, incluant :
- des tests techniques terrain sur Microsoft 365 / Purview,
- une validation méthodologique orientée PME,
- une prise en compte des exigences de la nLPD (Suisse).

À titre strictement optionnel, les personnes ou organisations souhaitant soutenir
la maintenance de cette documentation et ses évolutions futures peuvent le faire
via la plateforme Ko‑fi :

👉 **[Soutenir le projet sur Ko-fi](https://ko-fi.com/doit4everyone)**

Aucune contrepartie, aucun service associé et aucune différence de traitement
n’est liée à ce soutien.
``
## 🧭 Accès rapide

🌐 Page de présentation du guide  
https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/

🏠 Page centrale des guides  
https://doit4everyone.github.io/
``
