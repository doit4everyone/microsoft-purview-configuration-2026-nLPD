---
title: "Préparation — Création du tenant Microsoft 365 | Configuration Microsoft Purview 2026"
description: "Créer un tenant Microsoft 365 Business Premium et ajouter les licences Purview Suite pour le POC."
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

# Partie 1 — Création du tenant Microsoft 365

## 1.1 Créer le tenant

Le tenant est votre espace privatif dans le cloud Microsoft. Toute la configuration repose sur lui.

1. Ouvrez votre navigateur et allez sur https://www.microsoft.com/fr-ch/microsoft-365/business/compare-all-plans

2. Cliquez sur « Essai gratuit » ou « Acheter maintenant » pour Microsoft 365 Business Premium.

3. Dans « Créez votre compte », entrez une adresse email personnelle temporaire.

4. Microsoft vous attribue un domaine temporaire : demo.onmicrosoft.com, acceptez-le.

5. Notez précieusement: admin@demo.onmicrosoft.com et son mot de passe.

6. Choisissez 15 licences Microsoft 365 Business Premium.

7. Entrez vos informations de paiement et finalisez la commande.

> **✅  CONSEIL** Pour un POC, le domaine demo.onmicrosoft.com suffit, vous n'avez pas besoin de configurer un domaine demo.ch réel, cependant pour le guide Purview on utilisera le domaine demo.ch. Si vous souhaitez utiliser un vrai domaine, ajoutez-le dans admin.microsoft.com > Paramètres > Domaines après la création du tenant.

## 1.2 Ajouter les licences Purview Suite

Purview Suite est une licence complémentaire à Business Premium. Elle est indispensable pour activer les fonctionnalités avancées du guide Purview.

1. Connectez-vous sur https://admin.microsoft.com avec votre compte admin.

2. Dans le menu gauche, cliquez sur Facturation > Acheter des services.

3. Cherchez « Microsoft Purview Suite » dans la barre de recherche.

4. Achetez autant de licences que d'utilisateurs qui vont tester Purview (15 minimum).

5. Cliquez sur Acheter. La licence apparaît dans Facturation > Vos produits.

> **⚠️ ATTENTION : NE PAS OUBLIER D'ATTRIBUER LA LICENCE** Acheter la licence ne suffit pas. Vous devez l'attribuer à chaque utilisateur. L'attribution se fait automatiquement si vous la cochez lors de la création des comptes (Partie 2). Un utilisateur sans licence Purview Suite ne pourra pas utiliser les étiquettes chiffrées ni l'auto-labelling.

---

[← Préparation — Vue d'ensemble et prérequis](prep-00-vue-ensemble.html) | [Préparation — Création des groupes Entra ID →](prep-02-groupes-entra-id.html)
