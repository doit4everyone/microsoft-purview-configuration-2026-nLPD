---
title: "Préparation — Création des sites SharePoint | Configuration Microsoft Purview 2026"
description: "Créer les sites SharePoint Intranet, Communication, Direction, RH, Finances et Transmission externe avec leurs permissions, y compris la configuration du partage externe sécurisé."
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

# Partie 4 — Création des sites SharePoint

Les sites SharePoint sont utilisés par le guide Purview pour tester les permissions, les étiquettes de sensibilité et la gouvernance des accès. Vous allez créer 5 sites avec des permissions précises.

Un sixième site nommé « Transmission externe » sera créé. Sa fonction sera décrite plus loin, après

la créations des sites standards.

| **Site** | **Type** | **URL** | **Qui peut accéder** |
| --- | --- | --- | --- |
| Intranet demo.ch | Hub Site (équipe) | sites/intranet | Tous les utilisateurs |
| Communication demo.ch | Communication | sites/communication | Tous les utilisateurs |
| Direction demo.ch | Équipe | sites/direction | GRP-Direction uniquement |
| RH demo.ch | Équipe | sites/RH | GRP-RH + GRP-Direction (lecture) |
| Finances demo.ch | Équipe | sites/Finances | GRP-Finances + GRP-Direction (lecture) |

## 4.1 Accéder au Centre d'administration SharePoint

1. Allez sur https://admin.microsoft.com.

2. Dans le menu gauche, cliquez sur Afficher tout > SharePoint.

3. Vous êtes dans le Centre d'administration SharePoint.

## 4.2 Créer le Hub Site : Intranet demo.ch

Le Hub Site est le site parent. Tous les autres sites s'y rattachent. C'est le premier à créer.

1. Cliquez sur Sites actifs > + Créer.

2. Choisissez Site de l'équipe.

3. Nom du site : Intranet demo.ch | Adresse : intranet | Langue : Français.

4. Cliquez sur Créer.

5. Une fois créé, dans la liste des sites, sélectionnez le site.

6. Cliquez sur les 3 points (…) > Hub > Inscrire comme hub site.

7. Nom du hub : Intranet demo.ch. Cliquez sur Enregistrer.

> **✅ QU'EST-CE QU'UN HUB SITE ?** Un Hub Site est simplement un site normal que vous désignez comme « central ». Les autres sites SharePoint s'associent à lui pour partager la navigation et l'apparence. C'est purement organisationnel, pas d'impact sur les permissions.

## 4.3 Créer le site Communication

1. Sites actifs > + Créer > Site de communication.

2. Nom : Communication demo.ch | Adresse : communication | Langue : Français.

3. Cliquez sur Créer.

4. Une fois créé : (…) > Hub > Associer au hub > choisissez Intranet demo.ch.

> **✅ PERMISSIONS : AUCUNE MODIFICATION NÉCESSAIRE** Ce site est destiné à toute l’entreprise. Les 3 groupes créés automatiquement par SharePoint conviennent tels quels. Groupe « Communication demo.ch - Membres » (Modification) : laissez-le vide ou ajoutez les communicants si besoin. Groupe « Communication demo.ch - Visiteurs » (Lecture) : ajoutez-y tous vos 4 groupes (GRP-Direction, GRP-RH, GRP-Finances, GRP-Standards) pour que tout le monde puisse lire le site. Ne supprimez PAS les groupes auto-créés, ils sont nécessaires au bon fonctionnement du site.

## 4.4 Créer le site Direction (accès restreint)

1. Sites actifs > + Créer > Site de l’équipe.

2. Nom : Direction demo.ch | Adresse : direction | Langue : Français.

3. Cliquez sur Créer.

4. Associez au hub Intranet demo.ch.

### Configurer les permissions du site Direction

SharePoint crée automatiquement 3 groupes dès la création du site. Voici comment les utiliser pour restreindre l’accès à la Direction uniquement :

| **Groupe créé par SharePoint** | **Niveau** | **Action à faire** |
| --- | --- | --- |
| Propriétaires de Direction demo.ch | Contrôle total | Conserver tel quel : admin du site |
| Direction demo.ch - Membres | Modification | Ajouter GRP-Direction ici : les directeurs peuvent déposer et modifier |
| Direction demo.ch - Visiteurs | Lecture | Vider ce groupe : personne d’autre ne doit accéder à ce site |

1. Ouvrez le site Direction demo.ch (cliquez sur son URL dans Sites actifs).

2. Cliquez sur l’engrenage ⚙️ > Paramètres du site > Autorisations du site > Paramètres d’autorisation avancée.

3. Si le message « Ce site hérite des autorisations » apparaît : cliquez sur Arrêter l’héritage des autorisations dans le ruban > confirmez.

4. Cliquez sur le groupe « Direction demo.ch - Membres » > Nouveau > Ajouter des utilisateurs > entrez GRP-Direction > cliquez sur Partager.

5. Cliquez sur le groupe « Direction demo.ch - Visiteurs » > sélectionnez tous les membres présents > Actions > Supprimer les utilisateurs du groupe. Ce groupe doit rester vide.

> **⚠️ PERSONNE D’AUTRE NE DOIT ACCÉDER À CE SITE** Vérifiez que le groupe Visiteurs est bien vide après l’étape 5. Si vous voyez « Tout le monde » ou « Tous les utilisateurs authentifiés » dans la liste, supprimez-les. Seuls les membres de GRP-Direction (via le groupe Membres) peuvent accéder à ce site.

## 4.5 Créer le site RH (accès restreint)

1. Sites actifs > + Créer > Site de l’équipe.

2. Nom : RH demo.ch | Adresse : rh-demo.ch | Langue : Français.

3. Cliquez sur Créer.

4. Associez au hub Intranet demo.ch.

### Configurer les permissions du site RH

Le site RH est accessible aux RH (modification) et aux Directeurs (lecture seulement). Personne d’autre ne doit y accéder.

| **Groupe créé par SharePoint** | **Niveau** | **Action à faire** |
| --- | --- | --- |
| Propriétaires de RH demo.ch | Contrôle total | Conserver tel quel : admin du site |
| RH demo.ch - Membres | Modification | Ajouter GRP-RH ici : les RH peuvent déposer et modifier |
| RH demo.ch - Visiteurs | Lecture | Ajouter GRP-Direction ici : les directeurs peuvent lire, pas modifier |

1. Ouvrez le site RH demo.ch > ⚙️ > Paramètres du site > Autorisations du site > Paramètres d’autorisation avancée.

2. Si le message « Ce site hérite des autorisations » apparaît : cliquez sur Arrêter l’héritage dans le ruban > confirmez.

3. Cliquez sur « RH demo.ch - Membres » > Nouveau > Ajouter des utilisateurs > entrez GRP-RH > cliquez sur Partager.

4. Cliquez sur « RH demo.ch - Visiteurs ». Supprimez les membres existants via Actions > Supprimer les utilisateurs du groupe, puis ajoutez GRP-Direction via Nouveau > Ajouter des utilisateurs.

> **ℹ️ CE SITE CONTIENDRA** Contrats de travail, fiches de paie, évaluations, ce sont des données personnelles sensibles au sens de la nLPD. C’est sur ce site que le guide Purview testera les étiquettes RH-Confidentiel et l’auto-labelling numéro AVS.

## 4.6 Créer le site Finances (accès restreint)

1. Sites actifs > + Créer > Site de l’équipe.

2. Nom : Finances demo.ch | Adresse : finances-demo.ch | Langue : Français.

3. Cliquez sur Créer.

4. Associez au hub Intranet demo.ch.

### Configurer les permissions du site Finances

Même logique que le site RH, les Finances travaillent en autonomie, les Directeurs peuvent consulter. Personne d’autre ne doit y accéder.

| **Groupe créé par SharePoint** | **Niveau** | **Action à faire** |
| --- | --- | --- |
| Propriétaires de Finances demo.ch | Contrôle total | Conserver tel quel : admin du site |
| Finances demo.ch - Membres | Modification | Ajouter GRP-Finances ici : les financiers peuvent déposer et modifier |
| Finances demo.ch - Visiteurs | Lecture | Ajouter GRP-Direction ici : les directeurs peuvent lire, pas modifier |

1. Ouvrez le site Finances demo.ch > ⚙️ > Paramètres du site > Autorisations du site > Paramètres d’autorisation avancée.

2. Si le message « Ce site hérite des autorisations » apparaît : cliquez sur Arrêter l’héritage dans le ruban > confirmez.

3. Cliquez sur « Finances demo.ch - Membres » > Nouveau > Ajouter des utilisateurs > entrez GRP-Finances > cliquez sur Partager.

4. Cliquez sur « Finances demo.ch - Visiteurs ». Supprimez les membres existants via Actions > Supprimer les utilisateurs du groupe, puis ajoutez GRP-Direction via Nouveau > Ajouter des utilisateurs.

> **ℹ️ CE SITE CONTIENDRA** Comptabilité, factures, budgets, IBAN, donc des données financières sensibles. Le guide Purview testera l’auto-labelling Finances-Confidentiel et la règle IBAN sur ce site.

## 4.7 Créer le site Transmission externe

Ce site a un rôle spécifique : il permet le dépôt de documents destinés à être transmis à des partenaires ou clients externes à l’entreprise.
Le partage s’effectue via des liens sécurisés nécessitant une authentification de l’utilisateur externe (invité), généralement à l’aide d’un compte Microsoft ou Entra ID externe, et non via un lien public accessible sans authentification par email.

Contrairement aux autres sites, le partage externe y est autorisé, mais de façon strictement contrôlée. La DLP et la stratégie de rétention qui s’appliquent à ce site seront décrites dans le guide Microsoft Purview.

> **📤 La LOGIQUE DE CE SITE : À LIRE AVANT DE COMMENCER** Chaque département dispose de SA PROPRE bibliothèque, isolée des autres. Un membre RH peut déposer dans la bibliothèque RH, il ne voit pas les bibliothèques Finance ou Direction. La racine du site est en lecture seule pour tout le monde sauf l’admin, on ne dépose rien à la racine. Le partage externe s’effectue exclusivement via des liens sécurisés nécessitant l’authentification de l’utilisateur externe (invité) ; aucun lien public accessible sans authentification n’est autorisé. La DLP et la rétention seront configurées par Purview, ne les touchez pas ici, cela sera décrit dans le guide Purview

> **RISQUE OPÉRATIONNEL IDENTIFIÉ** La sécurité du site Transmission externe repose sur : •  le respect strict de la procédure, •  l’absence de modification manuelle des groupes et autorisations. Une erreur de manipulation administrateur peut réouvrir l’accès croisé entre départements. Ce risque est accepté dans le cadre du POC.

### Étape 1 — Activer le partage externe au niveau du tenant

Avant de créer le site, vous devez autoriser le partage externe dans les paramètres généraux de SharePoint. Sans cette étape, il est impossible de partager un document avec quelqu’un hors de votre organisation.

> **⚠️ CETTE ACTION OUVRE LE PARTAGE POUR TOUT LE TENANT** Activer le partage externe au niveau du tenant est le prérequis technique, mais ce n’est pas une ouverture totale. Chaque site garde son propre niveau de partage. Les sites RH, Finances et Direction restent fermés à l’externe. Seul le site « Transmission externe » autorise le partage externe, via authentification des utilisateurs invités et jamais via des liens publics.

Connectez-vous sur https://admin.microsoft.com, puis cliquez sur Afficher tout > SharePoint > Stratégies > Partage.

La page Partage contient deux zones distinctes, configurez-les dans cet ordre :

### Zone 1 : Le contenu peut être partagé avec (curseurs SharePoint et OneDrive)

Cette zone contient deux curseurs verticaux : un pour SharePoint, un pour OneDrive. Chaque curseur a 4 positions. Déplacez les deux curseurs sur « Invités nouveaux et existants ».

| **Position du curseur** | **Libellé exact dans l’interface** | **Explication** | **À choisir ?** |
| --- | --- | --- | --- |
| Plus permissif (haut) | Tout le monde | Lien public sans connexion - n’importe qui sur internet peut accéder | ❌ Non |
| 2e position | Invités nouveaux et existants | Les invités doivent s’authentifier avec un compte Microsoft ou une identité Entra ID externe. Niveau recommandé. | ✅ Oui - choisir celui-ci |
| 3e position | Invités existants | Uniquement les personnes déjà présentes dans l’annuaire de votre organisation | ❌ Non |
| Plus restrictif (bas) | Uniquement les membres de votre organisation | Aucun partage externe autorisé | ❌ Non |

> **ℹ️ POURQUOI « INVITÉS NOUVEAUX ET EXISTANTS » ET PAS UN AUTRE NIVEAU ?** Ce niveau permet également l’accès à des destinataires sans compte Microsoft, via création d’un compte invité Microsoft temporaire lors de la première connexion. Le niveau inférieur (« Invités existants ») bloquerait les partenaires qui ne sont pas encore dans votre annuaire. Ce niveau du curseur est un plafond pour tout le tenant, chaque site peut le restreindre davantage, mais jamais l’assouplir.

### Zone 2 - Paramètres de partage externe supplémentaires

Cliquez sur « Paramètres de partage externe supplémentaires ∨ » pour déplier la section. Cinq cases à cocher apparaissent. Configurez-les comme suit :

| **Option visible dans l’interface** | **État recommandé** | **Pourquoi** |
| --- | --- | --- |
| Limiter le partage externe par domaine | ☐ Laisser décoché (POC) | Permet de restreindre le partage à certains domaines email (ex : uniquement @client.ch). Laissez décoché pour le POC afin de pouvoir tester avec n’importe quelle adresse. |
| Autoriser uniquement les membres de groupes de sécurité spécifiques à partager du contenu en externe | ☐ Laisser décoché (POC) | Le contrôle se fait ici via les permissions des bibliothèques, seuls les membres du bon groupe peuvent déposer des fichiers dans leur bibliothèque. |
| Autoriser les invités à partager des éléments qui ne leur appartiennent pas | ☑ DÉCOCHER si coché | Coché par défaut dans l’interface. Décochez-la : un partenaire qui reçoit un document ne doit pas pouvoir le redistribuer à une tierce personne sans votre accord. |
| L’accès invité à un site ou OneDrive expirera automatiquement après X jours | ☐ Laisser décoché (POC) | Utile en production pour révoquer automatiquement l’accès des invités. Pour le POC, laissez décoché. |
| Les personnes utilisant un code de vérification doivent s’authentifier de nouveau après X jours | ☑ COCHER saisir 30 | Force le destinataire à se réauthentifier après 30 jours. Sans cette option cochée, l’accès reste valide indéfiniment si cette case est décochée, risque de sécurité. |

> **⚠️ POINT D’ATTENTION**** :**** CASE « AUTORISER LES INVITÉS À PARTAGER »** Cette case est cochée par défaut dans la plupart des tenants Microsoft 365. Si vous la laissez cochée, un partenaire externe qui reçoit votre document peut créer un nouveau lien et l’envoyer à n’importe qui. Décochez-la systématiquement c’est la première chose à corriger dans un tenant neuf.

### Zone 3 - Liens des fichiers et des dossiers

Cette zone définit le type de lien proposé PAR DÉFAUT quand un utilisateur clique sur « Partager ». Deux groupes de boutons radio sont présents dans l’interface :

| **Paramètre** | **Valeur à sélectionner** | **Explication** |
| --- | --- | --- |
| Type de lien par défaut | Personnes spécifiques (personnes spécifiées par l’utilisateur uniquement) | Quand un utilisateur clique sur Partager, SharePoint propose automatiquement un lien nominatif. L’utilisateur doit saisir une adresse email le lien ne fonctionne que pour cette personne, qui doit s’authentifier pour y accéder. |
| Autorisation par défaut | Consultation | Le destinataire peut lire le document mais pas le modifier. L’utilisateur peut changer ce réglage au moment du partage si besoin. |

> **⚠️ NE PAS CHOISIR « TOUTE PERSONNE DISPOSANT DU LIEN »** Ce type de lien crée une URL réutilisable et transmissible si quelqu’un la reçoit par erreur, il accède au document sans vérification. Avec « Personnes spécifiques », le lien est lié à une adresse email et protégé par authentification il ne peut pas être réutilisé par un tiers sans connexion. Note : l’option « Toute personne disposant du lien » apparaît grisée si le curseur Zone 1 ne l’autorise pas, c’est normal.

1. Une fois les trois zones configurées, cliquez sur Enregistrer en bas de la page.

### Étape 2 — Créer le site

1. Dans le Centre d’administration SharePoint, cliquez sur Sites actifs > + Créer.

2. Choisissez Site de l’équipe.

3. Nom du site : Transmission externe | Adresse : transmission-externe | Langue : Français.

4. Propriétaire du site : admin@demo.ch.

5. Cliquez sur Créer.

6. N’associez PAS ce site au Hub Site, il doit rester indépendant.

> **✅ POURQUOI NE PAS ASSOCIER AU HUB ?** Les sites associés au Hub Intranet héritent de sa navigation et apparaissent dans les recherches internes. Le site Transmission externe est destiné aux échanges avec l’extérieur, le garder hors du Hub limite son exposition aux utilisateurs internes.

### Étape 3 — Rendre la racine du site non modifiable

La racine du site ne doit servir qu’à accueillir les bibliothèques. Personne ne doit pouvoir y déposer de documents directement.

Quand SharePoint crée un nouveau site, il génère automatiquement 3 groupes visibles dans la page des autorisations avancées. Voici ce que vous voyez dans l’interface :

| **Groupe créé automatiquement par SharePoint** | **Niveau par défaut** | **Ce que vous allez en faire** |
| --- | --- | --- |
| Propriétaires de Transmission externe | Contrôle total | Conserver tel quel : c’est le groupe admin du site |
| Transmission externe - Membres | Modification | Vider ce groupe (retirer tous les membres) : personne ne doit avoir Modification à la racine |
| Transmission externe - Visiteurs | Lecture | Ajouter vos 3 groupes Purview ici : ils verront le site mais ne pourront pas y déposer de fichiers |

### Procédure détaillée

1. Ouvrez le site Transmission externe (cliquez sur son URL dans Sites actifs).

2. Cliquez sur l’engrenage ⚙️ en haut à droite > Paramètres du site > Autorisations du site.

3. Cliquez sur Paramètres d’autorisation avancée. Vous arrivez sur la page avec le ruban et les 3 groupes.

4. Si la page affiche un message « Ce site hérite des autorisations », cliquez sur Arrêter l’héritage des autorisations dans le ruban > confirmez. Les 3 groupes sont maintenant indépendants du site parent.

> **ℹ️**** SI VOUS NE VOYEZ PAS « ARRÊTER L’HÉRITAGE »** Si le ruban montre directement les boutons Modifier les autorisations et Supprimer les autorisations (comme dans la capture), l’héritage est déjà stoppé passez directement à l’étape suivante.

5. Cliquez sur le groupe « Transmission externe - Membres » pour l’ouvrir.

6. Sélectionnez tous les membres présents (cochez la case en haut) > cliquez sur Actions > Supprimer les utilisateurs du groupe. Ce groupe reste vide, personne n’a Modification à la racine.

7. Revenez à la page des autorisations (bouton Retour du navigateur).

8. Cliquez sur le groupe « Transmission externe - Visiteurs » pour l’ouvrir.

9. Cliquez sur Nouveau > Ajouter des utilisateurs. Ajoutez les 3 groupes Purview : GRP-Direction-Purview, GRP-RH-Purview, GRP-Finances-Purview. Cliquez sur Partager.

| **Résumé après configuration** | **Groupe assigné** | **Niveau à la racine** |
| --- | --- | --- |
| Propriétaires de Transmission externe | admin@demo.ch (déjà présent) | Contrôle total |
| Transmission externe - Membres | (vide) | Modification - personne |
| Transmission externe - Visiteurs | GRP-Direction-Purview + GRP-RH-Purview + GRP-Finances-Purview | Lecture |

> **⚠****️  RÉSULTAT**** ATTENDU À CE STADE** Si un membre RH ouvre le site, il voit la page d’accueil mais ne peut PAS y glisser-déposer un fichier. Le bouton « + Nouveau » et le bouton « Télécharger » sont grisés ou absents à la racine. C’est le comportement voulu, les dépôts se font uniquement dans les bibliothèques (par déparlement).

### Étape 4 — Créer les 3 bibliothèques par département

Dans SharePoint, une bibliothèque est un espace de stockage avec ses propres permissions, c’est différent d’un simple dossier. Chaque département aura sa propre bibliothèque, invisible pour les autres.

> **📚 BIBLIOTHÈQUE vs DOSSIER ****-**** POURQUOI C’EST IMPORTANT** Un dossier hérite des permissions de la bibliothèque parente, tout le monde avec accès au site voit tous les dossiers. Une bibliothèque peut avoir ses PROPRES permissions indépendantes. C’est ce qui permet l’isolation entre départements. Pour ce site, on utilise obligatoirement des bibliothèques, jamais des dossiers.

Créez les 3 bibliothèques suivantes (répétez la procédure pour chacune) :

1. Sur le site Transmission externe, cliquez sur l’engrenage ⚙️ > Ajouter une application.

2. Dans la liste, cliquez sur Bibliothèque de documents.

3. Nom : RH - Transmission externe. Cliquez sur Créer.

4. Répétez : Finance - Transmission externe.

5. Répétez : Direction - Transmission externe.

### Étape 5 — Sécuriser chaque bibliothèque

C’est l’étape la plus importante. Chaque bibliothèque doit avoir ses propres permissions indépendantes pour qu’un membre RH ne puisse pas voir la bibliothèque Finance, et inversement.

Répétez cette procédure pour chacune des 3 bibliothèques.

> **⚠️ POINT CRITIQUE**** :**** POURQUOI NE PAS CLIQUER SUR LE NOM DU GROUPE** Depuis la page des autorisations de la bibliothèque, si vous cliquez sur « Transmission externe - Membres » puis Nouveau, vous ouvrez la page du groupe au niveau du SITE. Ce groupe est partagé par toutes les bibliothèques. Ajouter GRP-Finances-Purview ici lui donnerait accès à RH et Direction également, c’est l’opposé de l’isolation recherchée. La bonne méthode est d’utiliser le bouton Accorder des autorisations depuis le ruban de la BIBLIOTHÈQUE, pas depuis la page du groupe.

### Accéder aux autorisations de la bibliothèque

1. Sur le site Transmission externe, cliquez sur Contenu du site dans le menu de gauche.

2. Repérez la bibliothèque à configurer (ex : Finance - Transmission externe).

3. Cliquez sur les 3 points (…) à droite du nom > Paramètres.

4. Dans la page Paramètres, sous la colonne Autorisations et gestion, cliquez sur Autorisations pour le composant : bibliothèque de documents.

> **ℹ️  CE**** QUE VOUS VOYEZ À CE STADE** Bandeau jaune : « Cette bibliothèque hérite des autorisations de son parent. (Transmission externe) ». Ruban : Gérer les parents │ Arrêter l’héritage des autorisations │ Vérifier les autorisations. Liste : les 3 groupes du site (Propriétaires, Membres, Visiteurs) - c’est normal, ils sont hérités.

5. Cliquez sur Arrêter l’héritage des autorisations dans le ruban. Cliquez sur OK dans la boîte de dialogue.

6. Le bandeau jaune disparaît. Le ruban affiche maintenant de nouveaux boutons : Accorder des autorisations | Modifier les autorisations | Supprimer les autorisations des utilisateurs.

### Supprimer les groupes hérités non nécessaires

1. Cochez la case à gauche de « Transmission externe - Membres ».

2. Cochez également la case de « Transmission externe - Visiteurs ».

3. Dans le ruban, cliquez sur Supprimer les autorisations des utilisateurs. Confirmez.

4. Il ne doit rester que « Propriétaires de Transmission externe » (Contrôle total) dans la liste.

### Accorder les autorisations au bon groupe Purview

Dans le ruban de la bibliothèque, cliquez sur Accorder des autorisations. Un dialogue apparaît intitulé : « Partager « [Nom de la bibliothèque] » et son contenu ». Vérifiez que le titre mentionne bien le nom de votre bibliothèque, c’est la confirmation que vous agissez sur la bibliothèque et non sur le site.

1. Dans le champ Inviter des personnes, tapez le nom du groupe Purview correspondant à cette bibliothèque (ex : GRP-RH-Purview pour la bibliothèque RH). Sélectionnez-le dans la liste déroulante qui apparaît.

2. Ne pas ajouter admin@demo.ch séparément, le groupe « Propriétaires de Transmission externe » contient déjà le compte admin avec Contrôle total. L’ajouter en double serait redondant.

3. Laissez la case « Partager tous les éléments de ce dossier, y compris ceux dotés d’autorisations uniques » cochée, c’est correct.

4. Cliquez sur AFFICHER LES OPTIONS (lien en minuscules sous la zone de message).

5. Une liste déroulante du niveau d’autorisation apparaît. Changez-la de Lecture à Modification.

6. Décochez « Envoyer une invitation par courrier électronique » si l’option est visible, inutile pour un groupe interne.

7. Cliquez sur Partager.

> **✅  RÉSULTAT ATTENDU DANS LA LISTE DES AUTORISATIONS** Le bandeau jaune indique : « Cette bibliothèque dispose d’autorisations uniques. », c’est la confirmation que l’héritage est bien stoppé. Vous devez voir exactement 2 entrées dans la liste, rien d’autre : •  GRP-[Département]-Purview - Type : Groupe de domaines - Niveau : Modification •  Propriétaires de Transmission externe - Type : Groupe SharePoint - Niveau : Contrôle total Le compte admin n’apparaît pas directement, c’est normal. Il est déjà membre de « Propriétaires de Transmission externe », ce qui lui donne Contrôle total sur toutes les bibliothèques. Si vous voyez encore Transmission externe - Membres ou Visiteurs : cochez-les et cliquez sur Supprimer les autorisations des utilisateurs.

| **Bibliothèque** | **Groupe à ajouter (Modification)** | **Groupes à supprimer** | **Admin couvert par** |
| --- | --- | --- | --- |
| RH - Transmission externe | GRP-RH-Purview | Transmission externe - Membres + Visiteurs | Propriétaires (Contrôle total) |
| Finance - Transmission externe | GRP-Finances-Purview | Transmission externe - Membres + Visiteurs | Propriétaires (Contrôle total) |
| Direction - Transmission externe | GRP-Direction-Purview | Transmission externe - Membres + Visiteurs | Propriétaires (Contrôle total) |

> **⚠️  ÉTAT FINAL ATTENDU POUR CHAQUE BIBLIOTHÈQUE** Propriétaires de Transmission externe : Contrôle total (conservé) Transmission externe - Membres : supprimé de la liste Transmission externe - Visiteurs : supprimé de la liste GRP-[Département]-Purview : Modification (entrée directe) admin@demo.ch : Modification (entrée directe)

> **✅  VÉRIFICATION RAPIDE** Connectez-vous avec rh1@demo.ch et ouvrez le site Transmission externe > Contenu du site. Vous devez voir UNIQUEMENT la bibliothèque RH - Transmission externe, pas les deux autres. Si les 3 bibliothèques sont visibles : l’héritage n’a pas été correctement arrêté, reprenez depuis l’étape 5 (Arrêter l’héritage). Si les autorisations semblent correctes mais la visibilité croisée persiste : vérifiez qu’il ne reste pas de trace de Transmission externe - Membres dans la liste des permissions de la bibliothèque.

### Étape 6 — Configurer le partage externe sécurisé sur ce site

Maintenant que le site est sécurisé, configurez le niveau de partage externe autorisé pour ce site uniquement. Ce réglage est indépendant des autres sites - RH, Finances et Direction restent fermés à l’externe.

1. Dans le Centre d’administration SharePoint (admin.microsoft.com > SharePoint), allez dans Sites actifs.

2. Cliquez sur le nom du site « Transmission externe » dans la liste. Un panneau s’ouvre à droite.

3. Dans le panneau de droite, cliquez sur l’onglet Paramètres.

4. Sous « Partage de fichiers externes », cliquez sur la liste déroulante. Vous voyez 4 options (voir tableau ci-dessous).

5. Sélectionnez Invités nouveaux et existants.

6. Cliquez sur Enregistrer en bas du panneau.

| **Option dans la liste déroulante** | **Explication** | **À choisir ?** |
| --- | --- | --- |
| Tout le monde | Lien public sans connexion, n’importe qui sur internet peut accéder sans aucune vérification | ❌ Non |
| Invités nouveaux et existants | Les invités doivent s’authentifier avec un compte Microsoft ou une identité Entra ID externe. Niveau recommandé - authentification obligatoire avant l’accès. | ✅ Oui :  choisir celui-ci |
| Invités existants | Uniquement les personnes déjà présentes dans l’annuaire. Un partenaire inconnu ne peut pas recevoir de lien. | ❌ Non |
| Uniquement les personnes de votre organisation | Aucun partage externe autorisé : réservé aux sites purement internes | ❌ Non |

> **ℹ️  CE RÉGLAGE EST UN PLAFOND POUR CE SITE** Le niveau choisi ici ne peut pas dépasser le niveau défini au niveau du tenant (Zone 1 de l’Étape 1). Si le tenant est réglé sur « Invités nouveaux et existants », ce site peut aussi utiliser ce niveau ou être plus restrictif. Les autres sites (RH, Finances, Direction) restent sur « Uniquement les personnes de votre organisation » par défaut, ils ne sont pas affectés par ce réglage.

> **🔐  COMMENT**** FONCTIONNE LE PARTAGE EXTERNE SÉCURISÉ ****-**** POUR LE DÉBUTANT** 1. Un membre RH ouvre un document dans la bibliothèque RH - Transmission externe. 2. Il clique sur Partager et entre l’adresse email du partenaire externe. 3. Le partenaire reçoit un email avec un lien sécurisé. En cliquant, il est invité à s’authentifier avec son compte Microsoft ou Entra ID (ou à créer un compte invité s’il n’en a pas). 4. Il saisit le code et peut accéder au document pendant la durée configurée (30 jours selon Zone 2 de l’étape 1). Sans le code, le lien est inutilisable. 5. La DLP Purview analyse le document avant le partage et peut le bloquer si le contenu est trop sensible. → La configuration DLP et la rétention de ce site sont dans le guide Microsoft Purview (pas ici).

---

[← Préparation — Création des comptes utilisateurs](prep-03-comptes-utilisateurs.html) | [Préparation — Activer l'audit Microsoft 365 →](prep-05-audit.html)
