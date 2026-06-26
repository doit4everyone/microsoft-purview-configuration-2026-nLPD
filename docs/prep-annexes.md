---
title: "Préparation — Annexes | Configuration Microsoft Purview 2026"
description: "Checklist de validation avant de démarrer le guide Purview, glossaire et portails utiles pour la préparation du tenant."
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

# Annexes

## Annexe A — Checklist de validation

Cochez chaque item avant de passer au guide Microsoft Purview. Ne commencez pas le guide si une case n'est pas cochée.

Phase 1 : Tenant et licences

- ☐  Tenant demo.onmicrosoft.com créé et accessible sur admin.microsoft.com

- ☐  15 licences Microsoft 365 Business Premium attribuées

- ☐  15 licences Microsoft Purview Suite attribuées

- ☐  MFA activé sur le compte admin

Phase 2 : Groupes

- ☐  GRP-Direction créé avec direction1, direction2, direction3

- ☐  GRP-RH créé avec rh1, rh2

- ☐  GRP-Finances créé avec finance1, finance2

- ☐  GRP-Standards créé avec standard1 à standard8

- ☐  GRP-Direction-Purview créé (mail-enabled) avec les 3 directeurs

- ☐  GRP-RH-Purview créé (mail-enabled) avec rh1, rh2

- ☐  GRP-Finances-Purview créé (mail-enabled) avec finance1, finance2

Phase 3 : SharePoint

- ☐  Hub Site « Intranet demo.ch » créé et enregistré comme hub

- ☐  Site « Communication demo.ch » créé et associé au hub

- ☐  Site « Direction demo.ch » créé, permissions : GRP-Direction uniquement

- ☐  Site « RH demo.ch » créé, permissions : GRP-RH (membres) + GRP-Direction (lecture)

- ☐  Site « Finances demo.ch » créé, permissions : GRP-Finances (membres) + GRP-Direction (lecture)

- ☐  Partage externe activé au niveau du tenant (Stratégies > Partage > Personnes spécifiques)

- ☐  Site « Transmission externe » créé, racine en lecture seule pour tous sauf admin

- ☐  Bibliothèque « RH – Transmission externe » créée, accès : GRP-RH-Purview + admin uniquement

- ☐  Bibliothèque « Finance - Transmission externe » créée, accès : GRP-Finances-Purview + admin uniquement

- ☐  Bibliothèque « Direction - Transmission externe » créée, accès : GRP-Direction-Purview + admin uniquement

- ☐  Partage externe configuré sur le site Transmission externe (niveau : Personnes spécifiques)

Phase 4 : Audit et documents de démonstration

- ☐  Audit Microsoft 365 activé dans compliance.microsoft.com

- ☐  Site Communication : fichiers 01, 03, 07 déposés

- ☐  Site Direction : fichiers 04, 05, 08 déposés

- ☐  Site RH : fichiers 11, 12, 13, 14, 15, 16 déposés (dont 14 = fichier clé AVS + IBAN)

- ☐  Site Finances : fichiers 06, 10, 17, 18 déposés (dont 18 contient IBAN)

- ☐  Site Intranet : fichier 02 déposé

> **🚀  PRÊT À COMMENCER LE GUIDE PURVIEW** Si toutes les cases ci-dessus sont cochées, vous pouvez ouvrir le document : Configuration Microsoft Purview (2026). Commencez par la Partie 0 du guide elle suppose que votre tenant est dans l'état décrit ici.

## Annexe B — Glossaire

| **Terme** | **Définition simple** |
| --- | --- |
| Entra ID | L'annuaire des utilisateurs et groupes Microsoft 365 (anciennement Azure Active Directory). |
| Groupe de sécurité | Groupe d'utilisateurs utilisé pour gérer les accès SharePoint et les politiques. |
| Mail-enabled | Groupe de sécurité avec une adresse email. Requis pour le chiffrement Purview (Azure RMS). |
| Hub Site | Site SharePoint désigné comme « central ». Les autres sites s'y associent pour la navigation. |
| Audit M365 | Journal de toutes les actions dans le tenant (connexions, accès fichiers, modifications). |
| Purview Suite | Licence complémentaire à Business Premium. Débloque les étiquettes chiffrées, l'auto-labelling avancé, l'eDiscovery Premium. |
| Azure RMS | Moteur de chiffrement de Microsoft, utilisé par les étiquettes de sensibilité Purview pour chiffrer les documents. |
| nLPD | Nouvelle Loi sur la Protection des Données : Loi suisse en vigueur depuis septembre 2023. |
| POC | Proof of Concept :Test technique sur un environnement de démonstration avant déploiement en production. |

## Annexe C — Portails utiles

| **Portail** | **URL** | **Usage dans ce document** |
| --- | --- | --- |
| Centre admin M365 | admin.microsoft.com | Groupes, utilisateurs, licences |
| Centre admin SharePoint | admin.microsoft.com/sharepoint | Création et gestion des sites |
| Centre admin Exchange | admin.exchange.microsoft.com | Groupes mail-enabled |
| Portail Entra ID | entra.microsoft.com | Vérification des groupes |
| Purview Compliance | compliance.microsoft.com | Activation de l'audit |

---

[← Préparation — Documents de démonstration](prep-06-documents-demo.html) | [Préface — Configuration Microsoft Purview 2026 →](00-preface.html)
