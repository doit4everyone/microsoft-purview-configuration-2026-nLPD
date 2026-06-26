---
title: "Préparation — Création des groupes Entra ID | Configuration Microsoft Purview 2026"
description: "Créer les 4 groupes de sécurité standards et les 3 groupes mail-enabled pour Purview via PowerShell Exchange Online."
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

# Partie 2 — Création des groupes Entra ID

## 2.1 Créer les 4 groupes de sécurité standards

Ces groupes servent à gérer les accès SharePoint. Ils doivent être créés en premier.

1. Connectez-vous sur https://admin.microsoft.com.

2. Dans le menu gauche, cliquez sur Équipes et groupes > Groupes actifs.

3. Cliquez sur + Ajouter un groupe.

4. Type de groupe : choisissez Sécurité. Cliquez sur Suivant.

5. Créez les 4 groupes suivants (répétez les étapes 3 et 4 pour chacun) :

| **Nom du groupe** | **Description** | **Membres à ajouter** |
| --- | --- | --- |
| GRP-Direction | Directeurs - accès étendu RH et Finances | direction1, direction2, direction3 |
| GRP-RH | Équipe Ressources Humaines | rh1, rh2 |
| GRP-Finances | Équipe Finances et Comptabilité | finance1, finance2 |
| GRP-Standards | Utilisateurs standards | standard1 à standard8 |

> **ℹ️  CONSEIL : AJOUTER LES MEMBRES PLUS TARD** Vous pouvez créer les groupes maintenant (vides) et ajouter les membres après avoir créé les comptes utilisateurs (Partie 3). Pour ajouter un membre : Groupes actifs > cliquez sur le groupe > onglet Membres > + Ajouter des membres.

## 2.2 Créer les 3 groupes mail-enabled pour Purview

Ces groupes sont techniquement différents des précédents. Ils doivent être créés via PowerShell dans Exchange Online, le portail web ne permet pas de créer ce type de groupe directement.

> **⚠️ POURQUOI POWERSHELL EST OBLIGATOIRE ICI** Les groupes de sécurité standard du portail web ne sont pas mail-enabled. Azure RMS (moteur de chiffrement de Purview) ne reconnaît que les groupes qui ont une adresse email. PowerShell est le seul moyen de créer ces groupes mail-enabled dans un tenant cloud pur. **Toutes les manipulations en Powershell pour Exchange Online sont à faire sur un poste local en Powershell 5.x. Le cloud-Shell supporte mal ce module**.

### Étape 1 — Installer le module Exchange Online PowerShell

Ouvrez PowerShell en administrateur (clic droit sur l'icône PowerShell > Exécuter en tant qu'administrateur).

Install-Module -Name ExchangeOnlineManagement -Force -AllowClobber

### Étape 2 — Se connecter à Exchange Online

Connect-ExchangeOnline

→ Une fenêtre de connexion Microsoft s'ouvre. Connectez-vous avec votre compte admin.

### Étape 3 — Créer les 3 groupes

Remplacez demo.ch par votre domaine réel si différent.

New-DistributionGroup -Name "GRP-Direction-Purview" -Alias "grp-direction-purview" -Type Security -PrimarySmtpAddress "grp-direction-purview@demo.ch"

New-DistributionGroup -Name "GRP-RH-Purview" -Alias "grp-rh-purview" -Type Security -PrimarySmtpAddress "grp-rh-purview@demo.ch"

New-DistributionGroup -Name "GRP-Finances-Purview" -Alias "grp-finances-purview" -Type Security -PrimarySmtpAddress "grp-finances-purview@demo.ch"

### Étape 4 — Ajouter les membres

Répétez pour chaque membre de chaque groupe. Remplacez les adresses email par les vôtres.

`# GRP-Direction-Purview`

Add-DistributionGroupMember -Identity "GRP-Direction-Purview" -Member "direction1@demo.ch"

Add-DistributionGroupMember -Identity "GRP-Direction-Purview" -Member "direction2@demo.ch"

Add-DistributionGroupMember -Identity "GRP-Direction-Purview" -Member "direction3@demo.ch"

`# GRP-RH-Purview`

Add-DistributionGroupMember -Identity "GRP-RH-Purview" -Member "rh1@demo.ch"

Add-DistributionGroupMember -Identity "GRP-RH-Purview" -Member "rh2@demo.ch"

`# GRP-Finances-Purview`

Add-DistributionGroupMember -Identity "GRP-Finances-Purview" -Member "finance1@demo.ch"

Add-DistributionGroupMember -Identity "GRP-Finances-Purview" -Member "finance2@demo.ch"

### Étape 5 — Vérifier et se déconnecter

Get-DistributionGroupMember "GRP-Direction-Purview"

Get-DistributionGroupMember "GRP-RH-Purview"

Get-DistributionGroupMember "GRP-Finances-Purview"

Disconnect-ExchangeOnline -Confirm:$false

> **✅ VÉRIFICATION DANS LE PORTAIL** Allez dans Entra ID (entra.microsoft.com) > Groupes. Les 3 groupes GRP-*-Purview doivent apparaître avec une icône enveloppe (mail-enabled). Si vous ne les voyez pas dans l'instant, attendez 2 à 5 minutes, la synchronisation prend un peu de temps.

---

[← Préparation — Création du tenant Microsoft 365](prep-01-creation-tenant.html) | [Préparation — Création des comptes utilisateurs →](prep-03-comptes-utilisateurs.html)
