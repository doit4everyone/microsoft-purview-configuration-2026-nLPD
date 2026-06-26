---
title: "Annexes | Configuration Microsoft Purview 2026"
description: "Annexes du guide Microsoft Purview 2026 : glossaire, articles nLPD, SIT personnalisés suisses (AVS, données médicales), différences POC vs Production, RACI et escalade, levée d'anonymat IRM, gestion des comptes et rôles, registre des activités de traitement (art. 12 nLPD)."
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

# Annexes

## Annexe A — Glossaire Purview

| **Terme** | **Définition en langage simple** |
| --- | --- |
| Étiquette de sensibilité | Marqueur attaché à un document qui définit son niveau de confidentialité et les protections associées (chiffrement, filigrane, restrictions d'accès) |
| Auto-labelling service | Purview analyse automatiquement les fichiers SharePoint et OneDrive de manière asynchrone, selon les capacités du service et les emplacements configurés et applique les étiquettes selon les règles configurées, sans qu’aucun utilisateur n’ouvre le fichier. Cette analyse n’est ni instantanée ni garantie exhaustive. |
| DLP | Data Loss Prevention, règles qui bloquent ou avertissent quand des données sensibles risquent de fuir en dehors du périmètre autorisé |
| Activity Explorer | Tableau de bord Purview qui affiche toutes les activités liées aux données sensibles : labels appliqués, DLP déclenchées, accès Copilot |
| Content Explorer | Vue globale de tous les fichiers classifiés dans votre tenant, permet de voir quels fichiers contiennent des AVS ou IBAN |
| DSPM for AI | Data Security Posture Management : surveillance spécifique des interactions avec Copilot M365, Copilot Studio et Azure AI Foundry |
| Mode simulation | Mode d'une politique DLP ou auto-labelling où les règles s'évaluent sans bloquer ni modifier quoi que ce soit. Pour tester avant d'activer |
| Conseil de stratégie | Message d'avertissement visible par l'utilisateur dans Word ou Outlook quand une règle DLP est déclenchée, sans bloquer |
| Purview RBAC | Système de rôles propre à Purview, distinct des rôles Entra ID. Donne accès aux différentes sections du portail Purview. |
| Organization Management | Groupe de rôles Exchange Online qui contient le droit Audit Logs, indispensable pour activer l'audit via PowerShell |
| nLPD art. 8 | Obligation légale suisse de mettre en place des mesures techniques et organisationnelles pour protéger les données personnelles |

## Annexe B — Articles nLPD et Purview

| **Article nLPD** | **Obligation** | **Fonctionnalité Purview qui y répond** |
| --- | --- | --- |
| Art. 6 — Finalité | Données utilisées uniquement pour la finalité prévue | DLP + Labels empêchent l'utilisation non prévue |
| Art. 6 — Proportionnalité | Pas plus de données que nécessaire | DLP complète les mécanismes de protection en auditant et restreignant certains usages Copilot (partage, exfiltration), sans en être le contrôle principal. Ordre de priorité :  (1) Permissions M365, un utilisateur n’accède qu’à ce qu’il est autorisé à voir ;  (2) Chiffrement RMS, protection intrinsèque du document ;  (3) DLP Copilot, complète en bloquant certaines actions. Un utilisateur autorisé par RMS peut toujours interroger le contenu via Copilot, la DLP réduit le risque résiduel mais ne le supprime pas |
| Art. 8 — Sécurité | Mesures techniques et organisationnelles obligatoires | Chiffrement + DLP + Audit = MTO complètes |
| Art. 9 — Privacy by design | Protection intégrée dès la conception | Auto-labelling protège dès le dépôt du fichier |
| Art. 24 — Notification PFPDT | Notifier dès que possible (art. 24 nLPD) en cas de violation | Audit logs = preuves et chronologie complète |
| Art. 5 let. c — Données sensibles | Protection renforcée : AVS, santé, biométrie | Labels confidentiels + DLP types sensibles suisses |
| Art. 62 — Secret professionnel | Obligation de confidentialité même sans intention | DSPM audite tous les accès Copilot aux données |

Les mesures mises en place visent à réduire les usages non autorisés, dissuader les comportements à risque et assurer la traçabilité des accès. Aucune solution technique ne permet d’empêcher de manière absolue la reproduction visuelle d’une information par un utilisateur autorisé.

## Annexe C — SIT personnalisé AVS-Suisse-PME

Le type d’information sensible (SIT) natif de Microsoft « Swiss Social Security Number AHV » valide le checksum EAN-13 en confiance élevée et exige des mots-clés de proximité. Dans un tenant PME suisse avec des documents en plusieurs langues, cette contrainte provoque des non-détections. Le SIT personnalisé AVS-Suisse-PME utilise un regex direct sur le format suisse et des mots-clés dans les 4 langues nationales plus l’anglais, sans dépendance au checksum.

### C.1 : Regex et configuration du modèle

**Expression régulière (regex) : **756\.\d{4}\.\d{4}\.\d{2}

Ce regex détecte exactement le format suisse avec points obligatoires : 756.XXXX.XXXX.XX. Le préfixe 756 est le code pays suisse. Les points échappés (\.) forcent le séparateur point, une séquence de chiffres collés comme 7561234567890 n’est pas détectée.

**Paramètres du modèle Purview : **Type de correspondance = Correspondance de chaîne | Niveau de confiance = Moyen | Nombre d’instances min = 1.

### C.2, Mots-clés multilingues (4 langues nationales + anglais)

Ces mots-clés sont des éléments de soutien, ils augmentent le niveau de confiance de la détection quand ils apparaissent dans une fenêtre de 300 caractères autour du numéro détecté. Ils ne ralentissent pas l’analyse et n’affectent pas les performances des politiques DLP ou d’auto-labelling.

ℹ️ **Astuce copier-coller : **copiez les mots-clés d’une ligne du tableau ci-dessous dans un document Word vierge, puis faites Ctrl+H (Remplacer) : dans Rechercher saisissez «  /  » (espace-slash-espace), dans Remplacer par saisissez « ^p » (caret-p, qui représente un saut de paragraphe dans Word), puis cliquez sur Tout remplacer. Vous obtenez la liste prête à coller directement dans l’interface Purview, un mot-clé par ligne.

⚠️ Notepad++ : activez le mode Expression régulière et utilisez \\n dans le champ Remplacer par au lieu de Ctrl+Entrée.

| **Langue** | **Mots-clés (un par ligne dans Purview)** |
| --- | --- |
| **Français** | AVS / numéro AVS / No. AVS / N° AVS / assurance vieillesse / assurance sociale / numéro d’assurance |
| **Allemand** | AHV / AHV-Nummer / AHV-Nr / Altersvorsorge / Sozialversicherung / Versicherungsausweis / AHV-Ausweis |
| **Italien** | AVS / numero AVS / No. AVS / assicurazione vecchiaia / assicurazione sociale / numero d’assicurazione |
| **Romanche** | numer AVS / AVS / assicuranza da vegliadetgna / assicuranza sociala / document d’assicuranza / attest d’assicuranza |
| **Anglais** | AHV number / AVS number / Swiss social security / social security number CH / Swiss insurance number |

### C.3 : Procédure de création dans Purview

1. Purview → Protection des données → Classifieurs → Types d’informations sensibles → + Créer un type d’informations sensibles.

2. Nom : AVS-Suisse-PME | Description : Détection du numéro AVS suisse format 756.XXXX.XXXX.XX — 4 langues nationales + anglais. → Suivant.

3. + Ajouter un modèle → Niveau de confiance : Moyen → + Ajouter un élément principal → Expression régulière → ID : regex-AVS-Suisse → collez le regex :

756\.\d{4}\.\d{4}\.\d{2}

Sélectionnez Correspondance de chaîne (pas Correspondance de mot). Cliquez sur Terminé.

4. Dans le même modèle → + Ajouter des éléments de soutien → Liste de mots-clés → ID : recherche-AVS-Suisse → Groupe de mots-clés #1 (Ne respecte pas la casse) → saisissez un mot-clé par ligne en reprenant la liste du tableau C.2 ci-dessus pour toutes les langues. Cliquez sur Terminé.

5. Cliquez sur Suivant → testez avec un numéro de votre choix au format 756.XXXX.XXXX.XX → vérifiez que la correspondance est détectée → Suivant → Créer.

### C.4 : Remplacement dans les politiques existantes

Une fois le SIT AVS-Suisse-PME créé, remplacez Swiss Social Security Number AHV par AVS-Suisse-PME dans les 4 emplacements suivants :

1. Auto-labelling Auto-Label-RH-Sensible → Règle-AVS-Suisse → remplacez le SIT natif par AVS-Suisse-PME.

2. Auto-labelling Auto-Label-RH-Sensible → Règle-Medical → Condition unique → remplacez le SIT natif par AVS-Suisse-PME.

3. DLP DLP-Protection-nLPD-demo → règle Bloquer-AVS-Externe → condition Le contenu inclut → remplacez le SIT natif par AVS-Suisse-PME.

4. DLP DLP-Protection-nLPD-demo → règle Avertir-Donnees-Sensibles-Interne → condition Le contenu inclut → remplacez le SIT natif par AVS-Suisse-PME.

**ℹ️**** INFO — **Après remplacement, relancez la simulation des politiques d’auto-labelling et vérifiez dans l’onglet Éléments correspondants que vos fichiers de test apparaissent. Le SIT personnalisé détecte tout numéro au format 756.XXXX.XXXX.XX indépendamment de la validité du checksum.

### C.5 : SIT personnalisé Medical-RH-Suisse

Le SIT natif Microsoft « All Medical Terms And Conditions » est trop large pour un contexte RH suisse : il détecte des termes médicaux génériques sans contexte nominatif, générant des faux positifs sur des documents qui ne contiennent pas de données personnelles de santé. Le SIT personnalisé Medical-RH-Suisse se limite aux termes directement liés aux arrêts maladie, incapacité de travail et certificats médicaux dans le contexte RH suisse, en 4 langues nationales.

#### C.5.1 : Mots-clés RH médicaux (4 langues nationales)

Ces mots-clés ciblent les documents RH contenant des informations médicales nominatives. Ils ne déclenchent pas sur des termes médicaux génériques ou scientifiques. Saisissez un mot-clé par ligne dans Purview.

ℹ️ **Astuce copier-coller : **copiez les mots-clés d’une ligne du tableau ci-dessous dans un document Word vierge, puis faites Ctrl+H (Remplacer) : dans Rechercher saisissez «  /  » (espace-slash-espace), dans Remplacer par saisissez « ^p » (caret-p, qui représente un saut de paragraphe dans Word), puis cliquez sur Tout remplacer. Vous obtenez la liste prête à coller directement dans l’interface Purview, un mot-clé par ligne.

⚠️ Notepad++ : activez le mode Expression régulière et utilisez \\n dans le champ Remplacer par au lieu de Ctrl+Entrée.

| **Langue** | **Mots-clés (un par ligne dans Purview)** |
| --- | --- |
| **Français** | arrêt maladie / certificat médical / incapacité de travail / conge maladie / aptitude au travail / inapte au travail / convalescence / accident de travail / maladie professionnelle |
| **Allemand** | Krankmeldung / ärztliches Zeugnis / Arbeitsunfähigkeit / Krankheitsfall / Arbeitsunfähigkeitszeugnis / Genesungsurlaub / Berufsunfall / Berufskrankheit / arbeitsunfähig |
| **Italien** | certificato medico / incapacità lavorativa / congedo malattia / idoneità al lavoro / inidoneità al lavoro / convalescenza / infortunio professionale / malattia professionale |
| **Anglais** | sick leave / medical certificate / incapacity to work / fit for work / unfit for work / occupational accident / occupational disease / sick note / medical leave |
| **Romanche** | annunzia da malsogna / certificat medical / incapacità da lavurar / congedi da malsogna / abilita da lavurar / nun abel da lavurar / convalescenza / accident da lavur / malsogna professiunala |

#### C.5.2 : Procédure de création dans Purview

1. Purview → Protection des données → Classifieurs → Types d’informations sensibles → **+ Créer un type d’informations sensibles**.

2. Nom : **Medical-RH-Suisse** | Description : Détection des données médicales RH (arrêts maladie, certificats, incapacité de travail), 4 langues nationales + anglais. | Suivant.

3. **+ Ajouter un modèle** → Niveau de confiance : **Moyen** → **+ Ajouter un élément principal** → **Liste de mots-clés** → ID : **medical-rh-suisse** → Groupe de mots-clés 1 (Ne respecte pas la casse) → saisissez un mot-clé par ligne en reprenant la liste du tableau C.5.1 ci-dessus pour toutes les langues → Cliquez sur Terminé.

4. Cliquez sur Suivant → testez avec un terme de la liste (ex : « arrêt maladie ») → vérifiez que la correspondance est détectée → Suivant → **Créer**.

5. Une fois créé, remplacez *All Medical Terms And Conditions* par **Medical-RH-Suisse** dans les emplacements suivants : Auto-labelling **Auto-Label-RH-Sensible** → Règle-Medical → Condition unique → remplacez *All Medical Terms And Conditions* par Medical-RH-Suisse.

Communication Compliance et IRM : les règles qui référencent Medical-RH-Suisse sont déjà correctement configurées si vous avez créé ce SIT avant de configurer ces sections. Si vous l’avez créé après, vérifiez que le SIT apparaît dans les conditions de chaque règle concernée.

## Annexe E — Différences POC vs Production

Ce guide est conçu pour un environnement POC / LAB avec Microsoft 365 Business Premium + Purview Suite. Avant tout déploiement en production, appliquez les ajustements suivants. Toute déviation non documentée constitue un risque d’audit nLPD.

| **Point** | **Configuration POC (ce guide)** | **Configuration Production obligatoire** |
| --- | --- | --- |
| **Admin dans étiquettes chiffrées** | admin@demo.ch ajouté aux autorisations des étiquettes 3, 4, 5 pour faciliter les tests | **Supprimer admin des autorisations. Accès via Super User RMS (section 2.6) uniquement, sur demande justifiée et tracée.** |
| **Politiques DLP en simulation** | Mode simulation recommandé 1 semaine | **Minimum 2 semaines de simulation avec revue métier (sponsor désigné) avant activation. Documenter les exceptions acceptées.** |
| **Endpoint DLP / Edge IA** | Auditer 7 jours puis bloquer | **Minimum 2 à 4 semaines audit-only. Désigner un sponsor métier. Documenter les usages légitimes à exclure avant blocage.** |
| **Power Platform DLP** | Blocage connecteurs sans procédure rollback | **Exporter tous les flux Power Automate avant activation. Définir point de restauration. Tester sur environnement de développement avant prod.** |
| Trainable Classifiers | Non disponible (Business Premium + Purview Suite) | Privilégier SITs personnalisés (Annexes C et D). Trainable Classifiers disponibles avec M365 E5 si évolution de licence. |
| Restriction override DLP par groupe (Règle 1) | Override ouvert à tous les utilisateurs. Restriction implicite assurée par les permissions SharePoint : seuls RH et Direction ont accès aux fichiers RH et peuvent donc déclencher l’override. La condition “expéditeur est membre de” n’est pas disponible en Business Premium. | **Vérifier que les permissions SharePoint RH sont strictement limitées à RH + Direction (principe du moindre ****privilège  Partie**** 11). Si évolution vers M365 E3/E5 : créer groupe DLP-Exceptions et utiliser la condition “L’expéditeur est membre de” pour architecture à deux règles (1 blocage strict tous users, 1b override groupe autorisé).** |

## Annexe F — RUN & Responsabilités (PME suisse)

**Exigence audit — responsabilité nominative**

Dans cette annexe, les rôles **Admin IT**, **Direction** et **Juridique/DPO** doivent correspondre à une personne physique nommée (Nom/Prénom) dans le registre interne. Un rôle générique seul n’est pas suffisant en audit PFPDT l’auditeur demandera qui, concrètement, est responsable et comment le prouver (art. 12 nLPD - registre des activités de traitement).

Sans organisation associée aux mesures techniques, Purview devient une boîte noire que personne ne surveille. Cette annexe définit le minimum opérationnel requis pour une PME suisse sous nLPD. Le PFPD peut demander lors d’un audit la preuve que ces contrôles sont effectivement exercés (art. 12 nLPD - registre des activités de traitement).

### F.1 : RACI Purview PME

| **Tâche** | **Responsable** | **Fréquence** | **Action si problème** |
| --- | --- | --- | --- |
| Vérifier Activity Explorer — alertes DLP non traitées | Admin IT (personne nommée) | Hebdomadaire | Traiter dans les 48h, escalade Direction si violation nLPD |
| Vérifier Content Explorer — documents non étiquetés | Admin IT | Mensuel | Campagne d’étiquetage manuel (section 2.7) |
| Vérifier simulation auto-labelling — dérives détectées | Admin IT | Mensuel | Corriger la règle concernée dans Purview |
| Auditer les accès Super User RMS — utilisation Break-glass | Direction + Admin IT | Chaque activation | Tracer dans registre incidents nLPD art. 12 |
| Revoir configuration après changement (nouveau site SP, agent Copilot, flux PP) | Admin IT + responsable concerné | Systématique | Vérifier impact sur auto-labelling et DLP (section 7.3) |

### F.2 — Escalade et incidents nLPD

**Délai de notification PFPD : **dès que possible après la découverte d’une violation de données personnelles (art. 24 nLPD). Toute alerte DLP de gravité élevée non résolue dans les 48h doit être traitée comme un incident potentiel nLPD.

**Chaîne d’escalade : **Alerte Purview → Admin IT (48h) → Direction (si violation confirmée) → Conseil juridique (si notification PFPD requise) → PFPD (dès que possible - art. 24 nLPD).

**ℹ️**** INFO : **Ce tableau RACI doit être intégré dans le registre des activités de traitement de l’organisation (art. 12 nLPD). Nommez une personne physique responsable pour chaque ligne. « Admin IT » seul ne suffit pas en audit PFPD.

## Annexe G — Procédure de levée d’anonymat IRM (obligatoire nLPD)

L’Insider Risk Management de Purview anonymise les utilisateurs par défaut dans les alertes et rapports. La levée d’anonymat (identification de l’utilisateur concerné) est une action irréversible qui doit être encadrée par un processus écrit. Sans ce processus, la conformité nLPD s’effondre. Ce document constitue la procédure officielle de l’organisation Axonix SA.

Prérequis : Information préalable des employés (art. 12 nLPD)

Avant toute activation des stratégies IRM, les employés concernés doivent être informés de l’existence du système de détection, de sa finalité (prévention des fuites de données), des types de signaux collectés, et des droits d’accès à leurs propres données. Cette information doit figurer dans le règlement du personnel ou la charte informatique, signée par chaque employé. Sans cette information préalable documentée, l’activation des stratégies IRM est juridiquement fragile en droit suisse (nLPD art. 12 + art. 26 OLT3).

Processus de traitement d’une alerte IRM

| **Étape** | **Action** | **Responsable** | **Trace requise** |
| --- | --- | --- | --- |
| **1** | Détection automatique — Purview génère une alerte anonymisée | Automatique (Purview) | Alerte enregistrée dans Activity Explorer |
| **2** | Analyse anonymisée — Admin IT ou Responsable Sécurité évalue la gravité sans lever l’anonymat | Admin IT / Resp. Sécurité | Note d’analyse datée et signée |
| **3** | Qualification — l’alerte est-elle un faux positif ? un incident mineur ? un incident grave ? | Admin IT + Direction | Décision documentée avec justification |
| **4** | Décision de levée d’anonymat — uniquement si l’incident est qualifié grave et que la levée est **réellement nécessaire** pour l’investigation | Direction + Conseil juridique | Autorisation écrite signée Direction + date |
| **5** | Levée d’anonymat dans Purview — Gestion des risques internes → Alertes → sélectionner l’alerte → action Afficher l’identité | Admin IRM (rôle Insider Risk Analyst) | Action loggée automatiquement dans Audit Log Purview |
| **6** | Clôture — archivage du dossier complet (alerte, analyse, décision, levée d’anonymat si effectuée) | Admin IT + Direction | Dossier archivé 5 ans minimum (prescription civile suisse) |

Principe fondamental

**La levée d’anonymat est l’exception, pas la règle. **La grande majorité des alertes IRM doit pouvoir être qualifiée et clôturée sans levée d’anonymat , faux positif, formation utilisateur, correction technique. La levée d’anonymat n’est justifiée que si (a) l’incident est confirmé grave, (b) l’identification est indispensable à l’investigation, et (c) une décision écrite de la Direction est obtenue préalablement. Toute levée d’anonymat sans autorisation écrite constitue une violation de la protection de la personnalité au sens de l’art. 26 OLT3 et expose l’employeur à une responsabilité civile.

## Annexe H — Gestion des comptes et rôles Purview

Un auditeur PFPDT attend : (1) séparation des rôles critiques, (2) traçabilité des actions sensibles, (3) accès limité par fonction, (4) absence de compte fourre-tout. Ce modèle respecte l’art. 8 nLPD et l’OLT3 art. 26 avec 4 comptes couvrant 100 % des besoins Purview.

### Compte 1 : Admin IT Purview (exploitation quotidienne)

Compte : **admin-it@demo.ch** | État : Actif | MFA : Obligatoire | Licence : Business Premium + Purview Suite.

**Rôles Entra ID : **Administrateur de la conformité, Administrateur de la sécurité, Administrateur IA (si DSPM/Copilot).

**Rôles Purview RBAC : **Compliance Administrator, Data Security AI Admins, Data Security Viewers, eDiscovery Manager (Gestionnaire + Administrateur), Communication Compliance Admin, Insider Risk Management Analysts.

**Exchange Online : **Organization Management (audit uniquement). **❌ Pas de Super User RMS. ❌ Pas de Global Admin (sauf POC).**

### Compte 2 : Break-glass Chiffrement RMS (urgence uniquement)

Compte : **rms-breakglass@demo.ch** | Type : cloud-only, non nominatif | État : **❌ Désactivé sauf incident** | MFA : FIDO2 ou TOTP offline | Licence : aucune.

**Rôle unique : **Super User RMS (via PowerShell uniquement). **❌ Aucun rôle Entra ID administratif. **
**❌ Aucun rôle Purview. ❌ Aucun accès SharePoint.**

Activation temporaire uniquement, désactivation immédiate après usage. Chaque activation est tracée dans les journaux d’audit Entra ID (Journaux d’audit → Gestion des utilisateurs).

### Compte 3 — Audit / Lecture conformité (optionnel)

Compte : **audit-purview@demo.ch** | État : Actif | MFA : Obligatoire | Licence : Business Premium + Purview Suite.

**Rôles Purview RBAC : **Data Security Viewers, Content Explorer List Viewer.

⚠️ Ne pas attribuer **Content Explorer Content Viewer **: permet l’audit sans accès au contenu des fichiers (proportionnalité nLPD art. 6).

### Compte 4 : Direction / Responsable traitement (lecture exécutive)

Compte : membre de **GRP-Direction-Purview** | État : Actif | MFA : Obligatoire | Licence : Business Premium + Purview Suite.

**Rôle unique : **Data Security Viewers. **❌ Pas d’administration. ❌ Pas d’accès Content Explorer contenu.** Conforme art. 12 nLPD : le responsable supervise sans opérer.

### Transition depuis admin@demo.ch procédure de désactivation safe

Le compte admin@demo.ch utilisé pendant le POC cumule tous les rôles. Ce modèle est acceptable en phase de test mais doit être séparé avant mise en production.

**Étape 1 : Créer et valider admin-it@demo.ch : **attribuer tous les rôles du tableau Compte 1. Tester intégralement toutes les configurations Purview avec ce nouveau compte avant de continuer.

**Étape 2 : Transférer la licence et retirer les rôles Purview : **désattribuer la licence Business Premium + Purview Suite de admin@demo.ch et l’attribuer à admin-it@demo.ch. **⚠️ Important** : les rôles Purview survivent à la désactivation du compte, retirer manuellement les rôles Purview de admin@demo.ch après transfert.

**Étape 3 : Bloquer la connexion (pas supprimer) : **Entra ID → Utilisateurs → admin@demo.ch → Modifier → **Bloquer la connexion** → Enregistrer. Le compte reste présent avec Global Admin conservé comme break-glass de dernier recours. En cas d’urgence absolue : réactiver temporairement → effectuer l’action → désactiver immédiatement. Chaque réactivation est tracée automatiquement dans les journaux d’audit Entra ID.

### Synthèse : Lecture auditeur PFPDT par compte

**admin-it@demo.ch:** configuré ✔ | voit contenu ✔ (si assigné) | casse chiffrement ❌ → « Compte clairement identifié comme opérateur technique. OK. »

**rms-breakglass@demo.ch:** configuré ❌ | voit contenu ✔ (exception) | casse chiffrement ✔ → « Séparation des tâches respectée. Usage exceptionnel et traçable. »

**audit-purview@demo.ch:** configuré ❌ | voit contenu ❌ | casse chiffrement ❌ → « Audit sans sur-accès aux données. Très bien. »

**direction-purview@demo.ch:** configuré ❌ | voit contenu ❌ | casse chiffrement ❌ → « Responsable supervise sans opérer. Conforme art. 12 nLPD. »

**admin@demo.ch (après transition) :** connexion bloquée ❌ | Global Admin conservé ✔ | break-glass de dernier recours uniquement → « Compte historique sécurisé. Traçabilité garantie. Acceptable en petite structure. »

## Annexe I — Modèle de registre des activités de traitement (art. 12 nLPD)

L’art. 12 nLPD impose aux responsables du traitement de tenir un registre des activités de traitement. Ce registre doit permettre de démontrer la conformité en cas de contrôle PFPDT. Le modèle ci-dessous est minimal et adapté à une PME suisse utilisant Microsoft 365 avec Purview.

### I.1 : Informations générales de l’organisation

| **Champ** | **À compléter par l’organisation** |
| --- | --- |
| **Raison sociale** | [Nom de la société] |
| **Responsable du traitement (art. 5 let. ****j**** nLPD)** | [Nom, Prénom, Fonction] |
| **DPO / Conseiller en protection des données** | [Nom, Prénom ou Externe / Pas de DPO désigné] |
| **Date de dernière mise à jour** | [JJ/MM/AAAA] |

### I.2 : Activités de traitement (une ligne par activité)

Activité 1 : Gestion des ressources humaines et de la paie

| **Champ** | **Valeur — Axonix SA** |
| --- | --- |
| **Désignation de l’activité** | Gestion des ressources humaines et de la paie |
| **Finalité du traitement (art. 6 nLPD)** | Administrer les contrats de travail, calculer et verser les salaires, gérer les absences, les congés et les évaluations. |
| **Personnes concernées** | Employés actifs et anciens employés. |
| **Catégories de données traitées** | Numéro AVS, coordonnées personnelles, salaire, évaluations, données médicales (certificats médicaux, arrêts maladie), données bancaires (IBAN). |
| **Destinataires des données** | Service RH interne, fiduciaire / comptabilité externe, caisse AVS, assureur accident LAA, assureur maladie collective, Microsoft Corporation (sous-traitant, infrastructure M365, données hébergées dans des centres de données EU). |
| **Transfert hors Suisse (art. 16 nLPD)** | Oui, Microsoft Azure, région EU, garanties adéquates selon la décision du Conseil fédéral et les DPA Microsoft. |
| **Durée de conservation** | 10 ans après la fin du contrat de travail (CO art. 958f). Suppression automatique configurée via Purview - politique Retention-RH-10ans. |
| **Base légale (art. 31 nLPD)** | Exécution du contrat de travail (art. 31 al. 2 let. a nLPD). Pour les données de santé : traitement de données sensibles nécessaire à l’exécution du contrat de travail (art. 31 al. 2 let. a nLPD). |
| **Mesures de sécurité (art. 8 nLPD)** | Étiquette 4 - RH-Confidentiel (chiffrement AES-256, Azure RMS). Accès restreint : GRP-RH-Purview + GRP-Direction-Purview. Politique DLP-Protection-nLPD-demo (blocage partage externe AVS/IBAN). Rétention automatique 10 ans. MFA administrateurs actif. |

Activité 2 : Comptabilité et gestion financière

| **Champ** | **Valeur — Axonix SA** |
| --- | --- |
| **Désignation de l’activité** | Comptabilité générale, gestion des paiements fournisseurs et des encaissements clients. |
| **Finalité du traitement (art. 6 nLPD)** | Tenir la comptabilité légale, établir les états financiers, gérer les flux de trésorerie et satisfaire aux obligations fiscales. |
| **Personnes concernées** | Fournisseurs, clients (contacts de facturation), employés (notes de frais). |
| **Catégories de données traitées** | IBAN, coordonnées de facturation, montants des transactions, références de contrats. |
| **Destinataires des données** | Fiduciaire externe, banque, Administration fédérale des contributions (AFC), administrations fiscales cantonales, Microsoft Corporation (sous-traitant). |
| **Transfert hors Suisse (art. 16 nLPD)** | Oui, Microsoft Azure, région EU, garanties adéquates selon la décision du Conseil fédéral et les DPA Microsoft. |
| **Durée de conservation** | 10 ans (CO art. 958f, délai légal comptable). Suppression automatique configurée via Purview - politique Retention-Finances-10ans. |
| **Base légale (art. 31 nLPD)** | Obligation légale (CO art. 957 ss, droit comptable suisse). |
| **Mesures de sécurité (art. 8 nLPD)** | Étiquette 5 - Finances-Confidentiel (chiffrement AES-256, Azure RMS). Accès restreint : GRP-Finances-Purview + GRP-Direction-Purview. Règle DLP Bloquer-IBAN-Externe (blocage strict sans remplacement possible). Rétention automatique 10 ans. |

Activité 3 : Gestion de la relation client

| **Champ** | **Valeur — Axonix SA** |
| --- | --- |
| **Désignation de l’activité** | Gestion des contrats clients, suivi commercial et facturation. |
| **Finalité du traitement (art. 6 nLPD)** | Exécuter les contrats de prestation, assurer le suivi commercial, établir la facturation et répondre aux demandes clients. |
| **Personnes concernées** | Contacts clients (personnes physiques, contexte B2B principalement). |
| **Catégories de données traitées** | Nom, prénom, coordonnées professionnelles (email, téléphone, adresse), historique des commandes et contrats. |
| **Destinataires des données** | Aucun destinataire externe sauf instruction explicite du client ou obligation légale. Microsoft Corporation (sous-traitant). |
| **Transfert hors Suisse (art. 16 nLPD)** | Oui, Microsoft Azure, région EU, garanties adéquates selon la décision du Conseil fédéral et les DPA Microsoft. |
| **Durée de conservation** | Durée du contrat + 5 ans après échéance (délai de prescription ordinaire, CO art. 127). |
| **Base légale (art. 31 nLPD)** | Exécution du contrat (art. 31 al. 2 let. a nLPD). |
| **Mesures de sécurité (art. 8 nLPD)** | Étiquette 2 - Interne. Accès restreint par site SharePoint (permissions spécifiques). Politique DLP-Protection-nLPD-demo (surveillance partages externes). |

Activité 4 : Surveillance IT et journaux d’audit (Purview / IRM)

| **Champ** | **Valeur — Axonix SA** |
| --- | --- |
| **Désignation de l’activité** | Surveillance des comportements à risque, traçabilité des accès aux données sensibles, constitution de preuves en cas d’incident nLPD. |
| **Finalité du traitement (art. 6 nLPD)** | Détecter les tentatives d’exfiltration de données, surveiller les violations des politiques de sécurité et satisfaire aux obligations de traçabilité imposées par l’art. 24 nLPD. Ces traitements ne sont pas utilisés à des fins de surveillance systématique du rendement ou du comportement individuel. |
| **Personnes concernées** | Employés membres des groupes GRP-RH-Purview, GRP-Finances-Purview et GRP-Direction-Purview uniquement (principe de proportionnalité, art. 6 nLPD). |
| **Catégories de données traitées** | Journaux d’accès aux fichiers, alertes DLP (déclenchements de règles), comportements d’exfiltration détectés (téléchargements massifs, partages externes), interactions Copilot M365 avec des données sensibles. |
| **Destinataires des données** | Administrateur IT Purview uniquement. Identités anonymisées par défaut, lève d’anonymat encadrée (voir Annexe G). |
| **Transfert hors Suisse (art. 16 nLPD)** | Oui, Microsoft Azure, région EU, garanties adéquates selon la décision du Conseil fédéral et les DPA Microsoft. |
| **Durée de conservation** | 1 an (journaux d’audit Purview Suite). Suppression automatique à échéance par Microsoft. |
| **Base légale (art. 31 nLPD)** | Intérêt légitime de l’entreprise (art. 31 al. 1 nLPD). Obligation d’information préalable des employés satisfaite (art. 19 nLPD, règlement intérieur ou charte IT). Contraintes OLT3 art. 26 (interdiction de surveillance permanente) respectées par l’anonymisation par défaut. |
| **Mesures de sécurité (art. 8 nLPD)** | Stratégies IRM-Fuites-Données-PME et IRM-Violations-Sécurité (section 8.2). Anonymisation activée par défaut. Procédure de levée d’anonymat documentée (Annexe G). Accès aux alertes limité au rôle Insider Risk Management Analysts. |

Activité 5 : Transmission externe de documents

| **Champ** | **Valeur — Axonix SA** |
| --- | --- |
| **Désignation de l’activité** | Transmission de documents à des partenaires externes autorisés via le site SharePoint Transmission externe. |
| **Finalité du traitement (art. 6 nLPD)** | Permettre le partage ponctuel et contrôlé de documents avec des destinataires externes dans le cadre d’une relation contractuelle ou d’une obligation légale. |
| **Personnes concernées** | Destinataires externes désignés au cas par cas par la Direction ou le service RH. |
| **Catégories de données traitées** | Variable selon le document transmis. Peut inclure des données personnelles (contrats, avenants, correspondances RH) ou des données financières. |
| **Destinataires des données** | Partenaires externes identifiés au cas par cas (avocats, experts, clients, fournisseurs). Microsoft Corporation (sous-traitant). |
| **Transfert hors Suisse (art. 16 nLPD)** | Possible selon l’identité du destinataire externe — à vérifier au cas par cas. Infrastructure Microsoft : Azure EU, garanties adéquates. |
| **Durée de conservation** | 2 jours sur le site Transmission externe (suppression automatique, politique Retention-TransmissionExterne-2jours). Conservation originale maintenue dans le site source (RH : 10 ans, Finances : 10 ans) conformément à l’activité concernée. |
| **Base légale (art. 31 nLPD)** | Exécution du contrat (art. 31 al. 2 let. a nLPD) lorsque la transmission découle d’une obligation contractuelle ou légale. Le consentement explicite de la personne concernée (art. 31 al. 2 let. b nLPD) n’est invoqué que subsidiairement, lorsqu’aucune base contractuelle ou légale ne couvre la transmission. |
| **Mesures de sécurité (art. 8 nLPD)** | Étiquette 6 - Transmission-Externe (marquages visuels, impression autorisée pour le destinataire). Politique DLP-Protection-Transmission-Externe (traçabilité nLPD art. 6, alerte admin). Label par défaut de bibliothèque SharePoint (protection automatique des dépôts sans étiquette). Suppression automatique 48 heures. |

**Note pratique : **ce registre doit être tenu à jour à chaque modification significative des traitements (nouvel outil, nouveau prestataire, nouvelle catégorie de données). Il n’existe pas de formulaire officiel PFPDT, ce modèle est conforme aux exigences minimales de l’art. 12 nLPD. Conservez-le dans un emplacement contrôlé (ex. : site SharePoint Direction, accès restreint à Direction + DPO).

## Annexe J — Soutien au travail de documentation

Ce document est mis à disposition gratuitement.

Il est le fruit de plusieurs semaines de travail, incluant :

- des tests techniques terrain sur Microsoft 365 / Purview ;

- une validation méthodologique orientée PME ;

- une relecture avec prise en compte des exigences de la nLPD (Suisse).

Aucune contrepartie financière n’est requise pour l’utilisation de ce contenu.

À titre strictement optionnel, les personnes ou organisations souhaitant soutenir

la maintenance de cette documentation et ses évolutions futures peuvent le faire

via la plateforme Kofi :

👉 https://ko-fi.com/doit4everyone

**Ce soutien n’implique aucune obligation, aucun service associé et aucune différence**

**de**** traitement dans l’accès au contenu.**

.

---

[← Partie 11 — Gouvernance des accès SharePoint](12-sharepoint.md) | [Retour à l'index →](../)
