---
title: "Préparation — Vue d'ensemble et prérequis | Configuration Microsoft Purview 2026"
description: "Vue d'ensemble du POC Microsoft Purview : tenant cloud, 15 utilisateurs, 7 groupes de sécurité, 5 sites SharePoint, audit Microsoft 365. Comptes nécessaires et liste des utilisateurs à créer."
lang: fr
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

[← Retour à l'index](../)

> **✅  À LIRE EN PREMIER** Ce document est la SEULE étape à réaliser avant de commencer le guide: Configuration Microsoft Purview (2026). Il ne couvre que la mise en place du tenant, des comptes, des groupes et des sites SharePoint. Une fois ce document terminé et toutes les cases cochées, passez directement au guide Purview.

# Partie 0 — Vue d'ensemble et prérequis

## 0.1 Ce que vous allez construire

Pour tester le guide Microsoft Purview, vous avez besoin d'un tenant Microsoft 365 configuré avec :

- Un tenant cloud (pas de serveur local, pas d'Active Directory on-premises)

- 15 utilisateurs répartis en 4 départements

- 7 groupes de sécurité (4 standards + 3 mail-enabled pour Purview)

- 5 sites SharePoint avec des permissions par département

- L'audit Microsoft 365 activé (requis par Purview)

> **⚠️  CE DOCUMENT NE COUVRE PAS** La configuration Purview elle-même c'est l'objet du guide séparé. Veeam, les sauvegardes, les imprimantes, ou Intune. La configuration Conditional Access avancée ou BitLocker.

> **IMPORTANT : POC VS ENVIRONNEMENT DE PRODUCTION** Cette préparation de tenant est volontairement simplifiée pour un POC. Certaines décisions (partage externe, compte admin unique, absence de séparation des rôles) NE DOIVENT PAS être reproduites telles quelles en production sans validation sécurité et juridique.

## 0.2 Comptes nécessaires avant de commencer

Créez ces comptes et notez les informations dans un gestionnaire de mots de passe (Bitwarden, 1Password).

| **Compte** | **URL** | **Usage** |
| --- | --- | --- |
| Microsoft 365 | admin.microsoft.com | Gestion du tenant : à créer en Partie 1 |
| Microsoft 365 (achat licences) | microsoft.com/fr-ch/microsoft-365 | Achat des licences Business Premium + Purview Suite |

## 0.3 Liste des utilisateurs à créer

Préparez ce tableau. Remplacez demo.ch par votre domaine réel si vous en avez un.

| **Prénom Nom** | **Email** | **Groupe Entra ID** | **Groupe Purview (mail-enabled)** |
| --- | --- | --- | --- |
| Directeur 1 | direction1@demo.ch | GRP-Direction | GRP-Direction-Purview |
| Directeur 2 | direction2@demo.ch | GRP-Direction | GRP-Direction-Purview |
| Directeur 3 | direction3@demo.ch | GRP-Direction | GRP-Direction-Purview |
| RH 1 | rh1@demo.ch | GRP-RH | GRP-RH-Purview |
| RH 2 | rh2@demo.ch | GRP-RH | GRP-RH-Purview |
| Finance 1 | finance1@demo.ch | GRP-Finances | GRP-Finances-Purview |
| Finance 2 | finance2@demo.ch | GRP-Finances | GRP-Finances-Purview |
| Standard 1 à 8 | standard1@demo.ch … standard8@demo.ch | GRP-Standards | (aucun) |

> **ℹ️ POURQUOI DEUX TYPES DE GROUPES ?** GRP-Direction / GRP-RH / GRP-Finances : groupes de sécurité standard, utilisés pour les permissions SharePoint. GRP-Direction-Purview / GRP-RH-Purview / GRP-Finances-Purview : groupes mail-enabled, requis par Azure RMS pour le chiffrement des étiquettes de sensibilité Purview. Les groupes standards NE PEUVENT PAS être utilisés pour le chiffrement. Ce n'est pas interchangeable.

---

[← Retour à l'index](../) | [Préparation — Création du tenant Microsoft 365 →](prep-01-creation-tenant.html)
