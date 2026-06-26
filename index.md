---
title: "Configuration Microsoft Purview 2026 — Guide complet Purview Suite pour PME suisses (nLPD) | DoIt4Everyone"
description: "Guide complet de configuration de Microsoft Purview Suite depuis un tenant vierge : étiquettes de sensibilité, SIT personnalisés suisses (AVS, IBAN, données médicales), auto-labelling, politiques DLP, DSPM for AI, Endpoint DLP, Insider Risk Management, eDiscovery, Communication Compliance, gouvernance SharePoint et Power Platform. Conformité nLPD pour PME suisses."
lang: fr
permalink: /
---
<style>
  header, footer { display: none !important; }
  .wrapper {
    max-width: 900px !important;
    margin: 0 auto !important;
    float: none !important;
    position: relative !important;
    padding: 40px 20px !important;
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif !important;
    font-size: 1.1em !important;
  }
  section {
    width: 100% !important;
    float: none !important;
    margin: 0 !important;
    padding-top: 0 !important;
  }
  h1, h2 { text-align: center; }
  table { width: 100%; display: table; margin: 20px 0; }
</style>

# 🛡️ Configuration Microsoft Purview 2026

<p align="center">
<strong>Purview Suite (Business Premium) — Guide complet</strong><br>
<em>Conformité nLPD — PME Suisse 🇨🇭 — Version 1.0</em>
</p>

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Disponible-brightgreen)](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## 📖 À propos

| | |
|---|---|
| **Tenant** | demo.onmicrosoft.com |
| **Licence** | Microsoft 365 Business Premium + Purview Suite |
| **Niveau** | Moyen — chaque étape expliquée en détail |
| **Version** | 1.0 — 2026 |
| **Objectif** | Protéger les données sensibles (nLPD) |

> ℹ️ **Guide complémentaire :** une fois Purview configuré, consultez [Gouvernance Shadow AI Microsoft 365](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/) pour gouverner et surveiller le Shadow AI (MDCA, DLP inline Edge for Business, Insider Risk Management).

---

## 📥 Fichiers et ressources

Le guide de préparation du tenant est intégré directement dans ce site (section ci-dessous). Les versions Word/PDF et le pack de fichiers de démo restent disponibles sur la page de [release v1.0](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0) :

| Fichier | Description |
|---|---|
| `Configuration_Microsoft_Purview_2026.docx` / `.pdf` | Ce guide en version Word et PDF |
| `Preparation_Tenant_POC_Purview.docx` / `.pdf` | Version Word/PDF du guide de préparation du tenant (identique à la version intégrée ci-dessous) |
| `FICHIERS_DEMO_AXONIX.zip` | Fichiers de démonstration utilisés dans les scénarios de test (Partie 6) : contrat de travail, fiches de salaire, registre du personnel, grand livre |

---

## Chronologie de déploiement

Cette procédure exhaustive peut être déployée progressivement sans interruption de service. Chaque semaine représente un palier stable.

| Période | Actions |
|---|---|
| Semaine 1 | Socle tenant : groupes de rôles, licences Purview Suite, audit, SIT personnalisés (AVS-Suisse-PME, Medical-RH-Suisse). |
| Semaine 2 | Étiquettes de sensibilité (7 étiquettes, publication) — Auto-labellisation (simulation + activation) — Politiques de rétention (5 politiques). |
| Semaine 3 | Politiques DLP (Exchange, SharePoint, Copilot, Transmission externe) — Endpoint DLP + Edge for Business. |
| Semaine 4 | DSPM for AI — Insider Risk Management (3 stratégies) — eDiscovery Premium — Communication Compliance — Étiquette 6 « Transmission-Externe ». |

> ℹ️ Si vous débutez, il est possible de créer un environnement de preuve de concept avec des licences d'évaluation Microsoft 365 : vous disposerez de deux mois pour déployer l'ensemble des configurations, les tester et acquérir la maîtrise opérationnelle de la solution.

## Positionnement de Microsoft Purview

Microsoft Purview est un outil de gouvernance et de protection des données, pas un outil de sécurité périmétrique. Il suppose des fondations saines.

✅ **Ce que Purview apporte**
- Une couche de protection centrée sur la donnée elle-même : les étiquettes et les contrôles voyagent avec le document, quel que soit l'endroit où il se trouve.
- Une défense en profondeur efficace et démontrable face au PFPDT, lorsqu'il est déployé sur des fondations saines.

❌ **Ce qu'il ne remplace pas**
- **Antivirus / EDR :** Purview ne détecte pas les logiciels malveillants ni les menaces sur les postes. Microsoft Defender (ou équivalent) reste un prérequis indépendant.
- **SIEM :** la corrélation des événements de sécurité et la détection des attaques relèvent d'une couche distincte (Microsoft Sentinel ou équivalent).
- **Structure de permissions :** Purview ne corrige pas une erreur de gouvernance des accès. Une structure de droits cohérente sur les sites, bibliothèques et dossiers est un préalable indispensable (voir Partie 11).
- **Pare-feu et proxy réseau :** le contrôle du trafic sortant relève d'une couche réseau dédiée.
- **Sauvegarde et continuité :** la rétention Purview garantit la conservation légale des données, pas leur restauration en cas de sinistre.

⚠️ Déployé sur des fondations fragiles, Purview donnera une fausse impression de sécurité. Déployé correctement, il constitue une défense en profondeur efficace et démontrable.

---

## 🧱 Préparation du tenant (à faire avant ce guide)

Si votre tenant Microsoft 365 n'est pas encore prêt, suivez d'abord ce guide pas à pas : il crée le tenant, les 15 comptes utilisateurs, les 7 groupes, les 5 sites SharePoint et l'audit nécessaires au guide Purview.

| Chapitre | Description |
|---|---|
| [Vue d'ensemble et prérequis](docs/prep-00-vue-ensemble.html) | Ce que vous allez construire, comptes nécessaires, liste des 15 utilisateurs |
| [Création du tenant Microsoft 365](docs/prep-01-creation-tenant.html) | Créer le tenant et ajouter les licences Purview Suite |
| [Création des groupes Entra ID](docs/prep-02-groupes-entra-id.html) | 4 groupes de sécurité standards + 3 groupes mail-enabled (PowerShell Exchange Online) |
| [Création des comptes utilisateurs](docs/prep-03-comptes-utilisateurs.html) | 15 comptes, affectation aux groupes, MFA administrateur |
| [Création des sites SharePoint](docs/prep-04-sites-sharepoint.html) | Intranet, Communication, Direction, RH, Finances et Transmission externe (partage externe sécurisé) |
| [Activer l'audit Microsoft 365](docs/prep-05-audit.html) | Première étape requise par le guide Purview |
| [Documents de démonstration](docs/prep-06-documents-demo.html) | Répartition des fichiers de démo (AVS, IBAN) dans les sites SharePoint |
| [Annexes](docs/prep-annexes.html) | Checklist de validation, glossaire, portails utiles |

---

## 🗂️ Sommaire complet — Configuration Microsoft Purview 2026

| Chapitre | Description |
|---|---|
| [Préface](docs/00-preface.html) | Contexte du projet, chronologie de déploiement, positionnement de Purview, limites du dispositif |
| [Partie 0 — Avant de commencer](docs/01-avant-de-commencer.html) | Accès au portail, PowerShell Exchange Online, rôles administrateur, vérification des licences |
| [Partie 1 — Introduction et concepts Purview](docs/02-introduction-concepts.html) | Les 4 piliers de Purview, ordre de configuration obligatoire, activation de l'audit |
| [Partie 2 — Les Étiquettes de sensibilité (Labels)](docs/03-etiquettes-sensibilite.html) | Groupes mail-enabled, SIT personnalisés, création et publication des 7 étiquettes, break-glass RMS |
| [Partie 3 — Auto-labelling](docs/04-auto-labelling.html) | Étiquetage automatique des données sensibles (AVS, IBAN, données médicales) |
| [Partie 4 — Politiques DLP](docs/05-politiques-dlp.html) | Politique DLP principale, transmission externe, protection Copilot |
| [Partie 5 — Audit et DSPM for AI](docs/06-audit-dspm-ai.html) | Surveillance Copilot, Activity Explorer, recherche dans les journaux d'audit |
| [Partie 6 — Scénarios de test et validation](docs/07-tests-validation.html) | Tests auto-labelling, DLP, protection Copilot, checklist finale |
| [Partie 7 — Gestion opérationnelle et démos](docs/08-gestion-operationnelle.html) | Suspendre les protections pour une démo, vérifications hebdomadaires, dette de labellisation |
| [Partie 8 — Fonctionnalités avancées Purview](docs/09-fonctionnalites-avancees.html) | Rétention, Insider Risk Management, eDiscovery Premium, Communication Compliance |
| [Partie 9 — Endpoint DLP, Edge for Business et AI Hub](docs/10-endpoint-dlp-aihub.html) | Endpoint DLP, contrôles IA dans Edge, extension navigateur via Intune, AI Hub |
| [Partie 10 — Gouvernance Copilot Studio et Power Platform](docs/11-power-platform.html) | Politique DLP connecteur, inventaire des agents, Managed Environments |
| [Partie 11 — Gouvernance des accès SharePoint](docs/12-sharepoint.html) | Modèle de permissions, audit du sur-partage, moindre privilège, OneDrive |
| [Annexes](docs/13-annexes.html) | Glossaire, articles nLPD, SIT personnalisés, RACI, levée d'anonymat IRM, comptes et rôles, registre des activités de traitement |

---

## 📚 Guide complémentaire

| Document | Description |
|---|---|
| [Gouvernance Shadow AI Microsoft 365](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/) | Une fois Purview configuré : MDCA, Purview DLP, DSPM for AI, Insider Risk Management, DLP inline Edge for Business |

---

## ⚖️ Avertissement

Ce guide a été rédigé et validé terrain sur la base des interfaces Microsoft 365 et Microsoft Purview telles qu'elles se présentaient au moment de sa rédaction (avril 2026). Microsoft déploie des mises à jour d'interface de manière continue et sans préavis : les chemins de navigation, les libellés de boutons, la disposition des menus et la disponibilité de certaines fonctionnalités sont susceptibles d'évoluer à tout moment.

Ce guide est fourni à titre informatif et pédagogique uniquement. Il ne constitue pas un avis juridique et ne remplace pas un accompagnement professionnel adapté à votre organisation. La conformité à la nLPD relève de la responsabilité exclusive de l'organisation qui déploie ces configurations.

---

ℹ️ *Ce document est mis à disposition gratuitement, fruit de plusieurs semaines de tests terrain sur un tenant Microsoft 365 / Purview réel. Aucune contrepartie financière n'est requise. Voir l'[Annexe J](docs/13-annexes.html) pour plus de détails.*
