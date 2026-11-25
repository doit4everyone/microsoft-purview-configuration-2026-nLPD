---
title: "Partie 0 : Avant de commencer | Configuration Microsoft Purview 2026"
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

# Partie 0 : Avant de commencer

Lisez cette section entièrement avant de toucher quoi que ce soit dans Purview.

> 💡 **Tenant vierge ou de test :** si vous n'avez pas encore de tenant Microsoft 365 sur lequel travailler, le guide [Préparation du tenant : POC Purview](prep-00-vue-ensemble.html) explique comment créer et préparer un environnement de preuve de concept avant de suivre cette procédure (également disponible en .docx/.pdf sur la [release v1.0](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0)).

## 0.1 Comment accéder au portail Purview

1. **Ouvrir le navigateur** Utilisez Microsoft Edge ou Google Chrome. Évitez Firefox, certaines pages Purview ne s'affichent pas correctement.

2. **Aller sur l’URL** Tapez exactement : https://purview.microsoft.com. L’ancienne URL https://compliance.microsoft.com redirige automatiquement mais peut afficher une page intermédiaire selon le navigateur.

3. **Se connecter** Entrez votre compte administrateur. Si vous avez un compte admin@demo.ch ou admin@demo.onmicrosoft.com, utilisez celui qui a les droits admin M365.

4. **Ce que vous devez voir** Le tableau de bord Purview avec le menu à gauche. Si vous voyez une page blanche ou une erreur d'accès → la section 0.2 explique comment résoudre.

> ⚠️ **ATTENTION : **Si vous voyez 'Vous n'avez pas accès', ne paniquez pas. C'est un problème de rôles. Allez directement à la section 0.3 qui explique comment s'attribuer les bons rôles.

## 0.2 Comment ouvrir PowerShell correctement

Plusieurs étapes de cette procédure nécessitent PowerShell. Voici comment l'ouvrir correctement sous Windows 11 :

1. **Ouvrir PowerShell en administrateur** Cliquez sur le bouton Démarrer Windows → tapez PowerShell → clic droit sur Windows PowerShell → Exécuter en tant qu'administrateur → cliquez Oui. La fenêtre doit afficher 'Administrateur' dans le titre.

2. **Vérifier la version** Tapez : $PSVersionTable.PSVersion et appuyez sur Entrée. Vous devez avoir la version 5.1 minimum. Si vous avez PowerShell 7 c'est encore mieux.

3. **Vérifier la stratégie d'exécution (si nécessaire)** Tapez : Get-ExecutionPolicy -List et appuyez sur Entrée. Sur un poste Windows client, la valeur par défaut est Restricted : elle empêche l'exécution des scripts, y compris le chargement des modules PowerShell utilisés dans ce guide. Si la valeur effective est Restricted, tapez : Set-ExecutionPolicy RemoteSigned -Scope CurrentUser puis O et Entrée pour confirmer. Si la valeur est déjà RemoteSigned ou plus permissive, ne changez rien.

> ℹ️ **INFO : **Le périmètre CurrentUser ne concerne que votre compte et ne nécessite pas d'élévation administrateur (seul le périmètre LocalMachine l'exige). Si un message indique que le paramètre est remplacé par une stratégie définie à un niveau plus spécifique, une stratégie de groupe (lignes MachinePolicy ou UserPolicy dans Get-ExecutionPolicy -List) impose la valeur : adressez-vous à l'administrateur du poste plutôt que de chercher à la contourner. RemoteSigned exige que les scripts téléchargés depuis Internet soient signés : ne l'assouplissez pas davantage.
>
> Le mode administrateur de l'étape 1 reste utile pour les commandes Install-Module de ce guide qui n'indiquent pas -Scope CurrentUser : sous Windows PowerShell 5.1, elles installent le module pour tous les utilisateurs du poste.

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

3. Cherchez le groupe Compliance Management (Gestion de la conformité).

4. Cliquez dessus → onglet Membres → + Ajouter des membres → cherchez votre compte → **Ajouter**.

5. Attendez 5 à 10 minutes que le rôle se propage avant de continuer.

> 🚨 **IMPORTANT : **L'activation de l'audit via PowerShell exige le rôle Exchange 'Audit Logs'. Selon la documentation Microsoft, ce rôle est attribué par défaut aux groupes Compliance Management et Organization Management. Compliance Management suffit pour ce guide et respecte le principe du moindre privilège : Organization Management donne un accès administratif très large à Exchange Online et ne doit pas être attribué à un compte d'exploitation pour ce seul besoin. Sans le rôle Audit Logs, la commande d'activation de l'audit peut s'exécuter sans erreur apparente sans produire d'effet.

Rôles Purview RBAC via le portail Purview

1. Dans Purview (https://purview.microsoft.com) → menu de gauche → Paramètres (icône engrenage en bas) → Rôles et étendues → Groupes de rôles.

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

> ℹ️ **NOM COMMERCIAL : **dans ce guide, « Purview Suite » désigne l’offre **Microsoft Purview Suite for Microsoft 365 Business Premium**. Cette extension exige un abonnement Microsoft 365 Business Premium et est limitée à 300 licences par client. Elle ne doit pas être confondue avec l’extension Purview destinée aux environnements Microsoft 365 E3. Chaque utilisateur qui bénéficie des fonctionnalités Purview (administrateurs Purview et utilisateurs couverts par les politiques) doit disposer de cette licence.

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

> 🚨 **PRÉREQUIS GLOBAL : **Sans cette activation, les étiquettes de type Groupes et sites n’apparaissent pas dans l’interface lors de la création d’un site SharePoint ou d’une équipe Teams. Cette activation est globale au tenant et s’effectue normalement une seule fois. Elle est techniquement réversible (EnableMIPLabels remis à False), mais Microsoft précise qu’une désactivation a des conséquences : les paramètres des étiquettes ne sont plus appliqués aux sites et groupes, les étiquettes ne s’y affichent plus, et les conteneurs reviennent à l’ancienne propriété Classification d’Entra ID. Ne désactivez pas cette fonction une fois les étiquettes de conteneurs déployées.

Il faut distinguer deux objets : le **modèle** Group.Unified, fourni par Microsoft et toujours présent, et le **paramètre** Group.Unified, créé dans votre tenant à partir de ce modèle. Le paramètre peut ne pas exister, ou exister avec EnableMIPLabels à False. Le script ci-dessous traite les deux cas.

Étape 1 : Installer les modules et se connecter à Microsoft Graph

Les commandes Get-MgBetaDirectorySetting et New-MgBetaDirectorySetting appartiennent au module Microsoft Graph **Beta**, distinct du module Microsoft.Graph. Installez les deux sous-modules nécessaires :

```powershell
Install-Module Microsoft.Graph.Authentication -Scope CurrentUser
Install-Module Microsoft.Graph.Beta.Identity.DirectoryManagement -Scope CurrentUser
Connect-MgGraph -Scopes "Directory.ReadWrite.All"
```

> ⚠️ **SI LA FENÊTRE POWERSHELL SE FERME : **sous Windows PowerShell 5.1, Connect-MgGraph peut fermer brutalement la fenêtre au moment de l’authentification. Ce comportement a été constaté sur un poste où plusieurs versions des modules Microsoft Graph coexistaient (vérification : Get-Module -ListAvailable Microsoft.Graph* | Select-Object Name, Version). Rouvrez PowerShell et connectez-vous par code d’appareil, méthode validée sur le terrain : Connect-MgGraph -Scopes "Directory.ReadWrite.All" -UseDeviceCode. PowerShell affiche un code à saisir sur https://microsoft.com/devicelogin.
>
> Le mélange de versions peut aussi produire l’erreur « Impossible de charger le fichier ou l’assembly 'Microsoft.Graph.Authentication, Version=…' » : une commande issue d’un module installé dans une version plus ancienne exige une autre version du module Authentication que celle déjà chargée. Les modules Microsoft.Graph.Authentication et Microsoft.Graph.Beta.Identity.DirectoryManagement utilisés dans cette section doivent avoir la même version.
>
> Exécutez l’étape 5 (Connect-IPPSSession) dans une fenêtre PowerShell séparée, après Disconnect-MgGraph : charger les modules Graph et Exchange Online dans la même session peut provoquer des conflits.

Étape 2 : Rechercher le paramètre Group.Unified existant dans le tenant

```powershell
$setting = Get-MgBetaDirectorySetting -All | Where-Object { $_.DisplayName -eq "Group.Unified" }
$setting.Values | Format-List
```

*→ Si aucun résultat : le paramètre n’existe pas encore, exécutez l’étape 3a. Si un résultat apparaît : le paramètre existe, exécutez l’étape 3b, quelle que soit la valeur affichée pour EnableMIPLabels.*

Étape 3a : Le paramètre n’existe pas, le créer à partir du modèle

```powershell
$template = Get-MgBetaDirectorySettingTemplate -All | Where-Object { $_.DisplayName -eq "Group.Unified" }
$body = @{ templateId = $template.Id; values = @(@{ name = "EnableMIPLabels"; value = "True" }) }
New-MgBetaDirectorySetting -BodyParameter $body
```

Étape 3b : Le paramètre existe, mettre à jour EnableMIPLabels

Selon la documentation de l’API Microsoft Graph, une mise à jour de ce paramètre doit transmettre la collection complète des valeurs : envoyer uniquement EnableMIPLabels risquerait d’effacer les autres réglages de Group.Unified (création de groupes, invités, etc.). Le script ci-dessous sauvegarde d’abord les valeurs existantes, modifie uniquement EnableMIPLabels, puis renvoie l’ensemble :

```powershell
# Sauvegarde des valeurs actuelles (permet une restauration en cas de besoin)
$setting.Values | Select-Object Name, Value | Export-Csv "$env:USERPROFILE\Group.Unified-avant.csv" -NoTypeInformation -Encoding UTF8

# Reprise de toutes les valeurs existantes, modification de EnableMIPLabels uniquement
$values = @($setting.Values | ForEach-Object { @{ name = $_.Name; value = $_.Value } })
$mip = $values | Where-Object { $_.name -eq "EnableMIPLabels" }
if ($mip) { $mip.value = "True" } else { $values += @{ name = "EnableMIPLabels"; value = "True" } }

Update-MgBetaDirectorySetting -DirectorySettingId $setting.Id -BodyParameter @{ values = $values }
```

*→ Si New-MgBetaDirectorySetting renvoie une erreur Request_BadRequest, le paramètre existe déjà : relancez l’étape 2 puis exécutez l’étape 3b.*

Étape 4 : Vérification

```powershell
$setting = Get-MgBetaDirectorySetting -All | Where-Object { $_.DisplayName -eq "Group.Unified" }
$setting.Values | Where-Object { $_.Name -eq "EnableMIPLabels" }
```

*→ Résultat attendu : Name : EnableMIPLabels | Value : True. Si la valeur est encore False, relancez l’étape 3b. Si vous avez exécuté l’étape 3b, comparez aussi l’ensemble des valeurs avec la sauvegarde : seule EnableMIPLabels doit avoir changé.*

```powershell
$avant = Import-Csv "$env:USERPROFILE\Group.Unified-avant.csv"
$apres = (Get-MgBetaDirectorySetting -All | Where-Object { $_.DisplayName -eq "Group.Unified" }).Values | Select-Object Name, Value
Compare-Object $avant $apres -Property Name, Value
```

*→ Déconnectez-vous puis reconnectez-vous au portail Purview avant de continuer.*

Étape 5 : Synchroniser les étiquettes vers Entra ID

Microsoft indique que les étiquettes doivent aussi être synchronisées vers Entra ID. Cette commande s’exécute dans Security & Compliance PowerShell, fourni par le module ExchangeOnlineManagement (également utilisé en section 1.5) :

```powershell
Install-Module -Name ExchangeOnlineManagement -Scope CurrentUser
Connect-IPPSSession
Execute-AzureAdLabelSync
```

*→ La disponibilité des étiquettes de conteneurs peut prendre jusqu’à 24 heures après la synchronisation.*

## 0.6 Résumé état requis avant de continuer

| **✓** | **Prérequis** | **Comment vérifier** |
| --- | --- | --- |
| ☐ | Accès à https://purview.microsoft.com sans erreur | La page s'ouvre et affiche le menu Purview |
| ☐ | 3 rôles Entra ID attribués | entra.microsoft.com → Rôles → vos attributions |
| ☐ | Rôle Compliance Management Exchange (contient Audit Logs) | admin.exchange.microsoft.com → Rôles → Membres |
| ☐ | 5 groupes de rôles Purview attribués | Purview → Paramètres → Rôles → vos groupes |
| ☐ | Licence Purview Suite attribuée à votre compte | admin.microsoft.com → Utilisateurs → votre compte → Licences |
| ☐ | PowerShell ouvert en administrateur | Titre de la fenêtre contient 'Administrateur' |

---

[← Préface](00-preface.html) | [Partie 1 : Introduction et concepts Purview →](02-introduction-concepts.html)
