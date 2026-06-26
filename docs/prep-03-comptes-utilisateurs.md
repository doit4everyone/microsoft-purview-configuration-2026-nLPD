---
title: "Préparation — Création des comptes utilisateurs | Configuration Microsoft Purview 2026"
description: "Créer les 15 comptes utilisateurs, les affecter à leurs groupes et activer le MFA pour l'administrateur."
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

# Partie 3 — Création des comptes utilisateurs

## 3.1 Créer les 15 comptes

Pour chaque utilisateur, répétez la procédure suivante. Commencez par les Directeurs, puis RH, puis Finances, puis Standards.

1. Allez dans admin.microsoft.com > Utilisateurs > Utilisateurs actifs.

2. Cliquez sur + Ajouter un utilisateur.

3. Remplissez : Prénom, Nom, Nom d'affichage, Nom d'utilisateur (@demo.ch).

4. Mot de passe : cochez « Laisser l'utilisateur créer son propre mot de passe » + « Envoyer le mot de passe par email ».

5. Licences : cochez Microsoft 365 Business Premium ET Microsoft Purview Suite.

6. Cliquez sur Terminer.

> **⚠️ LICENCES : COCHEZ LES DEUX** Business Premium : obligatoire pour Exchange, SharePoint, Teams. Purview Suite : obligatoire pour les étiquettes chiffrées, l'auto-labelling et les fonctionnalités avancées du guide Purview. Si vous oubliez Purview Suite maintenant, vous pouvez l'ajouter plus tard dans : Utilisateurs actifs > cliquez sur l'utilisateur > Licences et apps.

## 3.2 Affecter les utilisateurs à leurs groupes

Une fois tous les comptes créés, affectez chaque utilisateur à son groupe Entra ID ET à son groupe Purview.

1. Allez dans admin.microsoft.com > Équipes et groupes > Groupes actifs.

2. Cliquez sur GRP-Direction.

3. Onglet Membres > + Ajouter des membres > sélectionnez direction1, direction2, direction3.

4. Répétez pour GRP-RH (rh1, rh2), GRP-Finances (finance1, finance2), GRP-Standards (standard1 à 8).

> **ℹ️ GROUPES PURVIEW (mail-enabled)** Si vous n'avez pas encore ajouté les membres aux groupes GRP-*-Purview lors de la création PowerShell (Partie 2), faites-le maintenant via Exchange Online. Centre d'administration Exchange (admin.exchange.microsoft.com) > Destinataires > Groupes > cliquez sur GRP-Direction-Purview > onglet Membres.

## 3.3 Activer le MFA pour l'administrateur

Cette étape est indispensable avant de continuer. Elle protège le compte admin qui contrôle tout le tenant.

1. Allez sur https://aka.ms/mfasetup avec votre compte admin.

2. Cliquez sur Ajouter une méthode > Application d'authentification.

3. Téléchargez Microsoft Authenticator sur votre téléphone (App Store ou Google Play).

4. Scannez le QR code affiché avec l'application.

5. Test : déconnectez-vous et reconnectez-vous. Le téléphone doit demander une confirmation.

> **NOTE D’ARCHITECTURE : COMPTE ADMIN** Dans le cadre du POC, un seul compte administrateur est utilisé pour simplifier la mise en œuvre. En production, ce modèle doit évoluer vers : •  au minimum deux compte de secours (break-glass), •  une séparation entre administration technique et sécurité.

---

[← Préparation — Création des groupes Entra ID](prep-02-groupes-entra-id.html) | [Préparation — Création des sites SharePoint →](prep-04-sites-sharepoint.html)
