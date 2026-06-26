---
title: "Partie 11 — Gouvernance des accès SharePoint | Configuration Microsoft Purview 2026"
description: "Gouvernance des accès SharePoint : modèle de permissions, audit du sur-partage, correction des permissions selon le principe du moindre privilège, blocage du partage externe et routine opérationnelle de contrôle, y compris OneDrive."
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

# Partie 11 — Gouvernance des accès SharePoint

Purview classifie et protège les données mais il ne corrige pas les permissions. Si un site SharePoint est ouvert à tout le monde, Copilot peut y accéder, et un label confidentiel ne changera rien à cela. Cette partie explique comment auditer et corriger le sur-partage dans votre tenant Business Premium sans outil supplémentaire. C’est le complément indispensable de tout ce qui a été configuré dans les parties précédentes.

**⚠️ Rappel important :** La gouvernance des accès n’est pas optionnelle dans un contexte nLPD. L’art. 8 exige des mesures techniques adaptées et laisser des données RH accessibles à l’ensemble du personnel constitue une violation du principe de proportionnalité (art. 6 al. 4 nLPD).

## 11.1 Comprendre le modèle de permissions SharePoint

Les trois niveaux de permissions

SharePoint fonctionne avec trois niveaux qui héritent l’un de l’autre. Le niveau Site est le plus large :celui qui a accès au site a accès à tout son contenu par défaut. Le niveau Bibliothèque permet de restreindre l’accès à un dossier ou une bibliothèque de documents spécifique. Le niveau Document permet de restreindre l’accès à un fichier unique, à utiliser avec parcimonie car cela complexifie la gestion.

| **Niveau** | **Risque de sur-partage** | **Recommandation** |
| --- | --- | --- |
| **⚠️ Tout le monde (y compris les personnes extérieures)** | Maximal : internet peut accéder aux données | ❌ Jamais pour des données internes |
| **⚠️ Tous les membres de l’organisation** | Élevé : tout employé voit tout | ⚠️ À éviter pour les sites RH / Finance / Direction |
| **✅ Groupes de sécurité spécifiques** | Faible : seules les personnes autorisées accèdent | ✅ Modèle recommandé pour données sensibles |

## 11.2 Auditer le sur-partage existant dans votre tenant

> ⚠️ **AVERTISSEMENT nLPD (Art. 8) : **Le nettoyage manuel des permissions est une étape initiale indispensable, mais insuffisante sur le long terme. La sécurité des accès n’est pas un état statique, c’est un processus continu. Sans surveillance récurrente, le sur-partage réapparaîtra naturellement par les actions des utilisateurs. Consultez la section 11.6 pour la mise en place d’une routine de gouvernance pérenne.

Procédure : Rapport de partage dans le Centre d’administration SharePoint

1. Accédez directement à https://admin.sharepoint.com. Vous arrivez dans le Centre d’administration SharePoint sans passer par admin.microsoft.com. Autre chemin : admin.microsoft.com → faites défiler le menu gauche jusqu’à la section Centres d’administration → SharePoint.

2. Menu de gauche → Sites → Sites actifs. La liste affiche tous les sites SharePoint du tenant avec les colonnes Niveau de confidentialité et Partage externe. La colonne Partage externe indique l’état de chaque site : Désactivé ou Activé.

3. Identifiez les sites avec Partage externe = Activé. Attention : Activé ne signifie pas que des fichiers sont partagés, cela signifie que la fonctionnalité est disponible. Le niveau exact (Tout le monde, Invités nouveaux et existants, Uniquement les personnes de votre organisation) est visible dans les paramètres du site. État validé terrain pour le tenant Axonix SA : Applications (Désactivé), Direction (Désactivé), Finances (Désactivé), RH (Activé → corrigé en section 11.3), Intranet (Activé → à corriger), Site de communication (Activé → à corriger), Transmission externe (Activé par design).

4. Pour chaque site Activé : cliquez sur son nom → onglet Paramètres → vérifiez le champ Partage de fichiers externes. Si la valeur est Invités nouveaux et existants ou Tout le monde : le site est sur-partagé. Notez-le pour la procédure de correction (section 11.3). Note : l’héritage de ce niveau vient de la configuration du tenant. Microsoft configure par défaut Invités nouveaux et existants au niveau tenant, et tous les nouveaux sites en héritent automatiquement.

Procédure : Rapport des partages dans Purview (Content Explorer)

Purview Content Explorer vous permet de voir quels fichiers étiquetés sont accessibles à qui. C’est complémentaire au Centre d’administration SharePoint, il croise les labels avec les permissions.

1. Purview → Protection des informations → Explorateur de contenu.

2. Filtrez par Étiquette de sensibilité → sélectionnez 4 → RH-Confidentiel. Vous voyez tous les fichiers classifiés RH dans votre tenant avec leur emplacement SharePoint exact.

3. Pour chaque fichier listé, notez le site SharePoint parent. Si ce site figure dans votre liste de sites sur-partagés (étape précédente), c’est une priorité de correction immédiate.

## 11.3 Corriger les permissions, appliquer le principe du moindre privilège

Procédure ; Restreindre le partage externe d’un site

1. https://admin.sharepoint.com → Sites → Sites actifs → cliquez sur le nom du site à corriger → onglet Paramètres.

2. Champ Partage de fichiers externes : changez Invités nouveaux et existants en Uniquement les personnes de votre organisation.

3. Champ Restreindre le contenu Microsoft 365 Copilot : sélectionnez Activé pour les sites contenant des données sensibles (RH, Finances). Copilot ne pourra plus accéder aux données de ce site sans contrôle explicite.

4. Cliquez sur Enregistrer. Le partage externe est immédiatement restreint.

ℹ️ **Étiquette de confidentialité sur les sites : **seules les étiquettes dont la portée inclut Groupes et sites peuvent être appliquées à un site SharePoint. Dans ce POC, uniquement 3a - Direction - Site et 6 - Transmission-Externe ont cette portée. Les étiquettes 4 - RH-Confidentiel et 5 - Finances-Confidentiel s’appliquent aux fichiers, pas aux sites : le menu déroulant Étiquette de confidentialité affichera None pour ces sites, ce qui est correct.

ℹ️ **Héritage du niveau de partage tenant : **Microsoft configure par défaut le tenant sur Invités nouveaux et existants. Tous les nouveaux sites en héritent automatiquement. Pour changer le niveau par défaut : admin.sharepoint.com → Stratégies → Partage → curseur SharePoint → Uniquement les personnes de votre organisation → Enregistrer. Les sites déjà créés conservent leur niveau individuel.

**Configuration pour le tenant Axonix SA :**

RH : Partage externe = Uniquement les personnes de votre organisation, Copilot = Activé. | Intranet : à corriger (Activé → Uniquement les personnes). | Site de communication : à corriger. | Transmission externe : Activé par design (site de transit). | Direction, Finances, Applications : Désactivé.

Procédure : Remplacer les permissions héritées par des groupes spécifiques

Si un site SharePoint est actuellement accessible à Tout le monde de l’organisation et contient des données sensibles, voici la marche à suivre pour le restreindre au bon groupe.

1. Ouvrez le site SharePoint concerné → icône Paramètres (engrenage) → Autorisations du site. Un panneau s’ouvre à droite affichant les trois groupes du site : Propriétaires de site (contrôle total), Membres du site (contrôle limité), Visiteurs du site (aucun contrôle).

2. Pour ajouter un membre autorisé : cliquez sur le bouton + Ajouter des membres → saisissez le nom du groupe de sécurité (ex. : GRP-RH-Purview pour le site RH) → sélectionnez-le dans la liste → choisissez le niveau Modification → Envoyer.

3. Pour voir et gérer tous les groupes et utilisateurs (y compris les entrées héritées) : cliquez sur le lien Paramètres avancés des autorisations en bas du panneau. Vous accédez à la page classique des autorisations SharePoint où vous pouvez identifier et supprimer les entrées génériques (Tout le monde, Tous les utilisateurs authentifiés).

4. Pour supprimer un utilisateur ou groupe : dans la page Paramètres avancés des autorisations, cochez la case à gauche de l’entrée → menu Actions → Supprimer les autorisations utilisateur → confirmez.

**⚠️ ATTENTION —** Avant de supprimer des accès, vérifiez qu’au moins un administrateur reste Propriétaire du site. Sans propriétaire, vous perdez la gestion du site. Il est recommandé de faire ces modifications en dehors des heures de bureau et de prévenir les utilisateurs concernés.

## 11.4 Bloquer le partage externe au niveau du tenant

Si votre organisation n’a aucun besoin de partage avec des personnes extérieures, vous pouvez bloquer le partage externe globalement, ce qui est plus simple et plus sûr que de le gérer site par site.

1. admin.sharepoint.com → Stratégies → Partage.

2. Section Partage externe → curseur SharePoint → déplacez le curseur jusqu’à Uniquement les personnes de votre organisation. Faites de même pour le curseur OneDrive. Cliquez sur Enregistrer.

**ℹ️**** INFO :** Ce paramètre est un plafond, il ne peut pas être contredit au niveau des sites individuels. Si vous le définissez à Uniquement les personnes de votre organisation, aucun site ne pourra être partagé en externe, même si un administrateur essaie de l’activer sur un site spécifique.

## 11.5 Lien Copilot : ce que la gouvernance des accès change concrètement

Copilot M365 respecte strictement les permissions SharePoint. Si un utilisateur n’a pas accès à un fichier, Copilot ne peut pas le lui montrer quelles que soient les questions posées. En conséquence, restreindre les permissions SharePoint est la mesure la plus efficace pour contrôler ce que Copilot peut exploiter, plus encore que les labels ou les politiques DLP.

| **Sans gouvernance des accès** | **Avec gouvernance des accès** |
| --- | --- |
| Copilot peut agréger tous les fichiers RH auxquels l’employé a accès, même ceux qu’il n’aurait jamais ouverts manuellement | Copilot ne voit que les fichiers RH auxquels l’employé est explicitement autorisé |
| Label RH-Confidentiel appliqué mais site ouvert à tous, Copilot peut quand même résumer le contenu pour n’importe quel employé | Label RH-Confidentiel + site restreint au groupe GRP-RH-Purview : Copilot ne peut pas accéder au site pour les non-membres |

## 11.6 Routine opérationnelle de contrôle des accès

La gouvernance des accès SharePoint n’est pas un état statique : sans contrôle récurrent, le sur-partage réapparaîtra naturellement. Pour satisfaire durablement aux exigences de l’art. 8 nLPD, l’administrateur doit mettre en place les trois contrôles suivants.

Rapport de partage mensuel

Centre d’administration SharePoint → Sites → Sites actifs → exportez la liste en CSV. Filtrez la colonne "Partage externe" pour identifier tout site passé en Tout le monde ou Tout le monde sauf les invités externes depuis le mois précédent. Traitez chaque déviation conformément à la procédure de la section 11.3.

Audit Content Explorer trimestriel

Dans Purview → Protection des informations → Explorateur de contenu, filtrez par étiquette 4 — RH-Confidentiel et 5 — Finances-Confidentiel. Croisez la liste des fichiers avec les sites identifiés comme sur-partagés lors du rapport mensuel. Toute occurrence constitue une priorité de correction immédiate.

Incidents DLP dans Microsoft Defender (automatique)

Les incidents déclenchés par les politiques DLP remontent automatiquement dans security.microsoft.com → Enquête et réponse → Incidents et alertes → Incidents, sans configuration supplémentaire. La colonne Nom de la stratégie identifie quelle politique DLP a déclenché l’incident (ex. : DLP-Protection-Transmission-Externe, DLP-Protection-Copilot). Validé terrain : les tests de validation de ce POC ont généré 3 incidents visibles dans Defender, categorie. Exfiltration, ressource affectée identifiée. Aucune alerte manuelle à configurer. Ce mécanisme constitue une mesure technique automatique au sens de l’art. 8 nLPD.

Gouvernance des alertes (Drift) à définir avant mise en production

La section 11.6 met en place les routines techniques de détection. Pour être conformé à l’art. 8 nLPD (mesures techniques ET organisationnelles), quatre paramètres de gouvernance doivent être formalisés avant le déploiement en production.

| **Paramètre** | **Description** | **Valeur recommandée (POC)** |
| --- | --- | --- |
| **Destinataire(s) de l’alerte** | Personne(s) physique(s) désignée(s) pour recevoir et traiter les alertes. Ne pas utiliser une boîte mail générique non surveillée. | *À définir : Responsable SI / DPO (production). Pour le POC : admin@demo.ch* |
| **SLA d’analyse** | Délai maximal entre la réception de l’alerte et la première analyse (confirmation ou rejet de fausse alarme). | *À définir : recommandé ≤ 4 heures ouvrables* |
| **SLA de correction / escalade** | Délai maximal pour corriger la permission fautive ou escalader à la direction si la décision dépasse le périmètre de l’administrateur. | *À définir : recommandé ≤ 24 heures (ou notification PFPDT dès que possible (art. 24 nLPD))* |
| **Journalisation des interventions** | Chaque alerte traitée doit être consignée (date, action menée, décision) pour constituer la preuve d’un traitement conforme en cas de contrôle PFPDT. | *À définir : ticket ITSM ou registre des incidents sécurité* |

Automatisation PowerShell (tenants > 20 sites)

Pour les tenants dépassant 20 sites actifs, l’utilisation de scripts PowerShell (module PnP.PowerShell) est recommandée pour lister automatiquement les sites avec permissions larges et envoyer un rapport hebdomadaire par email à l’administrateur. Ce niveau d’automatisation transforme la correction ponctuelle en cycle de gouvernance continu, conformément à l’art. 8 nLPD (mesures techniques et organisationnelles).

> 💡 **LIEN nLPD ART. 8 DEUX PILIERS INDISSOCIABLES. **La protection des données repose sur deux piliers complémentaires. (1) L’étiquette (le Label) : protège le contenu si le fichier sort du périmètre grâce au chiffrement Azure RMS. (2) La permission (l’Accès) : sécurise le vecteur d’exposition en empêchant les utilisateurs non autorisés d’accéder au fichier dans SharePoint ou via Copilot. L’absence de l’un rend l’autre vulnérable : un fichier chiffré sur un site ouvert à tous reste protégé en contenu, mais reste exposé à l’extraction par Copilot pour tout utilisateur authentifié. Une conformité nLPD robuste nécessite les deux piliers activés simultanément.

## 11.7 Focus OneDrive : Gouvernance du SharePoint personnel

OneDrive est le site SharePoint personnel de chaque utilisateur. Techniquement identique à un site SharePoint d’équipe, il est souvent moins surveillé et plus difficile à auditer en masse. Un partage involontaire de données sensibles via OneDrive constitue une violation nLPD aussi grave qu’un sur-partage sur le site RH.

Aligner le niveau de restriction OneDrive sur SharePoint

admin.sharepoint.com → Stratégies → Partage. Dans la section Partage externe, vous disposez de deux curseurs indépendants : un pour SharePoint et un pour OneDrive. Vérifiez que le curseur OneDrive est positionné au même niveau que SharePoint ou à un niveau plus restrictif. OneDrive ne peut jamais être moins restrictif que SharePoint, si vous tentez de l’assouplir, le portail le bloquera automatiquement.

Audit mensuel des partages OneDrive

Dans Purview → Audit (menu gauche) → Rechercher : champ Activités : noms conviviaux → tapez Fichier partagé avec un utilisateur externe (SharePoint), plage de dates sur les 30 derniers jours, tous les utilisateurs. Champ ObjectId (fichier, dossier ou site) : laissez vide pour couvrir tous les OneDrive. Exportez les résultats en CSV. Tout partage externe sur un OneDrive contenant des fichiers étiquetés RH-Confidentiel, Finances-Confidentiel ou Direction-Confidentiel doit être examiné et justifié.

> ℹ️ **LIEN nLPD (Art. 8) : **L’art. 8 nLPD exige des mesures techniques couvrant l’ensemble des systèmes traitant des données personnelles y compris les espaces de stockage personnels. L’audit mensuel des partages OneDrive constitue une mesure de surveillance obligatoire. Un partage non justifié de données sensibles via OneDrive expose l’organisation à la même responsabilité qu’un partage sur un site d’équipe.

Checklist de validation section 11

| **✓** | **Action** | **Statut** |
| --- | --- | --- |
| ☐ | Rapport des sites actifs consulté — aucun site ouvert à Tout le monde | à faire |
| ☐ | Sites RH, Finance, Direction restreints aux groupes de sécurité correspondants | à faire |
| ☐ | Content Explorer vérifié — fichiers RH-Confidentiel sur sites correctement restreints | à faire |
| ☐ | Partage externe bloqué au niveau du tenant (si applicable) | à faire |

---

[← Partie 10 — Gouvernance Copilot Studio et Power Platform](11-power-platform.html) | [Annexes →](13-annexes.html)
