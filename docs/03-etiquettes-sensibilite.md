---
title: "Partie 2 — Les Étiquettes de sensibilité (Labels) | Configuration Microsoft Purview 2026"
description: "Comprendre et créer les étiquettes de sensibilité Purview : groupes de sécurité mail-enabled, SIT personnalisés suisses, création des 7 étiquettes, chiffrement Azure RMS, publication, procédure Super User RMS (break-glass) et gestion du stock existant."
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

# Partie 2 — Les Étiquettes de sensibilité (Labels)

## 2.1 Comprendre les étiquettes

> 💡 **PÉRIMÈTRE DIRECTION : Pas d’auto-labelling. **Les espaces Direction ne contiennent pas de types de données détectables de manière fiable (AVS, IBAN, données médicales). La sensibilité repose sur le contexte organisationnel (Direction / Board) et la nature stratégique des contenus. Aucune stratégie d’étiquetage automatique n’est créée pour ce périmètre. L’étiquette « 3a Direction Site » est appliquée manuellement lors de la création du site ; l’étiquette « 3b Direction Documents » est appliquée manuellement par les membres de la Direction.

Une étiquette de sensibilité est un marqueur qui s'attache à un document ou un email et lui applique des règles de protection. Une fois appliquée, l'étiquette suit le document partout, même si l'utilisateur l'envoie par email ou le télécharge sur son PC.

> ✅ **CONSEIL : **Les étiquettes de sensibilité permettent de définir des règles de protection. Lorsque le chiffrement est activé, il est assuré par Azure Rights Management (RMS). Un document ainsi protégé est illisible même si quelqu’un le vole physiquement. Les étiquettes configurent les politiques de protection, mais le chiffrement est réalisé par le service RMS sous-jacent.

## 2.2 Prérequis : Créer les groupes de sécurité mail-enabled

Deux méthodes coexistent selon votre contexte. **En environnement hybride (AD on-prem + Entra Connect) **il faut créer les groupes mail-enabled dans l’Active Directory local, puis les synchroniser vers Entra ID via Entra Connect. **En tenant cloud-only (sans AD on-prem) : **créer les groupes mail-enabled directement dans Exchange Online via PowerShell. Le guide décrit les deux cas, utilisez uniquement la méthode correspondant à votre environnement.

Le moteur de chiffrement de Purview (Azure Rights Management) exige que les groupes assignés aux étiquettes aient une adresse email, on parle de groupes de sécurité mail-enabled. Vos groupes GRP-Direction, GRP-RH et GRP-Finances sont des groupes de sécurité simples sans email, ils ne peuvent pas être utilisés pour le chiffrement des étiquettes 3, 4 et 5. Pour les étiquettes utilisant le chiffrement avec attribution explicite des autorisations, seuls les utilisateurs et les groupes disposant d’une adresse email (groupes mail-enabled) peuvent être utilisés. Cette contrainte est liée au fonctionnement d’Azure Rights Management.

> ⚠️ **ATTENTION :** En mode hybride, les groupes Entra ID existants ne peuvent pas être transformés en mail-enabled, il n’existe aucune commande de conversion. Les groupes doivent être créés directement dans l’Active Directory local avec l’attribut mail renseigné. La gestion des membres doit se faire exclusivement dans l’AD local pas via Exchange Online ni via Entra ID. ⚠️ Dans un tenant purement MS 365 les manipulations suivantes peuvent être effectuées directement via le portail d’Exchange Online ou PowerShell.

| **Type de groupe** | **Mail-enabled** | **Chiffrement Purview** | **Remarque** |
| --- | --- | --- | --- |
| Groupe de sécurité simple (Entra ID) | ❌ Non | ❌ Non | Vos GRP-RH, GRP-Direction etc. |
| **Groupe sécurité mail-enabled (Exchange)** | ✅ Oui | ✅ Oui | Type MailUniversalSecurityGroup, créé dans l’AD local, synchronisé via Entra Connect |
| Groupe Microsoft 365 | ✅ Oui | ✅ Oui | Créé automatiquement avec SharePoint |

### 2.2.1 Créer les 3 groupes dans l’Active Directory local

Effectuez les opérations suivantes dans la console Utilisateurs et ordinateurs Active Directory (ou via PowerShell AD local). Remplacez demo.ch par votre domaine réel.

**Étape 1 : Créer le groupe dans l’AD local :**

New-ADGroup -Name "GRP-Direction-Purview" -GroupScope Universal -GroupCategory Security -Path "OU=Groupes,DC=demo,DC=ch"

*→ Adaptez le chemin OU à la structure de votre AD. Répétez les mêmes commandes pour GRP-RH-Purview et GRP-Finances-Purview.*

**Étape 2 : Renseigner l’attribut mail sur chaque groupe :**

Set-ADGroup -Identity "GRP-Direction-Purview" -Replace @{mail="grp-direction-purview@demo.ch"}

Set-ADGroup -Identity "GRP-RH-Purview" -Replace @{mail="grp-rh-purview@demo.ch"}

Set-ADGroup -Identity "GRP-Finances-Purview" -Replace @{mail="grp-finances-purview@demo.ch"}

*→ Résultat attendu après le prochain cycle de synchronisation Entra Connect (délai : 15 à 30 minutes) : 3 groupes de type MailUniversalSecurityGroup visibles dans Entra ID avec leur adresse email. Aucun site SharePoint ni équipe Teams associé.*

### 2.2.2 Ajouter les membres et vérifier

La gestion des membres doit se faire exclusivement dans l’Active Directory local. Tout ajout ou retrait effectué dans l’AD sera synchronisé automatiquement vers Entra ID par Entra Connect au prochain cycle. Modifier les membres dans Exchange Online ou Entra ID provoquerait une erreur d’écriture ou une désynchronisation au prochain cycle de synchronisation.

> ℹ️ **PRÉREQUIS UTILISATEURS : **Chaque employé devant accéder aux documents chiffrés (étiquettes 3, 4, 5) doit posséder l’attribut mail renseigné dans l’AD local : Set-ADUser -Identity "prenom.nom" -EmailAddress "prenom.nom@demo.ch". Azure RMS identifie les utilisateurs via leur adresse email, sans cet attribut, l’utilisateur est invisible pour le moteur de chiffrement et ne pourra pas ouvrir les documents protégés, même s’il est membre du groupe de sécurité. Dans un tenant purement Microsoft 365 (sans AD on-prem), ce prérequis est automatiquement satisfait car chaque compte M365 possède nativement une adresse email.

Add-ADGroupMember -Identity "GRP-Direction-Purview" -Members utilisateur

`# Remplacez "utilisateur" par le sAMAccountName AD de la personne`

`# Répétez pour GRP-RH-Purview et GRP-Finances-Purview`

**Vérification : si les membres apparaissent, Purview les reconnaîtra :**

Get-ADGroupMember -Identity "GRP-Direction-Purview" | Select Name

*→ Résultat attendu : liste des membres avec leur nom. Vérifiez aussi dans le portail Entra ID → Groupes → GRP-Direction-Purview → Membres que la synchronisation est effective (délai : 15 à 30 minutes après un cycle Entra Connect).*

> ✅ **CONSEIL : **Ces 3 groupes (GRP-Direction-Purview, GRP-RH-Purview, GRP-Finances-Purview), créés dans l’AD local et synchronisés via Entra Connect sont ceux à utiliser à l’étape 6 de la section 2.4 lors de la configuration du chiffrement des étiquettes 3, 4 et 5. Les étiquettes 1 (Public), 2 (Interne) et 6 (Transmission-Externe) ne nécessitent pas de groupes mail-enabled car elles n’ont pas de chiffrement.

### 2.2.3 Alternative tenant cloud pur : Créer les groupes via Entra ID / Exchange Online

Dans un tenant purement Microsoft 365 sans Active Directory on-prem, les groupes mail-enabled sont créés directement dans Exchange Online via PowerShell. Les noms de groupes sont identiques à ceux utilisés dans la procédure hybride (GRP-Direction-Purview, GRP-RH-Purview, GRP-Finances-Purview) pour assurer la cohérence de la configuration.

**Étape 1 : Se connecter à Exchange Online :**

Connect-ExchangeOnline

*→ Une fenêtre de connexion Microsoft s’ouvre. Connectez-vous avec votre compte admin cela prend un peu de temps c’est normal*
*. *

**Étape 2 : Créer les 3 groupes mail-enabled :**

New-DistributionGroup -Name "GRP-Direction-Purview" -Alias "grp-direction-purview" -Type Security -PrimarySmtpAddress "grp-direction-purview@demo.ch"

New-DistributionGroup -Name "GRP-RH-Purview" -Alias "grp-rh-purview" -Type Security -PrimarySmtpAddress "grp-rh-purview@demo.ch"

New-DistributionGroup -Name "GRP-Finances-Purview" -Alias "grp-finances-purview" -Type Security -PrimarySmtpAddress "grp-finances-purview@demo.ch"

*→ Résultat attendu : 3 groupes de type MailUniversalSecurityGroup immédiatement visibles dans Entra ID avec leur adresse email. Aucun cycle de synchronisation requis.*

**Étape 3 : Ajouter les membres :**

Add-DistributionGroupMember -Identity "GRP-RH-Purview" -Member utilisateur@demo.ch

*→ Dans un tenant cloud pur, la gestion des membres se fait exclusivement via Exchange Online ou le portail d’administration Exchange. Remplacez l’identité du groupe et l’adresse du membre. Répétez pour chaque membre et chaque groupe.*

**Étape 4 : Vérification :**

Get-DistributionGroupMember "GRP-RH-Purview"

*→ Les membres apparaissent. Purgez la session : Disconnect-ExchangeOnline -**Confirm:$**false. Utilisez ensuite ces groupes à l’étape 6 de la section 2.4.*

## 2.3 Accéder à la section Étiquettes

1. Allez sur https://compliance.microsoft.com

2. Dans le menu de gauche, cherchez « Protection des informations » cliquez dessus.

3. Cliquez sur l'onglet Étiquettes dans la barre horizontale en haut.

4. Vous voyez la liste vide des étiquettes (normal au départ).

> ⚠️ **ATTENTION : **Si vous ne voyez pas 'Protection des informations' dans le menu gauche, votre rôle Compliance Administrator Purview n'est pas encore actif. Déconnectez-vous, reconnectez-vous, puis réessayez.

2.3bis : Créer les SIT personnalisés (à faire avant les étiquettes)

Les SIT (Sensitive Information Types) personnalisés définissent les motifs de détection utilisés par les étiquettes, les règles DLP et l’auto-labelling. Ils doivent être créés avant la configuration des étiquettes et des politiques, afin que les procédures suivantes puissent les référencer directement. La propagation après création peut prendre 15 à 30 minutes, attendez avant de configurer les règles qui les utilisent.

ℹ️ **Terminologie SIT : **dans ce guide, SIT (Sensitive Information Type) désigne ce que l’interface Purview appelle Types d’informations sensibles ou Types d’infos confidentielles selon l’endroit. Les deux termes désignent exactement la même chose. Quand le guide dit « sélectionnez AVS-Suisse-PME dans les SITs », cherchez + Ajouter → Types d’informations sensibles dans l’interface.

SIT 1 : AVS-Suisse-PME

Détecte les numéros AVS suisses au format exact 756.XXXX.XXXX.XX en 4 langues nationales + anglais. Complémentaire au SIT natif Swiss Social Security Number AHV : AVS-Suisse-PME ne valide pas le checksum EAN-13 et détecte les numéros AVS sur la base du format uniquement. Le SIT natif valide le checksum et rate donc les numéros dont le checksum est incorrect, ce qui peut arriver dans des documents de test ou des fichiers légèrement malformés. Les deux SITs sont complémentaires. Regex : 756\\\.\\d{4}\\\.\\d{4}\\\.\\d{2} | Niveau de confiance : Moyen | min : 1.

| **N°** | **Étape** | **Action** |
| --- | --- | --- |
| **1** | **Navigation** | Purview → Protection des données → Classifieurs → Types d’informations sensibles → **+ Créer un type d’informations sensibles** |
| **2** | **Page 1 : Nommer** | Nom : **AVS-Suisse-PME** │ Description : Détection numéro AVS suisse 756.XXXX.XXXX.XX, 4 langues nationales + anglais. → **Suivant** |
| **3** | **Page 2 : Définir les modèles** | + Ajouter un modèle → Niveau de confiance : **Moyen** → + Ajouter un élément principal → Expression régulière → ID : **regex-AVS-Suisse** → collez : **756\\.\\d{****4}\\.\\d****{****4}\\.\\d****{2}** → Correspondance de chaîne → **Terminé** |
| **4** | **Page 2 : Ajouter les mots-clés** | + Ajouter des éléments de soutien → Liste de mots-clés → ID : **recherche-AVS-Suisse** → saisissez un mot-clé par ligne (voir Annexe C pour la liste complète en 4 langues) → **Terminé** |
| **5** | **Page 3 : Tester et créer** | Suivant → testez avec un numéro format 756.XXXX.XXXX.XX → vérifiez la détection → Suivant → **Créer** |
| **⚠️** | **Propagation 15 à 30 minutes : **attendez avant de configurer les règles qui utilisent ce SIT. AVS-Suisse-PME ne valide pas le checksum EAN-13 et détecte les numéros AVS par format uniquement. Le SIT natif Swiss Social Security Number AHV valide le checksum et rate les numéros dont le checksum est incorrect. Les deux sont complémentaires : conservez-les en configuration OU pendant la période de propagation. |

SIT 2 : Medical-RH-Suisse

Détecte les données médicales RH (arrêts maladie, certificats, incapacité de travail) en 4 langues nationales + anglais. Plus ciblé que All Medical Terms And Conditions qui génère des faux positifs sur des termes médicaux génériques sans contexte RH.

| **N°** | **Étape** | **Action** |
| --- | --- | --- |
| **1** | **Navigation** | Purview → Protection des données → Classifieurs → Types d’informations sensibles → **+ Créer un type d’informations sensibles** |
| **2** | **Page 1 : Nommer** | Nom : **Medical-RH-Suisse** │ Description : Détection données médicales RH, 4 langues nationales + anglais. → **Suivant** |
| **3** | **Page 2 : Définir les modèles** | + Ajouter un modèle → Niveau de confiance : **Moyen** → + Ajouter un élément principal → Liste de mots-clés → ID : **medical-rh-suisse** → saisissez un mot-clé par ligne (voir Annexe C.5.1 pour la liste complète) → **Terminé** |
| **4** | **Page 3 : Tester et créer** | Suivant → testez avec « arrêt maladie » → vérifiez la détection → Suivant → **Créer** │ Attendez 15 à 30 minutes avant de configurer les règles des sections suivantes. |
| **ℹ️** | **Medical-RH-Suisse vs SIT natif : **Medical-RH-Suisse est ciblé sur le contexte RH suisse (arrêts maladie, certificats, incapacité de travail) en 4 langues nationales. Le SIT natif All Medical Terms And Conditions génère des faux positifs sur des termes médicaux génériques sans contexte RH. Ne pas substituer l’un à l’autre. |

## 2.4 Créer les 7 étiquettes de sensibilité

Vous allez créer 7 étiquettes dans cet ordre exact, du moins sensible au plus sensible. Créez-les dans l'ordre 1 à 7.

| **Ordre** | **Nom étiquette** | **Couleur** | **Chiffrement** | **Qui peut appliquer** | **Qui peut lire** |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 - Public | Vert | Non | Tous | Tous |
| 2 | 2 - Interne | Bleu | Non | Tous | Internes uniquement |
| 3 | 6 - Transmission-Externe | Orange | Non | Tous (label par défaut bibliothèque) | Destinataires externes (pas de chiffrement, impression autorisée) |
| 4-5 | 3a - Direction - Site (conteneur) 3b - Direction - Documents (fichiers) | Violet | Oui | GRP-Direction-Purview + Admin | Direction + Admin |
| 6 | 4 - RH-Confidentiel | Bordeaux | Oui | GRP-RH-Purview + GRP-Direction-Purview + Admin | RH + Direction + Admin |
| 7 | 5 - Finances-Confidentiel | Orange | Oui | GRP-Finances-Purview + GRP-Direction-Purview + Admin | Finances + Direction + Admin |

Procédure détaillée : Créer l'étiquette RH-Confidentiel (étiquette 4)

Suivez ces étapes exactement. L’annexe visuelle illustre chaque étape par capture d’écran. Pour les étiquettes 1 - Public, 2 - Interne et 5 - Finances-Confidentiel, répétez la même procédure en adaptant le nom, la couleur et les paramètres de protection selon le tableau récapitulatif ci-dessus. Les étiquettes 3a et 3b disposent d’une procédure détaillée aux sous-sections suivantes.

| **N°** | **Étape** | **Action** |
| --- | --- | --- |
| **1** | **Navigation** | Purview → Protection des données → Étiquettes de confidentialité → **+ Créer une étiquette** |
| **2** | **Page 1 : Nommer** | Nom : **4 - RH-Confidentiel** │ Description utilisateurs : Documents RH confidentiels, contrats, salaires, évaluations, données personnelles. │ Description admin : Label RH avec chiffrement AES-256, accès restreint GRP-RH-Purview et GRP-Direction-Purview. │ Couleur : **Bordeaux** → **Suivant** |
| **3** | **Page 2 : Définir l’étendue** | Cochez les 3 cases : **Fichiers et autres ressources de données**, **Emails**, **Réunions** → **Suivant** |
| **4** | **Page 3 : Choisir la protection** | Sélectionnez **Contrôler l’accès** et **Appliquer le marquage de contenu** → **Suivant** |
| **5** | **Sous-page : Contrôle de l’accès** | Chiffrement : **Configurer les paramètres de contrôle d’accès** │ Attribuer : **Attribuer des autorisations maintenant** │ Expiration : **Jamais** │ Hors connexion : **30 jours** │ Cliquez sur **+ Attribuer des autorisations** |
| **⚠️** | **RAPPEL POC : **l’ajout du compte admin aux autorisations est un choix de test (voir section 0.3). En production, limitez l’accès au rôle Super User de RMS, activé uniquement pour les urgences. |
| **6** | **Sous-page : Attribuer des autorisations** | Dans le panneau, cliquez **+ Ajouter des utilisateurs et des groupes** → sélectionnez : **GRP-RH-Purview**, **GRP-Direction-Purview**, **admin@demo.ch** (Azure RMS ne donne aucun accès implicite à l’admin, sans cette étape l’admin ne peut pas ouvrir les documents chiffrés). |
| **7** | **Sous-page : Niveau et enregistrement** | Niveau d’autorisation : **Éditeur** (Co-auteur : lecture et modification, pas suppression). Cliquez sur **Enregistrer** |
| **8** | **Sous-page : Vérifier le contrôle d’accès** | Le résumé affiche : **GRP-RH-Purview + GRP-Direction-Purview + admin@demo.ch**, niveau Co-auteur, 30 jours hors ligne. → **Suivant** |
| **9** | **Sous-page : Marquage, filigrane** | Activez le bouton **Marquage du contenu** │ Cochez **Ajouter un filigrane** → texte : « CONFIDENTIEL RH » → taille : 40 → diagonal → couleur : Rouge. |
| **10** | **Sous-page : Marquage, en-tête** | Cochez **Ajouter un en-tête** → texte : « RH CONFIDENTIEL Usage interne uniquement » → couleur : Rouge → taille : 11 → Centré. |
| **11** | **Sous-page : Marquage, pied de page** | Cochez **Ajouter un pied de page** → texte : « Document protégé nLPD, Ne pas diffuser » → couleur : Rouge → taille : 11 → Centré. → **Suivant** |
| **12** | **Page 6 : Auto-labelling, activer** | Activez le toggle **Étiquetage automatique pour les fichiers et les e-mails** │ Cliquez sur **+ Ajouter une condition** |
| **13** | **Auto-labelling : créer le groupe** | Choisissez **Le contenu contient** │ Nom du groupe : **RH-Donnees-Sensibles** │ Opérateur : **N’importe lequel de ces éléments (OU)** : les deux conditions déclenchent indépendamment l’étiquette. |
| **14** | **Auto-labelling : Condition 1 (AVS)** | Cliquez **+ Ajouter → Types d’infos confidentielles** → tapez et sélectionnez **AVS-Suisse-PME** → Confiance : Moyenne → min : 1. |
| **⚠️** | **SITs personnalisés : **AVS-Suisse-PME ne valide pas le checksum EAN-13 et détecte les numéros AVS par format uniquement. Le SIT natif Swiss Social Security Number AHV valide le checksum et rate les numéros dont le checksum est incorrect. Les deux sont complémentaires, conservez-les en configuration OU pendant la période de propagation. Ne pas ajouter IBAN ici : l’IBAN déclenche l’étiquette 5 - Finances-Confidentiel (section 3.2). |
| **15** | **Auto-labelling : Condition 2 (Médical)** | Cliquez **+ Créer un groupe** → opérateur : OU │ Cliquez **+ Ajouter → Types d’infos confidentielles** → tapez et sélectionnez **Medical-RH-Suisse** → Confiance : Moyenne → min : 2. |
| **16** | **Auto-labelling : valider** | Vérifiez : **RH-Donnees-Sensibles** avec **AVS-Suisse-PME (min 1) OU Medical-RH-Suisse (min 2)**, Confiance Moyenne. Cliquez sur **Ajouter**, puis **Suivant** |
| **17** | **Page 7 : Groupes et sites** | **Laissez telle quelle.** → **Suivant** |
| **18** | **Page 8 : Ressources** | **Laissez telle quelle.** → **Suivant** |
| **19** | **Page 9 : Créer et terminer** | Vérifiez le résumé : Chiffrement = Activé, Marquage = Activé. Cliquez sur **Créer une étiquette** │ Sélectionnez **Ne créez pas encore de stratégie** → **Terminer** |
| **ℹ️** | **INFO : **Répétez les étapes 1 à 19 pour les étiquettes 1, 2 et 5. Étiquettes 1 et 2 : étape 4 choisissez Aucune protection, désactivez le marquage à l’étape 9. Étiquette 5 - Finances-Confidentiel : chiffrement identique (étiquette 4) avec GRP-Finances-Purview + GRP-Direction-Purview + admin@demo.ch, Co-auteur, 30 jours. Marquages : En-tête = « Finances - CONFIDENTIEL » │ Pied de page = « Document protégé nLPD, Ne pas diffuser » │ Filigrane = « CONFIDENTIEL FINANCES » (taille 40, diagonal, rouge). Limitation Excel : les marquages visuels ne s’affichent pas dans .xlsx, le chiffrement RMS et la détection DLP restent actifs. |

💡 **Logique de création des règles de conditions : **vous venez de configurer votre premier groupe de conditions avec l’opérateur OU et des SITs. Cette logique (groupe, opérateur, SIT, confiance, seuil minimum) est identique dans toutes les configurations suivantes de ce guide : stratégies d’auto-labelling (section 3.2), règles DLP (sections 4.2, 4.3, 4.4), Insider Risk Management (section 8.2) et Communication Compliance (section 8.4). Maîtrisez cette logique ici elle sera votre référence pour tout le reste du guide.

Procédure détaillée, créer l’étiquette : 3a - Direction - Site (conteneur)

Cette étiquette gouverne les conteneurs Direction (site SharePoint, équipe Teams, groupe Microsoft 365). Elle ne s’applique pas aux fichiers individuels. Son activation nécessite que la section 0.5 (EnableMIPLabels) ait été exécutée.

| **N°** | **Étape** | **Configuration exacte** |
| --- | --- | --- |
| **1** | **Lancer l’assistant** | Purview → Protection des données → Étiquettes de confidentialité → + Créer une étiquette |
| **2** | **Page 1 : Nommer** | Nom : 3a - Direction - Site │ Description utilisateur : Espace collaboratif strictement confidentiel réservé à la Direction (site SharePoint, Teams, groupe Microsoft 365). │ Description admin : Étiquette conteneur Direction - gouvernance accès et confidentialité. Aucun chiffrement fichier. → cliquer sur **Suivant.** |
| **3** | **Page 2 : Définir l’étendue** | **Cochez UNIQUEMENT : Groupes et sites. **Décochez : Fichiers et autres ressources de données, Courriers électroniques, Réunions. → cliquer sur **Suivant.** |
| **4** | **Page 3 : Éléments** | **Aucune action requise : **Les options Contrôler l’accès et Appliquer le marquage de contenu sont grisées et non disponibles pour l’étendue Groupes et sites. Cliquez simplement sur **Suivant.** |
| **5** | **Page 4 : Groupes et sites** | **Cochez les 3 cases : **Confidentialité et accès des utilisateurs externes │ Partage externe et accès conditionnel │ Découvrabilité des équipes privées et paramètres des canaux partagés. │ Appliquer un libellé aux réunions de canal : Aucun. → cliquer sur **Suivant.** |
| **6** | **Sous-page : Confidentialité et accès** | Confidentialité : sélectionnez Privé (seuls les propriétaires et membres peuvent accéder, les propriétaires peuvent ajouter des membres). │ Accès utilisateur externe : décochez Autoriser les propriétaires de groupes Microsoft 365 à ajouter des personnes externes en tant qu’invités. → cliquer sur **Suivant****.** |
| **7** | **Sous-page : Partage externe et accès conditionnel** | Contrôler le partage externe : cochez │ Contenu partagé avec : sélectionnez Uniquement les membres de votre organisation, aucun partage externe autorisé. │ Utiliser Accès conditionnel Microsoft Entra : cochez │ Option : Déterminer si les utilisateurs peuvent accéder depuis des appareils non gérés │ Accès : sélectionnez Autoriser un accès limité au web uniquement. → cliquer sur **Suivant.** |
| **8** | **Sous-page : Équipes privées et canaux partagés** | Détectabilité : décochez Autoriser les utilisateurs à découvrir les équipes privées, la Direction ne doit pas être découvrable dans les recherches. │ Canaux partagés Teams : cochez les 3 options : Interne uniquement + Même étiquette uniquement + Équipes privées uniquement. → cliquer sur **Suivant.** |
| **9** | **Enregistrer l’étiquette** | Vérifiez le résumé : Nom d’affichage = 3a - Direction - Site │ Portée = Site, UnifiedGroup │ Étiquetage automatique = Aucun │ Paramètres du groupe = Privé │ Paramètres du site = Accès limité au web + Membres org uniquement. │ Cliquez sur Enregistrer l’étiquette → Terminer. |

Procédure : Appliquer l’étiquette 3a au site SharePoint Direction existant

Une fois l’étiquette 3a créée et publiée, elle doit être appliquée manuellement au site SharePoint Direction existant. Trois méthodes sont disponibles selon votre contexte.

**Méthode 1 : Via le Centre d’administration SharePoint** (recommandée pour les administrateurs SharePoint) : admin.microsoft.com → Centres d’administration → SharePoint → Sites actifs → cliquez sur le site Direction → onglet Stratégies → Étiquette de confidentialité → Modifier → sélectionnez : 3a – Direction - Site → Enregistrer. Les paramètres de confidentialité et de partage externe sont appliqués immédiatement.

**Méthode 2 : Via le portail Microsoft Entra** (si le site est associé à un groupe Microsoft 365) : entra.microsoft.com → Groupes → Tous les groupes → cherchez le groupe lié au site Direction → Propriétés → Étiquette de confidentialité → sélectionnez : 3a - Direction - Site → Enregistrer. C’est la méthode que vous avez utilisée elle est valide et produit le même résultat.

**Méthode 3 — Via PowerShell** (pour plusieurs sites simultanément) :

Connect-SPOService -Url https://demo-admin.sharepoint.com

`# Récupérer l'ID de l'étiquette 3a depuis Purview`

Set-SPOSite -Identity https://demo.sharepoint.com/sites/Direction -SensitivityLabel "GUID-de-l-etiquette-3a"

**ℹ️**** INFO — **Le GUID de l’étiquette se trouve dans Purview → Protection des données → Étiquettes de confidentialité → cliquez sur : « 3a - Direction – Site » → onglet Propriétés → copiez l’ID de l’étiquette. Répétez la commande Set-SPOSite pour chaque site à étiqueter (RH, Finances, Équipe IT).

Procédure détaillée, créer l’étiquette : 3b - Direction - Documents (fichiers)

Cette étiquette protège et marque visuellement les documents Direction lorsqu’ils sortent du site (partage externe, email, téléchargement). ⚠️Application manuelle par les membres de la Direction uniquement, aucune détection automatique.

| **N°** | **Étape** | **Configuration exacte** |
| --- | --- | --- |
| **1** | **Lancer l’assistant** | Purview → Protection des données → Étiquettes de confidentialité → + Créer une étiquette |
| **2** | **Page 1 : Nommer** | Nom : 3b - Direction - Documents │ Description utilisateur : Document strictement confidentiel émanant de la Direction. Application manuelle. │ Description admin : Étiquette fichier Direction avec chiffrement AES-256, accès GRP-Direction-Purview. → cliquer sur **Suivant.** |
| **3** | **Page 2 : Définir l’étendue** | Cochez : Fichiers et autres ressources de données et Courriers. │ Décochez : Réunions et Groupes et sites. → **Suivant.** |
| **4** | **Page 3 : Choisir la protection** | Sélectionnez Contrôler l’accès et Appliquer le marquage de contenu. │ cliquer sur **Suivant.** |
| **5** | **Page 4 : Configurer le chiffrement** | Chiffrement : Configurer les paramètres de contrôle d’accès (chiffrement) │ Attribuer des autorisations maintenant │ Expiration : Jamais │ Accès hors connexion : 30 jours │ + Attribuer des autorisations → ajoutez GRP-Direction-Purview et admin@demo.ch │ Niveau : Éditeur (Co-auteur) │ Enregistrer → cliquer sur **Suivant.** |
| **6** | **Page 5 : Marquage du contenu** | Activez Marquage du contenu. │ En-tête : « Direction - Strictement confidentiel » • Rouge • taille 11 • Centré. │ Pied de page : « Document interne - Diffusion interdite sans validation Direction » • Rouge • taille 11 • Centré. │ Filigrane : « CONFIDENTIEL – DIRECTION » • taille 40 • diagonal • Rouge. → **Suivant.** |
| **7** | **Sous-page : Étiquetage automatique des fichiers et des courriers** | Laissez désactivé**. **Laissez le bouton désactivé. Le périmètre Direction ne contient pas de données détectables automatiquement, application manuelle uniquement (voir section 3.1). → **Suivant.** |
| **8** | **Page 4 : Groupes et sites** | Aucune case à cocher.** **L’étiquette 3b étant de type Fichiers uniquement, les paramètres Groupes et sites ne s’appliquent pas. Laissez toutes les cases décochées et le libellé réunion de canal sur Aucun. → **Suivant.** |
| **9** | **Vérifier et créer** | Vérifiez le résumé : Nom = 3b - Direction - Documents │ Portée = Fichiers et autres ressources de données, E-mail │ Contrôle de l’accès = Activé (GRP-Direction-Purview + admin@demo.ch, Co-auteur, 30 jours hors ligne) │ Marquage = Activé (filigrane + en-tête + pied de page) │ Étiquetage automatique = Aucun. → Cliquez sur **Enregistrer l’étiquette.** |

Étiquette 6 : Transmission-Externe

Rôle : étiquette de transit sans chiffrement. Sa protection repose sur les marquages visuels, les permissions SharePoint du site Transmission externe, la politique DLP-Protection-Transmission-Externe et la rétention de 2 jours. Cette étiquette permet aux destinataires externes d’imprimer et de télécharger les documents.

| **N°** | **Étape** | **Action** |
| --- | --- | --- |
| **1** | **Navigation** | Purview → Protection des données → Étiquettes de confidentialité → **+ Créer une étiquette** |
| **2** | **Page 1 : Détails** | Nom : **6 - Transmission-Externe** │ Description utilisateurs : Document préparé pour transmission externe. Les données sensibles ont été vérifiées et approuvées pour partage par un responsable autorisé. │ Description admin : Étiquette transit sans chiffrement, marquages visuels seuls. │ Couleur : **Orange** → **Suivant** |
| **3** | **Page 2 : Portée** | Cochez **Fichiers et autres ressources de données** et **Groupes et sites** │ Décochez Courriers et Réunions → **Suivant** |
| **⚠️** | **Groupes et sites obligatoire : **sans cette case cochée, l’étiquette n’apparaît pas dans le menu SharePoint Admin Center et ne peut pas être assignée comme label par défaut de bibliothèque. |
| **4** | **Page 3 : Paramètres de protection** | Cochez uniquement **Appliquer le marquage de contenu** │ Ne cochez pas Contrôler l’accès (pas de chiffrement) → **Suivant** |
| **5** | **Sous-page : Marquage, filigrane** | Activez le bouton **Marquage du contenu** │ Cochez **Ajouter un filigrane** → texte : « TRANSMISSION EXTERNE » → taille : 40 → diagonal → couleur : Rouge. |
| **6** | **Sous-page : Marquage, en-tête** | Cochez **Ajouter un en-tête** → texte : « Axonix SA - Document transmis à titre confidentiel » → taille : 11 → couleur : Rouge → Centré. |
| **7** | **Sous-page : Marquage, pied de page** | Cochez **Ajouter un pied de page** → texte : « Usage restreint au destinataire désigné - nLPD Art. 6 » → taille : 11 → couleur : Rouge → Centré. → **Suivant** |
| **8** | **Page 6 : Auto-labelling** | **Laissez le toggle désactivé.** L’application se fait via le label par défaut de bibliothèque SharePoint. → **Suivant** |
| **9** | **Page 7 : Groupes et sites** | Ne cochez rien. → **Suivant** |
| **10** | **Page 9 : Créer et terminer** | Cliquez sur **Créer une étiquette** │ Sélectionnez **Ne créez pas encore de stratégie** → **Terminé** │ L’étiquette sera incluse dans la politique de publication en section 2.5. |

Configurer le label par défaut sur le site Transmission externe

L’auto-labelling Purview est basé sur le contenu (AVS, IBAN) ; il n’est pas adapté ici. La méthode correcte est le label par défaut de bibliothèque SharePoint : tout fichier uploadé sans étiquette existante reçoit automatiquement l’étiquette 6 - Transmission-Externe.

**⚠️ Propagation pouvant atteindre 24 heures : **si l’étiquette n’apparaît pas dans le menu SharePoint Admin Center après 2 heures, vérifiez que la portée inclut bien Groupes et sites (condition obligatoire).

| **N°** | **Étape** | **Action** |
| --- | --- | --- |
| **1** | **SharePoint Admin Center** | https://demo-admin.sharepoint.com → Sites → Sites actifs → **Transmission externe** → onglet Paramètres → section Étiquette de confidentialité → sélectionnez **6 - Transmission-Externe** → **Enregistrer** |
| **✓** | Tout fichier déposé sans étiquette existante reçoit automatiquement l’étiquette 6 avec filigrane diagonal « TRANSMISSION EXTERNE » (rouge), en-tête « Axonix SA - Document transmis à titre confidentiel » et pied de page « Usage restreint au destinataire désigné - nLPD Art. 6 », sans chiffrement, impression disponible. |

## 2.5 Publier les étiquettes

Créer les étiquettes ne suffit pas. Il faut les publier pour qu’elles apparaissent dans Word, Excel, Outlook, PowerPoint et SharePoint. Sans publication, les étiquettes sont invisibles pour les utilisateurs.

| **N°** | **Étape** | **Action** |
| --- | --- | --- |
| **1** | **Accès** | Purview → Protection des données → Stratégies → Stratégies de publication d’étiquettes → **+ Publier l’étiquette** |
| **2** | **Page 1 : Étiquettes à publier** | Cliquez sur Sélectionnez les étiquettes de confidentialité à publier → cochez les 7 étiquettes : **1 - Public, 2 - Interne, 6 - Transmission-Externe, 3a - Direction - Site, 3b - Direction - Documents, 4 - RH-Confidentiel, 5 - Finances-Confidentiel** → Ajouter → **Suivant** |
| **3** | **Page 2 : Unités d’administration** | Laissez sur **Répertoire complet** → **Suivant** (Unités d’administration = licence E5, hors périmètre POC.) |
| **4** | **Page 3 : Utilisateurs et groupes** | Laissez sur **Tous utilisateurs et groupes** → **Suivant** |
| **5** | **Page 4 : Paramètres de stratégie** | Cochez les 2 options obligatoires : (1) **Les utilisateurs doivent fournir une justification pour supprimer une étiquette ou abaisser son niveau de classification** │ (2) **Obliger les utilisateurs à appliquer une étiquette à leurs courriers et documents** │ Décochez : Demander aux utilisateurs d’appliquer une étiquette à leur contenu Power BI. → **Suivant** |
| **6** | **Page 5 : Documents** | Étiquette par défaut : **2 - Interne** → **Suivant** |
| **7** | **Page 6 : E-mails** | Étiquette par défaut : **2 - Interne** │ Cochez : **Obliger les utilisateurs à appliquer une étiquette à leurs e-mails** │ Cochez : **E-mail hérite de l’étiquette de priorité la plus élevée des pièces jointes** → Appliquer automatiquement l’étiquette. → **Suivant** |
| **8** | **Page 7 : Réunions** | Étiquette par défaut : Aucune (POC) │ Cochez : **Hériter de l’étiquette des fichiers partagés lors des réunions** → Appliquer automatiquement une étiquette de priorité supérieure. → **Suivant** |
| **9** | **Page 8 : Sites et groupes** | Étiquette par défaut : Aucune │ Laissez décoché : Obliger les utilisateurs à appliquer une étiquette à leurs groupes et aux sites. → **Suivant** |
| **10** | **Page 9 : Structure et Power BI** | Étiquette par défaut : Aucune. → **Suivant** |
| **11** | **Page 10 : Nom de la stratégie** | Nom : **Politique-Labels-demo** │ Description : optionnelle. → **Suivant** |
| **12** | **Page 11 : Examiner et finaliser** | Vérifiez le résumé : 7 étiquettes publiées, paramètres obligatoires actifs, étiquette par défaut documents et e-mails = 2 - Interne. Cliquez sur **Envoyer** |
| **⚠️** | **POINT DE VÉRIFICATION CRITIQUE : **après publication, le résumé peut afficher « Courrier Exchange : Tous comptes » dans la section Publier pour des utilisateurs et groupes. Il ne s’agit pas d’une restriction à Exchange uniquement, c’est la présentation de l’étendue de déploiement. Vérifiez immédiatement que les étiquettes apparaissent dans Word et SharePoint (délai de propagation jusqu’à 24 heures). Si après 24 heures les étiquettes sont absentes des applications Office, revenez à l’étape 4 et vérifiez la configuration Utilisateurs et groupes. |

## 2.6 Procédure de secours, Super User RMS (Break-glass)

En cas d’urgence (départ conflictuel, perte d’accès aux groupes de sécurité, litige juridique ou corruption de la base des identités), l’administrateur doit pouvoir accéder au contenu chiffré sans passer par les autorisations standards des étiquettes. Le rôle Super User de RMS agit comme un passe-partout capable de lever le chiffrement de n’importe quel document du tenant, quelle que soit l’étiquette appliquée (3, 4 ou 5).

> ⚠️ **ATTENTION, PRÉCAUTIONS DE SÉCURITÉ (GOUVERNANCE) :** Compte Break-glass : utilisez un compte cloud-only (non synchronisé depuis l’AD local) pour éviter d’être bloqué par une panne d’Entra Connect. Protégez-le par une authentification MFA physique (FIDO2) et stockez les identifiants dans un coffre-fort physique. Toute activation du rôle Super User est enregistrée dans les journaux d’audit de Purview, la traçabilité est automatique. Désignez au minimum deux comptes Break-glass indépendants pour éviter le point de défaillance unique.

Procédure technique, activation via PowerShell :

**Attention, ne fonctionne PAS en Cloud Shell, à exécuter sur un poste en local sous Windows PowerShell 5.1 uniquement.**

**Prérequis : Installer le module AIPService :**

Install-Module -Name AIPService -Force

**Étape 1 : Se connecter au service de chiffrement :**

Connect-AipService

*→ Connectez-vous avec votre compte administrateur global.*

**Étape 2 : Activer la fonctionnalité Super User (désactivée par défaut) :**

Enable-AipServiceSuperUserFeature

**Étape 3 : Désigner le compte de secours :**

Add-AipServiceSuperUser -EmailAddress admin-secours@demo.ch

**Étape 4 : Vérifier l’activation :**

Get-AipServiceSuperUser

*→ Résultat attendu : le compte admin-secours@demo.ch apparaît dans la liste. La fonctionnalité est désactivée dès que l’urgence est résolue via *: Disable-AipServiceSuperUserFeature*.*
Disable-AipServiceSuperUserFeature

> ℹ️ **LIEN nLPD (Art. 8) : **L’Art. 8 nLPD impose des mesures techniques garantissant la disponibilité des données. Chiffrer sans procédure de récupération expose l’organisation à une perte d’intégrité définitive en cas de corruption du tenant ou de départ d’un administrateur clé. La procédure Break-glass garantit que l’entreprise reste propriétaire et maîtresse de ses données dans les situations les plus critiques. **Toute utilisation du rôle Super User doit faire l’objet d’une trace dans le registre des incidents.**

## 2.7 Gérer le stock de documents existants

L’auto-labelling détecte les patterns techniques (AVS, IBAN) mais ne couvre pas les documents confidentiels sans données structurées : contrats commerciaux, rapports de direction, présentations stratégiques, correspondances juridiques. En entreprise, ces documents représentent souvent la majorité du stock sensible.

**Niveau 1 : Étiquetage obligatoire (flux futur)** Déjà configuré.

Votre politique « Politique-Labels-demo » force chaque utilisateur à choisir une étiquette à l’ouverture de tout nouveau document. Le stock existant n’est pas couvert, mais tout document créé ou modifié après activation est maîtrisé immédiatement.

**Niveau 2 : Audit et campagne manuelle via Content Explorer**

Purview → Protection des informations → Content Explorer. Affiche les contenus étiquetés et/ou détectés par les classifieurs Purview. Il permet d’analyser la couverture d’étiquetage et de cibler des campagnes de rattrapage. Pour identifier et piloter la dette de labellisation (fichiers sans étiquette de sensibilité), utilisez l’approche combinée décrite en section 7.5, simulation auto-labellisation, inventaires SharePoint/OneDrive et gouvernance par emplacement.

**⚠️ PRÉREQUIS : Accès à Content Explorer bloqué par défaut. **Même un Global Admin ne peut pas voir le contenu de Content Explorer sans les deux rôles spécifiques ci-dessous. Sans ces rôles, un message "Autorisation requise" s’affiche et bloque tout accès. Ajoutez-les avant de continuer.

**Procédure : Ajouter les rôles Content Explorer :**

1. Purview → Paramètres (roue dentée en bas à gauche) → Rôles et étendues → Groupes de rôles.

2. Cherchez **Content Explorer List Viewer** (Afficheur de liste de l’Explorateur de contenus) → cliquez dessus → Modifier → Choisir des membres → ajoutez admin@demo.ch → **Enregistrer**. Ce rôle permet de voir la liste des fichiers et leur emplacement.

3. Revenez à la liste → cherchez **Content Explorer Content Viewer** (Visionneuse de contenu de l’Explorateur de contenus) → même procédure → ajoutez admin@demo.ch → **Enregistrer.** Ce rôle permet d’ouvrir et lire le contenu source de chaque fichier.

4. Attendez **15 à 30 minutes** avant de retourner dans Content Explorer, les rôles ne sont pas actifs immédiatement. Rafraîchissez la page après ce délai.

**ℹ️**** INFO — **Les deux rôles sont cumulatifs et indépendants : le premier donne accès à la liste, le second au contenu. Pour un auditeur externe qui doit vérifier les étiquettes sans voir le contenu des fichiers, assignez uniquement le premier rôle, c’est le principe de moindre privilège nLPD.

**Procédure d’audit recommandée :**

1. Purview → Protection des informations → Content Explorer.

2. Filtrez par site SharePoint (Direction, RH, Finances) → colonne Étiquette de sensibilité → triez sur Aucune.

3. Exportez la liste → transmettez au responsable de chaque département pour étiquetage manuel ciblé.

**⚠️ ATTENTION nLPD : **Tout document contenant des données personnelles identifiables (nom + coordonnées + fonction) est soumis à la nLPD art. 5, même sans AVS ni IBAN. Les contrats de prestation avec personnes physiques nominatives, les listes de clients, les correspondances juridiques entrent dans cette catégorie et doivent être étiquetés manuellement avec l'étiquette « 2 – Interne » au minimum. Si le document est à usage exclusif de la Direction, appliquez « 3b - Direction – Documents ».

**Niveau 3, Trainable Classifiers :** (M365 E5 uniquement, non disponible avec Business Premium + Purview Suite)

Microsoft propose des classifieurs entraînables capables de reconnaître des types de documents (contrats, factures, documents RH) sans pattern technique. Le modèle est entraîné sur 50 à 200 exemples de votre propre corpus documentaire. Une fois entraîné, il détecte et étiquette automatiquement les contrats de maintenance, les rapports de clôture, les évaluations, sans intervention humaine. C’est la solution industrielle pour les volumes importants (plusieurs milliers de documents).

**Approche recommandée pour une PME suisse**

| **Étape** | **Action** | **Délai estimé** |
| --- | --- | --- |
| Immédiat | Étiquetage obligatoire actif flux futur maîtrisé | **✅ Fait** |
| Semaines 1-2 | Content Explorer → identifier les documents non étiquetés par site | 1 jour |
| Semaines 2-4 | Campagne manuelle ciblée par département sur le stock critique | 2-3 jours |
| Phase 2 | Trainable Classifiers si volume > 5 000 documents (M365 E5 requis, non disponible avec Business Premium) | 2-4 semaines |

**ℹ️**** INFO : **Pour le POC, concentrez-vous sur les sites Direction, RH et Finances. Un document mal placé dans SharePoint (ex : contrat commercial dans le site Finances) ne sera jamais auto-étiqueté correctement. Le bon emplacement SharePoint est le premier niveau de gouvernance documentaire. Le bon site avant la bonne étiquette.

---

[← Partie 1 — Introduction et concepts Purview](02-introduction-concepts.html) | [Partie 3 — Auto-labelling (étiquetage automatique) →](04-auto-labelling.html)
