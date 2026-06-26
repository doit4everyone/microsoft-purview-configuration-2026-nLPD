---
title: "Partie 9 — Endpoint DLP, Edge for Business et AI Hub | Configuration Microsoft Purview 2026"
description: "Endpoint DLP Purview Suite, contrôles IA dans Edge for Business avec déploiement de l'extension navigateur via Intune, et Purview AI Hub pour la surveillance unifiée de l'IA dans le tenant."
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

# Partie 9 — Endpoint DLP, Edge for Business et AI Hub

Cette partie couvre trois fonctionnalités complémentaires qui étendent la protection de vos données jusqu’aux appareils physiques et aux interactions avec l’intelligence artificielle. Les parties précédentes protégeaient vos données dans le cloud (SharePoint, Exchange, Teams). Cette partie protège ce qui se passe sur l’ordinateur lui-même par exemple quand un utilisateur copie un fichier sur une clé USB, imprime un document confidentiel, ou demande à ChatGPT de reformuler un contrat RH.

| **Fonctionnalité** | **Licence requise** | **Section** |
| --- | --- | --- |
| Endpoint DLP (protection sur l’appareil) | ✅ Purview Suite | 9.1 |
| Contrôles IA dans Edge for Business | ✅ Business Premium | 9.2 |
| Purview AI Hub / DSPM for AI (tableau de bord) | ✅ Purview Suite | 9.3 |

## 9.0 Positionnement de la DLP Appareils (Endpoint DLP)

Les politiques DLP configurées dans les parties précédentes protègent les données lorsqu’elles circulent dans le cloud Microsoft 365 (Exchange, SharePoint, OneDrive, Teams, Copilot). Elles ne couvrent pas les actions effectuées localement sur les postes de travail : copie vers un support amovible (clé USB), impression, copier-coller entre applications, ou upload via un navigateur vers des services externes.

La **DLP Appareils (Endpoint DLP)** constitue une politique distincte dédiée à la surveillance et au contrôle des usages sur les appareils Windows intégrés dans Microsoft Purview. Elle ne remplace pas les politiques DLP cloud, les permissions SharePoint ni le chiffrement RMS, elle constitue une mesure complémentaire visant à réduire les risques d’exfiltration en dehors des canaux Microsoft 365, conformément à l’art. 8 nLPD (mesures techniques et organisationnelles).

## 9.1 Endpoint DLP, inclus Purview Suite

Concept : pourquoi le cloud ne suffit pas

Les politiques DLP des parties précédentes protègent les données quand elles circulent dans le cloud Microsoft 365, SharePoint, Exchange, Teams. Mais imaginez ce scénario : un employé ouvre un contrat RH depuis SharePoint, le télécharge sur son bureau Windows, puis le copie sur une clé USB pour l’emporter chez lui. Aucune des règles cloud ne voit ce qui se passe sur l’ordinateur local. C’est exactement la faille que comble l’Endpoint DLP.

L’Endpoint DLP s’appuie sur les capacités locales de Microsoft Defender for Endpoint (MDE), déjà présent sur les appareils Windows via Microsoft 365 Business Premium. Il ne s’agit pas d’un nouvel agent indépendant, mais d’une extension des capacités MDE pour surveiller et contrôler les actions sur les données sensibles au niveau du poste : copie vers un stockage USB, impression, envoi vers une application non autorisée (WhatsApp, Telegram). Il peut bloquer l’action, demander une justification, ou simplement logger l’événement dans Purview.

Ce que l’Endpoint DLP surveille et peut bloquer

| **Action surveillée** | **Exemple concret** | **Réponse possible** |
| --- | --- | --- |
| Copie vers périphérique USB | Glisser-déposer le fichier RH-Confidentiel.docx vers une clé USB | Bloquer ou exiger justification |
| Impression sur imprimante locale ou réseau | Imprimer un contrat salarial depuis Word | Logger + alerte admin |
| Upload vers application cloud non autorisée | Déposer un fichier confidentiel sur Dropbox ou Google Drive personnel | Bloquer |
| Copier-coller dans une application non autorisée | Copier un numéro AVS depuis Excel et le coller dans WhatsApp Desktop | Bloquer le presse-papiers |

Prérequis : intégrer les appareils Windows dans Purview

Pour que l’Endpoint DLP fonctionne, chaque poste Windows doit être intégré dans Purview (onboarding). Cela se fait via Microsoft Intune si vos appareils sont gérés (MDM), ou via un script local pour les appareils non-MDM. Voici la procédure pour un appareil individuel :

> ⚠️ **RISQUE RÉSIDUEL : BYOD (Bring Your Own Device) **La protection assurée par l’Endpoint DLP ne s’applique qu’aux appareils intégrés (onboarded) dans Purview. Les ordinateurs personnels non gérés (BYOD) constituent un risque résiduel : si un utilisateur télécharge un fichier confidentiel sur son poste professionnel protégé, puis le copie sur son ordinateur personnel, aucune règle Endpoint DLP ne s’applique sur ce dernier. Le chiffrement Azure RMS reste la dernière ligne de défense dans ce scénario. Le fichier demeure illisible hors du tenant même sur un appareil non géré, pour autant que l’étiquette de chiffrement ait été appliquée.

1. Dans Purview → Paramètres → Intégration d'un appareil → Intégration : sélectionnez le système d'exploitation Windows 10 → Méthode de déploiement : Script local (jusqu'à 10 ordinateurs) → Télécharger le package. Un fichier .zip est téléchargé.

2. Sur le poste Windows à intégrer : décompressez le fichier .zip → clic droit sur DeviceComplianceLocalOnboardingScript.cmd → Exécuter en tant qu'administrateur → acceptez le contrôle de compte utilisateur (UAC).

3. Attendez environ 15 minutes. Retournez dans Purview → Paramètres → Intégration d'un appareil → Appareils. Votre appareil doit apparaître dans la liste avec le statut Mis à jour.

⚠️ Script identique à Defender for Endpoint : si vos postes sont déjà intégrés dans Microsoft Defender for Endpoint (ce qui est le cas sur tout tenant Business Premium correctement configuré), ils apparaissent automatiquement dans la liste des appareils Purview sans exécuter le script. Vérifiez d'abord Purview → Paramètres** → **Intégration d'un appareil** → **Appareils avant de procéder à l'onboarding manuel.

**ℹ️**** INFO :** si vos postes sont gérés par Intune (Azure AD joint), l'intégration peut se faire en masse depuis Purview** → **Paramètres** → **Intégration d'un appareil** → **Intégration** → **Méthode de déploiement** : **Microsoft Intune**. **Tous les appareils Intune sont alors intégrés automatiquement sans script manuel.

Prérequis : Vérifier la protection en temps réel (RTP) et l’analyse comportementale (BM)

Endpoint DLP dépend de Microsoft Defender for Endpoint (MDE). Sans la Protection en temps réel (RTP) et l’Analyse comportementale (BM) actives sur le poste, Purview ne peut pas intercepter les activités navigateur (upload, copier-coller), seules les activités système minimales (presse-papiers) sont loggées. Vérifiez l’état de l’appareil dans Purview → Paramètres** → **Intégration d'un appareil** → **Appareils → sélectionnez l'appareil → section État de configuration. Si RTP ou BM affiche Désactivé, corrigez avant de tester.

Correction via PowerShell (admin) sur le poste affecté :

Set-MpPreference -DisableRealtimeMonitoring $false

Set-MpPreference -DisableBehaviorMonitoring $false

Déploiement forcé via Intune (production) **→** Intune **→** Sécurité des points de terminaison **→** Antivirus **→** Créer une politique **→** Windows **→** Microsoft Defender Antivirus **→** Protection en temps réel : Activé **→** Surveillance du comportement : Activé. Cette politique Intune écrase les GPO locales conflictuelles. 

⚠️ Sur les VM de test : vérifiez que la Protection en temps réel (RTP) et l'Analyse comportementale (BM) de Defender sont bien actives. Si elles sont désactivées, Endpoint DLP ne peut pas intercepter les activités navigateur. Vérifiez dans security.microsoft.com → Appareils → sélectionnez l'appareil → État de l'appareil.

Procédure : Créer la politique DLP-Protection-Endpoint

L’Endpoint DLP doit être configuré dans une **politique séparée** et non dans DLP-Protection-nLPD-demo. Raison : la condition « Du contenu est partagé avec des personnes extérieures » est incompatible avec l’emplacement Appareils. Purview renvoie une erreur de validation si on tente de combiner les deux dans la même politique.

1. Purview → Protection contre la perte de données → Stratégies → + Créer une stratégie → Personnalisé → Personnalisé → Suivant.

2. Nom : **DLP-Protection-Endpoint** : Description : Protection Endpoint DLP appareils Windows → **Suivant.**

3. Unités administratives : Répertoire complet → **Suivant.**

4. Emplacements : activez Appareils uniquement : désactivez tous les autres emplacements → **Suivant.**

5. Définir les paramètres de protection → + Créer une règle.

6. Nom de la règle : **Surveiller-Activités-Appareils :** Conditions : Le contenu inclut → Étiquettes de confidentialité → sélectionner les 3 étiquettes suivantes (architecture OU) :

| **Étiquette** | **Justification** |
| --- | --- |
| **3b - Direction - Documents** | Documents Direction chiffrés AES-256 |
| **4 - RH-Confidentiel** | Documents RH chiffrés AES-256 |
| **5 - Finances-Confidentiel** | Documents Finances chiffrés AES-256 |

ℹ️** Avantage des étiquettes vs SIT : **un fichier étiqueté 4 - RH-Confidentiel sera protégé même s’il ne contient pas de pattern AVS ou IBAN détectable. Les étiquettes couvrent aussi les fichiers chiffrés dont le contenu ne peut pas être analysé par les SIT. Petit plus : ajouter également les SIT AVS-Suisse-PME, Swiss Social Security Number AHV, IBAN et Medical-RH-Suisse dans le même groupe (opérateur N’importe lequel), cela étend la protection aux fichiers non étiquetés contenant des données sensibles. Dans l’interface, ajoutez-les via Ajouter + Types d’infos confidentielles dans le groupe « Surveiller-Activités-Appareils ».

7. Pas de condition « partage externe » :  cette condition est incompatible avec l’emplacement Appareils. L’Endpoint DLP surveille toutes les activités sur l’appareil indépendamment de la destination. Dans section Actions → sélectionnez Auditer ou restreindre les activités sur les appareils → Appliquer des restrictions à une activité spécifique → configurez le tableau suivant :

| **Activité** | **Phase 1 (sem. 1-2)** | **Phase 2 (après validation)** |
| --- | --- | --- |
| Téléchargement vers service cloud / navigateur non autorisé | Auditer uniquement | Bloquer avec remplacement |
| Copier dans le presse-papiers | Auditer uniquement | Bloquer avec remplacement |
| Copier sur USB amovible | Auditer uniquement | Bloquer avec remplacement |
| Copier vers partage réseau | Auditer uniquement | Auditer uniquement |
| Imprimer | Auditer uniquement | Bloquer avec remplacement |
| Copier via RDP (Bureau à distance) | Auditer uniquement | Auditer uniquement |

8. Alertes et rapports : niveau Moyen • Envoyer une alerte à admin@demo.ch → Activity Explorer logge automatiquement chaque événement (DLPRuleMatch). → Enregistrer la règle.

9. Mode de la stratégie → **Exécuter la stratégie en mode simulation** → Soumettre. Après 1-2 semaines, consulter l’Activity Explorer → valider l’absence de faux positifs → activer la stratégie et passer les activités clés en Phase 2.

ℹ️** Bloquer avec remplacement : **l’action est bloquée mais l’utilisateur peut cliquer « Remplacer » et saisir une justification professionnelle pour outrepasser le blocage. La justification + l’identité de l’utilisateur sont enregistrées dans l’Activity Explorer. C’est l’équilibre idéal pour une PME : protection réelle sans blocage total des cas légitimes.

ℹ️** Impression : comportement Phase 2 **l'action Bloquer avec remplacement affiche une notification : l'utilisateur doit cliquer sur Remplacer et saisir une justification. L'impression repart ensuite automatiquement. Ce n'est pas un blocage définitif : c'est une friction consciente qui génère une trace horodatée dans l'Activity Explorer (utilisateur, justification, fichier concerné).

## 9.2 Contrôles IA dans Edge for Business

La protection contre l’upload de fichiers sensibles vers des sites d’IA générative (ChatGPT, Gemini, Perplexity...) se configure directement dans la règle **Surveiller-Activites-Appareils** de la politique DLP-Protection-Endpoint (section 9.1), elle ne nécessite pas de configuration séparée dans les paramètres Purview.

Configurer la restriction des sites IA dans la règle Endpoint

Dans la règle Surveiller-Activités-Appareils → section Domaine de service et activités du navigateur → deux options à activer :

1. Téléchargement vers un domaine de service cloud restreint :** **cocher → + Choisir différentes restrictions pour les domaines de service sensibles → Ajouter un groupe → sélectionner Sites web d’IA générative (groupe prédéfini Microsoft, contient automatiquement ChatGPT, Gemini, Perplexity, Mistral, etc.) → Action : Auditer uniquement (Phase 1) puis **Bloquer avec remplacement** (Phase 2 après validation) → Enregistrer.

2. Coller dans les navigateurs pris en charge : Phase 1 : Auditer uniquement. Phase 2 : Bloquer avec remplacement : bloque le copier-coller de contenu sensible dans un navigateur vers un site IA.

⚠️ Couverture navigateur : « Coller dans les navigateurs » fonctionne selon le navigateur. Edge (version actuelle) natif, aucune extension requise. Chrome et Firefox nécessitent l’extension Microsoft Purview installée sur le poste. Sans extension, le copier-coller passe sans interception sur ces navigateurs. Les navigateurs non listés (Brave, Opera, etc.) ne sont pas couverts, à bloquer via Web Content Filtering si nécessaire. Note terrain : l'extension Firefox est en version Beta et son comportement peut être instable, des uploads ou copier-coller peuvent passer sans interception malgré une installation correcte. Edge reste le navigateur de référence pour une protection Endpoint DLP fiable.

ℹ️** Le groupe « Sites web d’IA générative » est maintenu automatiquement par Microsoft : **vous n’avez pas à lister manuellement ChatGPT, Gemini, etc. Microsoft met à jour la liste au fil des nouvelles plateformes IA. Cette configuration s’applique à tous les navigateurs (Edge, Chrome, Firefox) sur les appareils intégrés dans Purview.

### 9.2.1 Déploiement de l’extension navigateur Microsoft Purview via Intune

L’extension navigateur Microsoft Purview est requise pour que Chrome et Firefox interceptent les activités copier-coller et upload. Microsoft Edge (Windows) intègre nativement les contrôles Purview Endpoint DLP pour les activités navigateur ; aucune extension supplémentaire n’est requise. L’extension ne s’installe pas automatiquement même sur un poste géré Intune — un profil de configuration doit être créé explicitement.

**Chrome — via Intune Settings Catalog**

Intune → Appareils → Profils de configuration → **Créer** → Windows 10 et ultérieur → Settings Catalog → chercher *Google Chrome* → Extensions → **Configure the list of force-installed apps and extensions** → ajouter :

echcggldkblhodogklpincgchnpgcdco;https://clients2.google.com/service/update2/crx

**⚠️ **L’URL clients2.google.com/service/update2/crx est obligatoire, sans elle l’extension peut ne pas s’installer ou rester en état incohérent.

**Firefox via ADMX + OMA-URI (Settings Catalog non supporté)**

L’approche Settings Catalog n’est pas valide pour Firefox dans le cas de Purview. Firefox ne supporte pas l’installation forcée des extensions via le Settings Catalog Intune. Le déploiement doit obligatoirement passer par les policies Firefox (ADMX) et le paramètre ExtensionSettings via OMA‑URI.

Méthode correcte (Microsoft + terrain) :

Phase 1 — Importer les ADMX Firefox dans Intune : téléchargez le fichier ADMX officiel Firefox, importez-le dans Intune → Appareils → Profils de configuration → Paramètres de configuration.

Phase 2 — Créer un profil Custom (OMA-URI) avec ExtensionSettings contenant :

{

  "microsoft.defender.browser_extension.native_message_host@microsoft.com": {

    "installation_mode": "force_installed",

    "install_url": "https://github.com/microsoft/purview/raw/main/endpointDLP/browser_extension/prod-1.1.0.212.xpi",

    "updates_disabled": false

  }

}

**Vérification post-déploiement** — sur le poste cible après sync Intune (env. 15 min) : Chrome → chrome://extensions — Firefox → about:addons — vérifier la présence de « Microsoft Purview Extension » et son statut activé.

⚠️ Risque résiduel : Le presse-papiers partagé VM/hôte (vmtoolsd.exe) sur les postes exécutant une VM (VMware Tools), le presse-papiers peut transiter via vmtoolsd.exe entre la VM et l’hôte physique. Endpoint DLP détecte cet événement (FileCopiedToClipboard, application vmtoolsd.exe) mais ne peut pas bloquer ce vecteur il opère au niveau hyperviseur, hors portée de Purview.

## 9.3 Purview AI Hub et DSPM for AI, tableau de bord unifié

Concept : voir tout ce qui touche l’IA dans votre tenant

L’AI Hub est le tableau de bord central dans Purview qui regroupe toutes les informations liées à l’utilisation de l’intelligence artificielle dans votre organisation. Il centralise trois types d’informations : quels utilisateurs utilisent Copilot M365 et quelles données ils lui soumettent, quels agents Copilot Studio sont actifs, et quelles interactions avec des IA externes (ChatGPT, etc.) ont été détectées via Endpoint DLP.

DSPM for AI (Data Security Posture Management for AI) est la partie analytique de l’AI Hub, elle évalue la posture de sécurité de votre tenant vis-à-vis de l’IA et signale les risques : fichiers sur-partagés accessibles à Copilot, politiques manquantes, utilisateurs sans étiquettes qui interagissent avec des agents. Vous l’avez déjà partiellement configuré en section 5.1. Cette section complète cette configuration avec les nouvelles capacités liées à Endpoint DLP et Edge.

Procédure : Accéder à l’AI Hub et activer les alertes

1. Dans Purview → menu de gauche → sécurité des données IA. Si vous ne voyez pas ce menu, cherchez AI Hub ou DSPM pour l’IA, le nom varie selon la version du portail. Attendez au moins 24h après avoir intégré un appareil (section 9.1) pour que les premières données apparaissent.

2. Onglet Vue d’ensemble, vous voyez le résumé des activités IA des 30 derniers jours : nombre d’interactions Copilot, nombre d’alertes DLP liées à l’IA, fichiers sensibles accédés via Copilot. C’est votre indicateur de santé global.

3. Onglet Activités de l’IA → filtrez par Type d’activité → IA générative, vous voyez les interactions avec des IA externes détectées par Endpoint DLP. Chaque ligne montre : l’utilisateur, l’appareil, le site IA utilisé, et si une donnée sensible a été soumise.

4. Onglet Recommandations → DSPM for AI liste les actions prioritaires pour améliorer votre posture. Chaque recommandation est cliquable et vous amène directement à la configuration à faire. Traitez en priorité les recommandations marquées Critique ou Risque élevé.

5. Pour configurer une alerte email sur les incidents IA → Purview → Prévention des pertes de données → Alertes → + Créer une règle d’alerte → choisissez les politiques DLP concernées → fréquence : Dès que possible → Email des destinataires : votre adresse admin. Vous recevrez une alerte à chaque incident détecté sur un appareil intégré.

Lien nLPD : pourquoi cette configuration compte

| **Fonctionnalité** | **Ce que ça couvre (nLPD)** | **Article nLPD** |
| --- | --- | --- |
| Endpoint DLP | Mesures techniques contre la fuite physique de données (USB, impression) | Art. 8 al. 3 |
| Contrôles IA Edge for Business | Contrôle du transfert de données vers des applications IA non gérées | Art. 8 al. 3 + Art. 9 |
| AI Hub / DSPM for AI | Preuve de surveillance et traçabilité pour notification PFPDT en cas d’incident | Art. 24 |

Checklist de validation section 9

| **✓** | **Action** | **Statut** |
| --- | --- | --- |
| ☐ | Appareil(s) Windows intégré(s) dans Purview via script ou Intune | à faire |
| ☐ | Politique DLP-Protection-Endpoint créée (politique séparée, incompatibilité Appareils + "Du contenu est partagé avec des personnes extérieures à mon organisation) | à faire |
| ☐ | Applications IA non gérées configurées comme restriction dans la règle DLP | à faire |
| ☐ | AI Hub accessible dans Purview, premières activités IA visibles | à faire |
| ☐ | Alerte email configurée sur les incidents DLP appareils | à faire |

---

[← Partie 8 — Fonctionnalités avancées Purview](09-fonctionnalites-avancees.md) | [Partie 10 — Gouvernance Copilot Studio et Power Platform →](11-power-platform.md)
