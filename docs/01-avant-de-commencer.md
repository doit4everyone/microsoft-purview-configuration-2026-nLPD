---
title: "Partie 0 — Avant de commencer | Configuration Microsoft Purview 2026"
description: "Prérequis avant de configurer Microsoft Purview : accès au portail, installation et connexion PowerShell Exchange Online, attribution des rôles administrateur, vérification de la licence Purview Suite et activation des étiquettes sur les conteneurs."
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

[← Retour à l’index](../)

# Partie 0 — Avant de commencer

Lisez cette section entièrement avant de toucher quoi que ce soit dans Purview.

> 💡 **Tenant vierge ou de test :** si vous n'avez pas encore de tenant Microsoft 365 sur lequel travailler, le guide [Préparation du tenant — POC Purview](prep-00-vue-ensemble.html) explique comment créer et préparer un environnement de preuve de concept avant de suivre cette procédure (également disponible en .docx/.pdf sur la [release v1.0](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0)).

## 0.1 Comment accéder au portail Purview

1. **Ouvrir le navigateur** Utilisez Microsoft Edge ou Google Chrome. Évitez Firefox, certaines pages Purview ne s'affichent pas correctement.

2. **Aller sur l'URL** Tapez exactement : https://compliance.microsoft.com, ne passez pas par admin.microsoft.com pour Purview, l’accès direct est plus fiable. Mise à jour 2026 : Microsoft a migré le portail vers https://purview.microsoft.com. L’ancienne URL redirige automatiquement mais peut afficher une page intermédiaire selon le navigateur. En cas de doute, utilisez directement https://purview.microsoft.com.

3. **Se connecter** Entrez votre compte administrateur. Si vous avez un compte admin@demo.ch ou admin@demo.onmicrosoft.com, utilisez celui qui a les droits admin M365.

4. **Ce que vous devez voir** Le tableau de bord Purview avec le menu à gauche. Si vous voyez une page blanche ou une erreur d'accès → la section 0.2 explique comment résoudre.

> ⚠️ **ATTENTION : **Si vous voyez 'Vous n'avez pas accès', ne paniquez pas. C'est un problème de rôles. Allez directement à la section 0.3 qui explique comment s'attribuer les bons rôles.

## 0.2 Comment ouvrir PowerShell correctement

Plusieurs étapes de cette procédure nécessitent PowerShell. Voici comment l'ouvrir correctement sous Windows 11 :

1. **Ouvrir PowerShell en administrateur** Cliquez sur le bouton Démarrer Windows → tapez PowerShell → clic droit sur Windows PowerShell → Exécuter en tant qu'administrateur → cliquez Oui. La fenêtre doit afficher 'Administrateur' dans le titre.

2. **Vérifier la version** Tapez : $PSVersionTable.PSVersion et appuyez sur Entrée. Vous devez avoir la version 5.1 minimum. Si vous avez PowerShell 7 c'est encore mieux.

3. **Autoriser l'exécution des scripts** Tapez : Set-ExecutionPolicy RemoteSigned -Scope CurrentUser et appuyez sur Entrée. Tapez O puis Entrée pour confirmer. Sans ça, les modules ne s'installent pas.

> 🚨 **IMPORTANT : **Si PowerShell affiche une erreur rouge sur Set-ExecutionPolicy, c'est que vous n'êtes pas en mode Administrateur. Fermez et rouvrez PowerShell en administrateur (clic droit → Exécuter en tant qu'administrateur).

**⚠️ Limites connues du dispositif**

Ce guide décrit une configuration robuste de Microsoft Purview, mais ne constitue pas une protection absolue. Il ne couvre pas :

1. Les erreurs ou abus d’un administrateur disposant de privilèges étendus. Un Global Admin peut contourner la quasi-totalité des contrôles Purview.

2. Les accès hors Microsoft 365, export local, copie d’écran, appareil non conforme ou non onboardé dans Intune.

3. L’ingénierie sociale ou la compromission d’un compte utilisateur, si un attaquant opère avec des identifiants légitimes, les contrôles DLP et IRM s’appliquent mais ne constituent pas une barrière absolue.

Ces risques doivent être traités par des mesures organisationnelles complémentaires. Séparation des rôles (voir Annexe H), formation des utilisateurs, politique de mots de passe, MFA, gestion des incidents. En production, le MFA doit être activé sur l’ensemble des comptes utilisateurs (rh1, finance1, direction1 et tous les comptes accédant aux données sensibles), dans ce POC, seul le compte administrateur en bénéficie. L’absence de MFA sur les comptes utilisateurs constitue un vecteur d’attaque par credential stuffing directement exposé aux données AVS et IBAN.

## 0.3 Comment s'attribuer les rôles nécessaires

Microsoft Purview utilise trois systèmes de rôles distincts : Entra ID, Exchange Online et Purview RBAC. Vous devez vous attribuer des rôles dans chacun des trois. Voici la procédure exacte pour chaque système.

Rôles Entra ID : via le portail Azure

1. Allez sur https://entra.microsoft.com et connectez-vous avec votre compte admin.

2. Dans le menu de gauche, cliquez sur Rôles et administrateurs.

3. Dans la barre de recherche, tapez le nom du rôle (ex : Administrateur de la sécurité).

4. Cliquez sur le rôle → + Ajouter des attributions → cherchez votre compte → Sélectionner → **Ajouter**.

5. Répétez pour chacun des 3 rôles Entra ID listés dans le tableau ci-dessous.

| **Rôle Entra ID à s'attribuer** | **Pourquoi c'est nécessaire** |
| --- | --- |
| Administrateur de la sécurité | Voir les signaux de risque Entra ID dans Purview |
| Administrateur de mise en conformité | Accéder aux rapports de conformité et politiques Purview |
| Administrateur d'IA | Voir les données Copilot et agents IA dans DSPM |

> ⚠️ **ATTENTION : **Après avoir ajouté un rôle Entra ID, déconnectez-vous du portail et reconnectez-vous. Les rôles ne prennent pas effet immédiatement dans la session en cours.

Rôle Exchange Online via le Centre d'administration Exchange

1. Allez sur https://admin.exchange.microsoft.com et connectez-vous.

2. Dans le menu de gauche, cliquez sur Rôles → Rôles d'administrateur.

3. Cherchez le groupe Organization Management (Gestion de l'organisation).

4. Cliquez dessus → onglet Membres → + Ajouter des membres → cherchez votre compte → **Ajouter**.

5. Attendez 5 à 10 minutes que le rôle se propage avant de continuer.

> 🚨 **IMPORTANT : **Organization Management est le seul rôle Exchange qui contient le droit 'Audit Logs'. Sans ce groupe, la commande PowerShell d'activation de l'audit s'exécute sans erreur mais ne fait rien.

Rôles Purview RBAC via le portail Purview

1. Dans Purview (https://compliance.microsoft.com) → menu de gauche → Paramètres (icône engrenage en bas) → Rôles et portées → Groupes de rôles.

2. Cherchez chaque groupe dans le tableau ci-dessous → cliquez dessus → Modifier le groupe de rôles → Ajouter des membres → votre compte → **Enregistrer.**

| **Groupe de rôles Purview** | **Ce que ça donne accès** |
| --- | --- |
| Compliance Administrator | Administration complète de Purview labels, DLP, politiques |
| Data Security AI Admins | Administration DSPM for AI surveillance Copilot et agents |
| Data Security Viewers | Lecture de tous les rapports risques données et IA |
| Content Explorer Content Viewer | Voir le contenu des fichiers dans Content Explorer |
| Content Explorer List Viewer | Voir la liste des fichiers dans Content Explorer |

> ⚠️ **ATTENTION : **Si vous ne trouvez pas ‘Data Security AI Admins’ dans la liste, c’est que Purview Suite n’est pas encore activé. Vérifiez dans admin.microsoft.com → Licences que Microsoft Purview Suite apparaît bien dans votre tenant.

## 0.4 Vérifier que Purview Suite est bien activé

Avant de continuer, confirmez que la licence est active sur votre compte :

1. Allez sur https://admin.microsoft.com → Utilisateurs → Utilisateurs actifs.

2. Cliquez sur votre compte administrateur.

3. Onglet Licences et applications.

4. Vérifiez que Microsoft Purview Suite est coché.

5. Si ce n'est pas coché → cochez-le et cliquez sur Enregistrer les modifications.

> ℹ️ **INFO : **La licence Purview Suite doit être attribuée à l’utilisateur qui va utiliser Purview, pas seulement achetée sur le tenant. Si vous ne la voyez pas dans la liste, elle n’a peut-être pas encore été achetée. Vérifiez dans Facturation → Vos produits.

> 💡 **NOTE DE MÉTHODOLOGIE - PREUVE DE CONCEPT (POC). **Ce document est conçu comme une preuve de concept pour un environnement de test. L’ajout explicite du compte Super-Admin (admin@demo.ch) aux autorisations de chiffrement (étiquettes 3, 4 et 5) est un choix pragmatique pour trois raisons : (1) Azure RMS n’accordant aucun droit implicite à l’administrateur, son ajout explicite garantit la visibilité technique lors des tests de validation du chiffrement, (2) la centralisation des tests sur un compte unique évite les changements de session constants lors des tests Copilot et DLP, (3) cela prévient tout verrouillage définitif des données de test en cas d’erreur de manipulation. En production, l’accès administrateur aux documents chiffrés doit être géré via le rôle "Super User" de RMS conformément au principe de séparation des tâches. ⚠️ IMPORTANT : Voir Annexe E (Différences POC VS Production), avant tout déploiement en environnement réel, supprimez le compte admin des autorisations de chiffrement et gérez l’accès via la procédure Break-glass (section 2.6) uniquement.

## 0.5 Activer les étiquettes sur les conteneurs (obligatoire)

Par défaut, Microsoft 365 n’active pas la fonctionnalité permettant d’appliquer des étiquettes de sensibilité aux conteneurs (sites SharePoint, équipes Teams, groupes Microsoft 365). Cette activation est globale au tenant et doit être effectuée une seule fois via PowerShell Microsoft Graph, avant toute création d’étiquette de type Groupes et sites.

> 🚨 **PRÉREQUIS GLOBAL : **Sans cette activation, les étiquettes de type Groupes et sites n’apparaissent pas dans l’interface lors de la création d’un site SharePoint ou d’une équipe Teams. L’opération est irreversible mais sans impact négatif sur les étiquettes existantes. À exécuter une seule fois par tenant.

Étape 1 : Connexion à Microsoft Graph

Install-Module Microsoft.Graph -Scope CurrentUser

Connect-MgGraph -Scopes "Directory.ReadWrite.All"

Étape 2 : Vérifier si le setting Group.Unified existe déjà

$template = Get-MgBetaDirectorySettingTemplate -All | Where-Object { $_.DisplayName -eq "Group.Unified" }

*→ Si aucun résultat : le setting n’existe pas encore, exécutez l’étape 3. Si un résultat apparaît : le setting existe déjà, passez directement à l’étape 4 pour vérifier la valeur.*

Étape 3 : Créer le setting avec EnableMIPLabels = True

$body = @{ templateId = $template.Id; values = @(@{ name = "EnableMIPLabels"; value = "True" }) }

New-MgBetaDirectorySetting -BodyParameter $body

Étape 4: Verification

$setting = Get-MgBetaDirectorySetting -All | Where-Object { $_.DisplayName -eq "Group.Unified" }

$setting.Values | Format-List

*→ Résultat attendu : Name : EnableMIPLabels | Value : True. Déconnectez-vous puis reconnectez-vous au portail Purview avant de continuer.*

## 0.6 Résumé état requis avant de continuer

| **✓** | **Prérequis** | **Comment vérifier** |
| --- | --- | --- |
| ☐ | Accès à https://compliance.microsoft.com sans erreur | La page s'ouvre et affiche le menu Purview |
| ☐ | 3 rôles Entra ID attribués | entra.microsoft.com → Rôles → vos attributions |
| ☐ | Rôle Organization Management Exchange | admin.exchange.microsoft.com → Rôles → Membres |
| ☐ | 5 groupes de rôles Purview attribués | Purview → Paramètres → Rôles → vos groupes |
| ☐ | Licence Purview Suite attribuée à votre compte | admin.microsoft.com → Utilisateurs → votre compte → Licences |
| ☐ | PowerShell ouvert en administrateur | Titre de la fenêtre contient 'Administrateur' |

---

[← Préface](00-preface.html) | [Partie 1 — Introduction et concepts Purview →](02-introduction-concepts.html)
