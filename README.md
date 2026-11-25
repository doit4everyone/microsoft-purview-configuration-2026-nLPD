🇺🇸 [Read this README in English](README.en.md)

# Configuration Microsoft Purview (2026)
## Guide conformité et IA pour PME

## 📥 Téléchargement

👉 [Télécharger la version Word/PDF (v1.0)](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0)

Ce dépôt contient une **procédure technique complète**, pas à pas, pour configurer **Microsoft Purview** dans un environnement **Microsoft 365 Business Premium**, avec un focus particulier sur :

- la protection des données,
- la gouvernance de Microsoft 365 Copilot,
- la conformité à la **nLPD suisse**.

> **Message clé**  
> La sécurité de Copilot et des données sensibles n'est **pas un problème de licence**.  
> C'est un **problème d'architecture et de configuration**.

---

## 🎯 Périmètre du guide

✅ **Inclus**
- Microsoft 365 Business Premium  
- Microsoft Purview Suite for Microsoft 365 Business Premium (module complémentaire)
- SharePoint Online et OneDrive
- Microsoft 365 Copilot
- Conformité nLPD (Suisse)

❌ **Non inclus**
- Microsoft 365 E5
- Scénarios avancés Entra ID P2
- Gouvernance complète du Shadow AI  
  *(traitée dans un [guide séparé](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/))*

---

## 🧩 Contenu du dépôt

Ce dépôt fournit un **environnement de test complet et reproductible** publié sous forme de site GitHub Pages.

### 1️⃣ Documentation en ligne (GitHub Pages)

Le guide est consultable directement en ligne à l'adresse :  
🌐 https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/

Il comprend **23 pages** organisées en deux parties :

**Préparation du tenant** (8 chapitres) : création du tenant Microsoft 365, groupes Entra ID, comptes utilisateurs, sites SharePoint, activation de l'audit et déploiement des fichiers de démonstration.

**Configuration Purview** (14 chapitres) : étiquettes de sensibilité, SIT personnalisés suisses (AVS, IBAN, données médicales), auto-labelling, politiques DLP, DSPM for AI, Endpoint DLP, Insider Risk Management, eDiscovery Premium, Communication Compliance, gouvernance SharePoint et Power Platform.

Le site utilise Jekyll avec le thème `jekyll-theme-minimal`.

---

### 2️⃣ Données de démonstration pour les tests
- **`FICHIERS_DEMO_AXONIX.zip`**  
  Une archive ZIP contenant des **fichiers de démonstration d'une société fictive (Axonix SA)**, utilisés tout au long de la procédure :
  - documents RH,
  - fichiers financiers,
  - contenus internes et confidentiels.

Ces fichiers permettent des **tests complets de bout en bout**  
(auto-labelling, DLP, Copilot, audit).

---

### 3️⃣ Versions Word et PDF (release v1.0)

Les versions Word (.docx) et PDF du guide principal et du guide de préparation du tenant restent disponibles en téléchargement sur la [page de release v1.0](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0).

> ⚠️ La version en ligne (GitHub Pages) est la version de référence, maintenue à jour. Les fichiers Word/PDF de la release v1.0 ne sont plus mis à jour.

---

## 🧠 Pourquoi Purview est indispensable avec Copilot

Microsoft 365 Copilot :
- respecte strictement les permissions utilisateur,
- mais **amplifie toute erreur de sur-partage ou de gouvernance**.

Ce dépôt montre comment :
- classifier correctement les données,
- encadrer les usages à risque,
- garantir la traçabilité,
**avant** un déploiement Copilot à grande échelle.

---

## 🏷️ Modèle de licence utilisé

| Composant | Licence |
|--------|--------|
| Microsoft 365 Business Premium | Requis |
| Microsoft Purview Suite for Microsoft 365 Business Premium | Requis |
| Microsoft 365 Copilot | Optionnel (pris en charge) |

💡 **Ordre de grandeur des coûts** :
- Microsoft 365 Business Premium : 17.82 CHF / utilisateur / mois
- Microsoft Purview Suite for Microsoft 365 Business Premium : 8.10 CHF / utilisateur / mois
- Total : 25.92 CHF / utilisateur / mois

*Prix publics Microsoft Suisse, hors TVA, abonnement annuel avec paiement annuel (novembre 2026). Les tarifs peuvent varier selon le revendeur et le mode de paiement.*

---

## 🇨🇭 Conformité nLPD

La configuration proposée répond notamment aux exigences suivantes :
- art. 6 nLPD : finalité et proportionnalité
- art. 8 nLPD : mesures techniques et organisationnelles
- art. 24 nLPD : annonce au PFPDT des violations de la sécurité des données présentant vraisemblablement un risque élevé

L'auditabilité et la preuve de conformité sont des **objectifs centraux**.

---

## ⚠️ Avertissement

Ce dépôt est fourni à des fins **pédagogiques et architecturales**.

- Testez toujours dans un tenant de démonstration.
- Adaptez les durées de rétention et les contrôles à votre contexte légal.
- En production, impliquez les parties juridiques et conformité.

---

## 📌 Guide complémentaire

Le **[guide dédié au Shadow AI et à la gouvernance Microsoft 365](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/)** est disponible : MDCA, Purview DLP, DSPM for AI, Insider Risk Management, DLP inline Edge for Business.

---

## ☕ Soutenir ce projet (optionnel)

Ce guide est mis à disposition gratuitement.

Il est le fruit de plusieurs semaines de travail, incluant :
- des tests techniques terrain sur Microsoft 365 / Purview,
- une validation méthodologique orientée PME,
- une prise en compte des exigences de la nLPD (Suisse).

À titre strictement optionnel, les personnes ou organisations souhaitant soutenir
la maintenance de cette documentation et ses évolutions futures peuvent le faire
via la plateforme Ko-fi :

👉 **[Soutenir le projet sur Ko-fi](https://ko-fi.com/doit4everyone)**

Aucune contrepartie, aucun service associé et aucune différence de traitement
n'est liée à ce soutien.

## 🧭 Accès rapide

🌐 Page de présentation du guide :    https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/

🏠  Retrouvez tous les guides sur :   https://doit4everyone.github.io/
