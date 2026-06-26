---
title: "Partie 8 — Fonctionnalités avancées Purview | Configuration Microsoft Purview 2026"
description: "Fonctionnalités avancées de Purview Suite : politiques de rétention, Insider Risk Management, eDiscovery Premium, Communication Compliance et gestionnaire de conformité."
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

# Partie 8 — Fonctionnalités avancées Purview

Cette partie présente quatre fonctionnalités avancées de Microsoft Purview qui complètent les protections configurées dans les parties précédentes. Elles sont toutes pertinentes pour la conformité nLPD et pour la préparation à la certification SC-401. Ces fonctionnalités ne sont pas licenciées individuellement : elles sont accessibles via Microsoft Purview Suite, disponible aussi bien pour Business Premium que pour E3 ou E5.

| **Fonctionnalité** | **Business Premium seul** | **Purview Suite (BP/E3) ou E5** | **Section** |
| --- | --- | --- | --- |
| Retention policies (rétention) | ⚠️ Standard uniquement | ✅ Inclus | 8.1 |
| Insider Risk Management | ❌ Non disponible | ✅ Inclus | 8.2 |
| eDiscovery Premium | ❌ Non disponible | ✅ Inclus | 8.3 |
| Communication Compliance | ❌ Non disponible | ✅ Inclus | 8.4 |

## 8.1 Retention Policies , Politiques de rétention Inclus dans Purview Suite

Concept

Une politique de rétention définit combien de temps un document ou un email doit être conservé et ce qui se passe quand ce délai expire (suppression automatique ou conservation archivée). C'est une obligation directe de la nLPD qui impose de ne pas conserver les données personnelles plus longtemps que nécessaire (art. 6 al. 4).

Lien nLPD

| **Type de donnée** | **Rétention recommandée** | **Base légale nLPD** |
| --- | --- | --- |
| Emails Exchange | 3 ans puis suppression | Art. 6 al. 4, proportionnalité |
| Docs RH (contrats, fiches salaire) | 10 ans (droit du travail CO) | Art. 6 al. 4 + CO art. 962 |
| Docs Finances (factures, comptabilité) | 10 ans (délai légal comptable) | CO art. 958f |
| Logs d’audit Purview | 1 an (inclus dans Purview Suite) | Art. 24, preuve incident |

Procédure de configuration

1. Purview → menu de gauche → Gestion du cycle de vie des données → Stratégies de rétention →
+ Nouvelle stratégie de rétention.

2. Page Nom : **Retention-RH-10ans** | Description : Rétention documents RH → **Suivant.**

3. Page Unités administratives : Répertoire complet → **Suivant.**

4. Page Type : sélectionnez **Statique** → **Suivant.**

5. Page Emplacements : activez **Sites SharePoint classiques et de communication** uniquement.
Cliquez sur **Modifier** → saisir l’URL du site dans le champ de recherche 
(ex : *https://demo.sharepoint.com/sites/RH*) → cliquer sur le **+** à côté de l’URL pour l’ajouter → **Terminé.** Désactivez Comptes OneDrive, Boîtes aux lettres Exchange, Teams et tous les autres emplacements. → **Suivant.**

6. Page Paramètres de rétention : **Conserver les éléments pendant une période spécifique** → 10 ans → Démarrer la période fondée sur : **Lorsque les éléments ont été modifiés en dernier lieu** → À la fin de la période : **Supprimer automatiquement les éléments** → **Suivant **→ **Soumettre.**

7. Répétez pour créer :

**Retention-Finances-10ans :** même procédure, saisir *https://demo.sharepoint.com/sites/Finances* + cliquer « + » → 10 ans → supprimer.

**Retention-Email-3ans :** emplacement **Boîtes aux lettres Exchange** → Modifier → Tous les utilisateurs → 3 ans → supprimer.

8. Créez une quatrième politique :

**Retention-OneDrive-3ans :** même procédure, emplacement **Comptes OneDrive** → Modifier → Tous les comptes d’utilisateur → 3 ans → Démarrer la période fondée sur : **Lorsque les éléments ont été modifiés en dernier lieu** → supprimer automatiquement.

9. Créez une cinquième politique :

**Retention-TransmissionExterne-2jours :** même procédure, emplacement Sites SharePoint classiques et de communication → Modifier → saisir *https://demo.sharepoint.com/sites/transmissionexterne* → cliquer **+** → Paramètres de rétention : **Conserver les éléments pendant une période spécifique** → **2 jours** → Démarrer la période fondée sur : **Lorsque les éléments ont été créés** → À la fin : **Supprimer automatiquement les éléments**. Les documents sont automatiquement supprimés 48 heures après leur création, qu’ils aient été consultés ou non.

ℹ️** ATTENTION : **la rétention de 2 jours est volontairement courte, le site Transmission externe est un sas de transit, pas un lieu de stockage. Informez les utilisateurs qu’ils doivent confirmer que le destinataire a bien reçu le document avant les 48 heures.

Cliquez sur Soumettre. Les politiques s’appliquent dans un délai variable (quelques minutes à plusieurs jours selon le tenant).

## 8.2 Insider Risk Management (Gestion des risques internes)
Inclus Business Premium + Purview Suite

Concept

L’Insider Risk Management (IRM) détecte les comportements à risque des utilisateurs internes, téléchargements massifs de fichiers, envois vers des destinations non autorisées, accès inhabituels, exfiltration vers des services cloud personnels. Contrairement à la DLP qui bloque des règles précises, l’IRM analyse les patterns de comportement sur la durée et génère des alertes sur les cas suspects. L’IRM est inclus dans Microsoft 365 Business Premium + Purview Suite aucune licence E5 requise.

Cadre nLPD obligatoire avant activation

En droit suisse, la surveillance des employés est encadrée par l’art. 26 OLT3 (interdiction de la surveillance permanente) et le nLPD art. 6 (proportionnalité). L’IRM est légalement défendable uniquement si trois conditions sont remplies simultanément : (1) les employés concernés sont informés de l’existence de la surveillance (information préalable : règlement du personnel ou charte IT), (2) l’anonymisation est activée par défaut dans Purview, (3) un processus écrit encadre la levée d’anonymat. Sans ces trois éléments, la stratégie est juridiquement fragile. Voir Annexe G : Procédure de levée d’anonymat IRM.

Stratégies IRM recommandées pour une PME suisse

| **Stratégie** | **Modèle Purview** | **Finalité** |
| --- | --- | --- |
| **Socle PME (obligatoire)** | Fuites de données | Détecter les anomalies d’exfiltration proportionné, neutre juridiquement |
| **IA / Copilot (optionnel)** | Utilisation à risque de l’IA (préversion, Nouveau) | Encadrer les usages Copilot / IA générative. Stratégie séparée obligatoire |
| **Sécurité (optionnel avancé)** | Violations de stratégie de sécurité | Désactivation antivirus, contournement DLP moins intrusif que la surveillance RH |

Prérequis : Affecter les rôles IRM obligatoires

Sans affectation de rôle IRM, les stratégies fonctionnent et génèrent des alertes, mais celles-ci sont invisibles dans l’interface c’est un piège classique lors de la première configuration. Chemin : Purview → Gestion des risques internes → Paramètres → Groupes de rôles → sélectionnez le rôle → Modifier → Ajouter des membres → ajoutez votre compte admin → **Enregistrer.**

**Insider Risk Management Analysts **: consulte et trie les alertes anonymisées. Ne peut pas lever l’anonymat. Rôle recommandé pour 95 % des cas. Conforme Annexe G.

**Insider Risk Management Investigators **: accès complet incluant la levée d’anonymat. À restreindre strictement. voir Annexe G.

Procédure d’affectation, à faire immédiatement après activation IRM :

1. Purview → Gestion des risques internes → Paramètres (roue dentée en bas à gauche) → Groupes de rôles.

2. Sélectionnez **Insider Risk Management Analysts** → Modifier → Ajouter des membres → ajoutez admin@demo.ch → **Enregistrer**.

3. Répétez pour **Insider Risk Management Investigators** si la levée d’anonymat est requise en production (voir Annexe G). Note : sans cette affectation, le bandeau orange « Affecter des autorisations » reste visible dans l’interface IRM ,ce n’est pas un dysfonctionnement mais l’indication qu’aucun analyste n’est encore désigné.

Procédure : Stratégie 1 : IRM-Fuites-Données-PME

Purview → Gestion des risques internes → Stratégies → + → Politique de douane (c’est traduit comme ça dans l’interface en Français).

1. Modèle de stratégie — sélectionnez Fuites de données → sous-modèle Fuites de données (le neutre, pas « utilisateurs à risque » ni « utilisateurs prioritaires ») → **Suivant.**

2. Nom : IRM-Fuites-Données-PME | Description : Détection des fuites de données internes, groupes exposés à des données sensibles, nLPD art. 8 → **Suivant.**

3. Utilisateurs et groupes : sélectionnez « Utilisateurs, groupes et étendues adaptatives spécifiques », ajoutez les trois groupes : GRP-Direction-Purview, GRP-Finances-Purview, GRP-RH-Purview. **Ne jamais choisir « Tous les utilisateurs »**, principe de proportionnalité nLPD art. 6. Exclusions : laisser vide. → **Suivant.**

4. Contenu à prioriser : sélectionnez « Je souhaite hiérarchiser le contenu » cochez : Sites SharePoint, Étiquettes de confidentialité, Types d’informations sensibles : décochez Extensions de fichiers et Classifieurs pouvant être formés (trop larges pour un POC) → **Suivant.**

5. Sites SharePoint à hiérarchiser : ajoutez les trois sites sensibles : RH, Finances, Direction. Ne pas ajouter Transmission externe (couvert par DLP dédiée).

6. Étiquettes de confidentialité : sélectionnez : 3a - Direction - Site, 3b - Direction - Documents, 4 - RH-Confidentiel, 5 - Finances-Confidentiel. Ne pas ajouter 1 - Public ni 2 - Interne.

7. Types d’informations sensibles : ajoutez : AVS-Suisse-PME (confiance moyenne), Swiss Social Security Number AHV (confiance élevée), International Banking Account Number IBAN (confiance élevée). Cela couvre les fichiers non étiquetés contenant des données sensibles.

8. Score : sélectionnez « Obtenir des alertes pour toutes les activités » → **Suivant.**

9. Événement déclencheur : sélectionnez « L’utilisateur effectue une activité d’exfiltration » → cochez les activités de la liste (Téléchargement depuis SharePoint, partage externe, envoi email externe, copie USB), ne cochez pas « Certaines séquences nécessitent l’activation d’un déclencheur spécifique » (pour le POC) → Seuils de déclenchement : Appliquer des seuils intégrés (recommandé) → **Suivant.**

10. Indicateurs : cliquez « Activer les indicateurs » si le bandeau rouge apparaît, activez uniquement les Indicateurs Office (téléchargement SharePoint/OneDrive, partage externe, email externe) et les Voyants d’appareil de base. Ne pas activer Applications d’IA générative (Stratégie 2 séparée), Indicateurs de réseau (trop larges), Conformité des communications (section distincte).

11 Options de détection : Détections de séquence : ne rien cocher pour le POC. Détections d'exfiltration cumulées : cocher uniquement : Détecter les activités qui dépassent les normes organisationnelles. Ne pas cocher : Normes du groupe de pairs, cette option nécessite que les attributs organisationnels (département, manager) soient correctement renseignés dans Entra ID et un nombre suffisant d'utilisateurs par groupe pour être statistiquement fiable. Non pertinent en POC avec peu d'utilisateurs de test. → Suivant.

12. Seuils d’indicateur : sélectionnez « Appliquer les seuils fournis par Microsoft ». L’option « seuils spécifiques à l’activité de vos utilisateurs » peut être indisponible sur un tenant récent (pas assez d’historique), c’est normal → **Suivant → Envoyer.**

Stratégie 2 : IRM-Risque-IA (optionnel en préversion) 

⚠️ Fonctionnalité en préversion à évaluer avant tout déploiement en production. À créer obligatoirement dans une politique séparée ne jamais mélanger avec la Stratégie 1.

1. Modèle : Utilisation à risque de l’IA → **Suivant.**

2. Nom : **IRM-Risque-IA **→ Description : Détection des usages à risque de l’IA générative, Copilot et IA externes, nLPD art. 8 → **Suivant.**

3. Utilisateurs et groupes : mêmes 3 groupes que Stratégie 1 : GRP-Direction-Purview, GRP-Finances-Purview, GRP-RH-Purview → **Suivant.**

4. Contenu à prioriser : cochez uniquement : Sites SharePoint, Étiquettes de confidentialité, Types d’informations sensibles → décochez Extensions de fichiers et Classifieurs → **Suivant.**

5. Sites SharePoint : RH, Finances, Direction. Étiquettes : 3, 3b, 4, 5. SIT : AVS-Suisse-PME, Medical-RH-Suisse, IBAN, Swiss Social Security Number AHV → **Suivant.**

6. Score : « Obtenir des alertes pour toutes les activités » → **Suivant.**

7. Événement déclencheur : sélectionnez « Contenu à risque ou sensible dans les expériences Microsoft Copilot, les applications IA d’entreprise et les versions web d’autres applications IA » — cochez Sectionner tout dans la liste des activités → Seuils intégrés → Suivant.

8. Indicateurs — section « Applications d’IA générative » divisée en quatre sous-catégories avec modèles de facturation différents :

**Expériences Microsoft Copilot (2/2) : Gratuit ✅** Cochez les 2 indicateurs. Détecte les invites et réponses Copilot M365 et Copilot Studio. Inclus dans Business Premium + Purview Suite.

**Applications IA d’entreprise (0/2) : Facturation Azure requise ⚠️** Ne pas activer pour le POC.

**Autres applications IA (0/1) : Facturation Azure requise ⚠️ **Ne pas activer. Détecte les IA externes (ChatGPT, Gemini) via Defender for Cloud Apps, hors périmètre POC.

**Indicateurs Azure AI Content Safety (0/2) : Nécessite Communication Compliance ⚠️** Activables désormais : les stratégies Communication Compliance sont configurées (section 8.4). Cependant, ces indicateurs nécessitent une facturation Azure supplémentaire, non inclus dans Business Premium + Purview Suite. Ne pas activer sans validation budget.

9. Options de détection : séquences : laisser Sectionner tout coché. Amplificateurs : Sectionner tout → **Suivant.**

10. Seuils — Appliquer les seuils fournis par Microsoft → Suivant → **Envoyer.**

Stratégie 3 : IRM-Violations-Sécurité

Ce modèle détecte les comportements techniques anormaux, désactivation antivirus (RTP/BM), installation de logiciels malveillants, contournement des contrôles de sécurité, navigation vers des sites dangereux. Juridiquement moins sensible que la surveillance comportementale RH car le modèle cible des actes techniques objectifs, pas le travail quotidien. 
Prérequis : Microsoft Defender for Endpoint actif et alertes MDE partagées avec le portail de conformité.

Activation du partage des alertes MDE vers Purview : security.microsoft.com → Paramètres → Points de terminaison → Fonctionnalités avancées → localiser Microsoft Purview dans la liste → activer le toggle → Enregistrer les préférences. Une fois activé, les alertes MDE de gravité faible, moyenne et élevée sont disponibles comme événements déclencheurs dans cette stratégie. Si le toggle est déjà actif, les conditions préalables sont vertes dans l'interface IRM, aucune action supplémentaire n'est requise.

1. Modèle : Violations de stratégie de sécurité → **Suivant.**

2. Nom : IRM-Violations-Sécurité : Description : Détection des violations de sécurité techniques MDE,
nLPD art. 8 → **Suivant.**

3. Utilisateurs et groupes : mêmes 3 groupes : GRP-Direction-Purview, GRP-Finances-Purview, GRP-RH-Purview → Exclusions : vide → **Suivant.**

**4. Contenu à prioriser : spécificité de ce modèle : **Purview affiche le message « Il n’est pas utile de spécifier le contenu à définir pour la hiérarchisation pour le modèle que vous avez sélectionné ». La hiérarchisation est désactivée automatiquement. Ce modèle est basé sur les alertes MDE, pas sur le contenu des fichiers → **Suivant.**

**5. Événement déclencheur : automatique : **la stratégie commence à attribuer des scores uniquement lorsqu’une alerte MDE de gravité faible, moyenne ou élevée existe pour l’utilisateur. Les alertes d’information ne déclenchent pas. Aucune configuration manuelle → **Suivant.**

6. Indicateurs : activez uniquement :
**Indicateurs Microsoft Defender for Endpoint (2/2)** Fraude à la défense (contournement des contrôles de sécurité) + Logiciels indésirables (logiciels non approuvés ou malveillants). Ces deux indicateurs couvrent exactement la désactivation RTP/BM.

**Indicateurs de navigation à risque (4/4)** Piratage, hameçonnage, enregistreurs de frappe, sites malveillants. À activer via « Choisir des indicateurs » si grisés.

**Defender for Cloud Apps (0/11) ne pas activer :** nécessitent des connecteurs MDCA non configurés dans ce POC.

7. Seuils : Appliquer les seuils fournis par Microsoft → Suivant → Envoyer. Cochez les 3 notifications email avant **Terminé.**

**Validation terrain : **IRM-Violations-Sécurité créée et opérationnelle dans le tenant Axonix SA. Déclencheur automatique lié aux alertes MDE, la désactivation de RTP/BM testée en section 9.1 devrait générer une alerte sur cette stratégie.

Lien nLPD : articles concernés

Art. 6 (proportionnalité). Populations limitées aux groupes exposés, pas à tous les utilisateurs. Art. 8 al. 3 (mesures techniques), IRM comme mesure complémentaire DLP. Art. 12 (transparence). Information préalable des employés obligatoire. Art. 24 (notification PFPD), les alertes IRM constituent des preuves de surveillance en cas d’incident. Art. 26 OLT3 (protection personnalité au travail), anonymisation activée par défaut, levée encadrée (voir Annexe G).

📊 Note. Indicateur de couverture "faible" dans les 3 stratégies IRM : Purview affiche un indicateur de couverture faible ou moyen pour chaque stratégie car le périmètre est volontairement limité aux groupes GRP-Direction-Purview, GRP-RH-Purview et GRP-Finances-Purview. C'est le comportement attendu et conforme. La recommandation Microsoft d'étendre la couverture à tous les utilisateurs est une recommandation technique générique qui ne tient pas compte du cadre légal suisse. Étendre la surveillance à l'ensemble des 25 utilisateurs du tenant constituerait une violation du principe de proportionnalité (art. 6 nLPD) et de l'art. 26 OLT3. Ne pas modifier le périmètre des stratégies

Licence

L’IRM est inclus dans Microsoft 365 Business Premium + Purview Suite. Aucune licence E5 requise.
Lien nLPD : art. 8 (mesures techniques) + art. 24 (preuve en cas d’incident).

| **Template IRM** | **Signaux surveillés** |
| --- | --- |
| Fuites de données | Téléchargements massifs, copie USB, envoi email externe, upload cloud non autorisé |
| Employé sur le départ | Activité inhabituelle après annonce de départ, accès dossiers hors périmètre habituel |
| Risky Agents (DSPM IA) | Agents Copilot ou Foundry accédant à des données hors périmètre normal |
| Violation de sécurité | Tentatives de contournement DLP, désactivation antivirus, accès comptes sensibles |

Licence et activation

L’IRM est inclus dans Microsoft Purview Suite (disponible pour Business Premium). Pour l’activer : Purview → Gestion des risques internes → Stratégies → + Créer une stratégie → choisissez le template Fuites de données ou Employé sur le départ. Lien nLPD : art. 8 (mesures techniques) + art. 24 (preuve en cas d’incident).

💡 Synergie IRM + Microsoft Defender for Endpoint : les deux outils fonctionnent en duo. Dans le sens Defender → IRM : les alertes MDE (désactivation antivirus, logiciels malveillants) alimentent directement la stratégie IRM-Violations-Sécurité, c'est son événement déclencheur. Dans le sens IRM → Defender : les comportements à risque détectés par IRM (téléchargement massif, exfiltration) remontent dans le portail Defender (security.microsoft.com) sous forme d'incidents corrélés. Résultat : si un utilisateur désactive l'antivirus puis télécharge 50 fichiers RH, Purview corrèle les deux événements et génère un score de risque élevé, ce qu'aucun des deux outils ne ferait seul. C'est le principe de défense en profondeur documenté en section 9.0.

## 8.3 eDiscovery Premium Inclus Business Premium + Purview Suite

Concept

eDiscovery (découverte électronique) permet de rechercher, collecter, analyser et exporter des données pour répondre à une obligation légale — litige civil, enquête, demande d'accès aux données par une personne (droit d'accès nLPD art. 25), ou investigation interne. La version Premium ajoute l'IA pour dépliquer les résultats et analyser les conversations.

Cas d'usage nLPD concrets

| **Situation** | **Ce que eDiscovery permet** |
| --- | --- |
| Demande d'accès nLPD (art. 25) | Rechercher toutes les données personnelles d'un individu dans Exchange, SharePoint, Teams en quelques minutes |
| Litige RH (départ Favre) | Collecter tous les emails et fichiers liés à Favre, appliquer un hold légal — documents non supprimables pendant le litige |
| Notification PFPDT (art. 24) | Identifier précisément quelles données ont été exposées, qui y avait accès, produire le rapport pour le PFPDT dès que possible (art. 24 nLPD) |

Accès dans Purview : Purview > Solutions > eDiscovery > Cas Premium > + Créer un cas. Licence : inclus dans Microsoft Purview Suite (Business Premium, E3 ou E5). Aucun add-on séparé requis.

Prérequis

Avant de commencer, affectez votre compte au groupe de rôles **eDiscovery Manager** : Purview → Paramètres (roue dentée en bas à gauche) → Rôles et étendues → Groupes de rôles → **eDiscovery Manager** → **Modifier**.

Le groupe contient deux sous-rôles à configurer séparément :

**Page 1 : Gérer le gestionnaire de découverte électronique : **cliquez **Choisir des utilisateurs** → sélectionnez admin@demo.ch → Sélectionner. Le gestionnaire voit et modifie uniquement les cas auxquels il a accès. → Suivant.

**Page 2 : Gérer l’administrateur eDiscovery : **même opération → ajoutez admin@demo.ch. L’administrateur eDiscovery peut voir et modifier **tous** les cas du tenant, quelles que soient les autorisations, à réserver au compte admin principal. → Suivant → Enregistrer.

⚠️ **Propagation 15–30 minutes** — après l’enregistrement, attendez avant de continuer. Sans cette attente, le bouton **Exporter** ne sera pas visible dans le menu Actions du jeu à réviser. Licence : **✅ inclus Business Premium + Purview Suite** — aucun add-on requis.

Procédure terrain : Démonstration nLPD art. 25

Contexte POC : simulation d’une réponse à une demande d’accès nLPD (art. 25) pour **Valérie Beraz**, personnage fictif Axonix SA. Le compte **admin@demo.ch** est utilisé comme dépositaire de substitution (Valérie Beraz n’existe pas dans le tenant demo.onmicrosoft.com). ⚠️ En production : pour identifier le compte Entra ID d’un employé réel, allez sur entra.microsoft.com → Utilisateurs → recherchez le nom → copiez son UPN (ex. : valerie.beraz@demo.ch). C’est cet UPN que vous saisissez comme dépositaire à l’étape 3. Ne jamais utiliser admin@demo.ch comme dépositaire en production.

**Étape 1 Créer le cas eDiscovery Premium**

1. Purview → Solutions → eDiscovery → Cas → + Créer un cas.

2. Nom : **eDiscovery-Demande-Acces-nLPD**, Description : *Cas de démonstration, réponse à une demande d’accès nLPD art. 25, données personnelles de Valérie Beraz*. Activez le toggle eDiscovery Premium (active l’IA de déduplication et l’analyse avancée : ne pas oublier) → **Créer.**

**Étape 2 Ajouter les sources de données.**

3. Dans le cas → onglet Sources de données → + Ajouter → dans le panneau, sélectionnez toutes les sources dans ce cas → les 2 sources s’affichent (boîte aux lettres + site OneDrive) → cochez les deux → **Enregistrer et fermer**.

Résultat : bannière verte « *Ajout réussi de 2 sources de données* », Type de source : Contacts (boîte aux lettres) + Groupe (site).

**Étape 3 Créer et exécuter la recherche **

4. Onglet Recherches → Créer une recherche → dans la fenêtre qui s’ouvre : Nom : **Recherche-Beraz-art25** | Description : *Recherche données personnelles Valérie Beraz, demande accès art. 25 nLPD* → **Créer**.

5. L’écran de la recherche s’ouvre. Panneau gauche, Sources de données : cliquez **+ → **Tous les utilisateurs et groupes (section « Recherche à l’échelle du locataire ») cela couvre **toutes** les boîtes aux lettres et tous les sites SharePoint du tenant en une seule opération.

6. Panneau droit → Générateur de conditions : dans le champ Entrez des mots clés, saisissez **Beraz** → cliquez Exécuter la requête (bouton bleu en haut à droite) → dans la fenêtre « Choisissez les résultats de la recherche », laissez Statistiques sélectionné par défaut → **Exécuter la requête**.

7 Onglet Statistiques → résultat dans mon POC  275 correspondances (34,9 Mo). Le chiffre de 252 boîtes aux lettres peut surprendre sur un tenant de test avec peu d'utilisateurs : il inclut toutes les boîtes aux lettres système de Microsoft 365 (salles, équipements, groupes, boîtes partagées, ressources techniques) indexées lors d'une recherche sur l'ensemble du tenant. Les 23 sites SharePoint couvrent les sites de communication, d'équipe et les sites système OneDrive. Ces chiffres sont spécifiques au tenant Axonix SA et varieront selon le tenant cible. 2 erreurs « eDiscovery is not available » peuvent apparaître, elles concernent quelques emplacements système non indexables et n'affectent pas la validité des résultats.

**Étape 4 Ajouter au jeu à réviser**

8. Revenez à l’onglet Requête → bouton Ajouter au jeu à réviser (en haut à droite) → **+ **Nouvel ensemble de révision → Nom : **Révision-Beraz** → laissez toutes les options par défaut → **Ajouter**. La collecte s’exécute en arrière-plan.

9. Onglet Vérifier les ensembles → ouvrez Révision-Beraz → 359 éléments collectés (49,57 Mo) : emails Exchange, fichiers SharePoint/OneDrive, alertes DLP, et interactions Copilot M365. L’onglet **Tableau de bord (aperçu)** affiche la répartition par source, extension de fichier et étiquette de confidentialité.

⚠️ Point nLPD notable : eDiscovery remonte également les **interactions Copilot M365** (messages Copilot Word, BotChat). Si un collaborateur a interrogé Copilot sur Valérie Beraz, ces échanges sont inclus dans la réponse à la demande d’accès art. 25.

**Étape 5 Exporter les données : validation terrain**

10. Dans l’ensemble Révision-Beraz → barre du haut → menu **Actions** → **Exporter**. (Note : le bouton 
« Exporter » dans la barre de navigation redirige vers l’onglet Exportations du cas. C’est le menu Actions qui lance réellement l’export.)

11. Dans le panneau Exporter : Nom : **Export-Beraz-art25.** Description : *Export données personnelles Valérie Beraz, demande accès nLPD art. 25*. Laissez toutes les options par défaut (All documents in the review set + Export items with item report + format PST) → **Exporter**.

12. Onglet Exportations du cas → statut In progress → cliquez Actualiser jusqu’à statut Complete (environ 1–2 minutes) → cliquez sur la ligne de l’export → **Télécharger** → autorisez les pop-ups du navigateur si demandé.

13. Le package téléchargé contient 3 archives : PSTs.001.Export_Beraz_art25 (1emails Exchange au format PST) + Items.1.001.Export_Beraz_art25 (fichiers natifs Office et SharePoint) 
+ Reports-eDiscovery_Demande_Acces_nLPD (4 fichiers CSV : Items, Locations, Settings, Summary).

Livrable nLPD : ce que vous transmettez et à qui

Le package export est un **outil interne d’inventaire.** Vous ne le transmettez pas tel quel, Il sert à garantir que votre réponse est complète et vérifiable (aucun fichier, email ou interaction Copilot oublié). Le livrable final dépend du contexte :

**Demande d’accès nLPD art. 25 (à la personne concernée) : **rédigez une **lettre de réponse** résumant : quelles données sont détenues, où, depuis quand et dans quel but. Joignez les documents pertinents sous forme lisible (contrat, fiches, etc.). Le CSV Items vous sert à vérifier l’exhaustivité avant envoi.

**Notification PFPDT art. 24 (violation de données) : **remplissez le **formulaire officiel du PFPDT** sur pfpdt.admin.ch. Le package export vous fournit les données nécessaires pour renseigner ce formulaire (nombre de personnes concernées, types de données, période). Vous ne joignez pas le CSV brut.

⚠️ En cas de litige ou d’enquête formelle, utilisez l’onglet **Stratégies de conservation** du cas pour placer un **hold légal** sur les sources concernées : les documents et emails deviennent non supprimables pour toute la durée du litige, indépendamment des politiques de rétention. Bonne pratique : activer la conservation d’emblée dès tout signalement interne d’incident (art. 24 nLPD) pour sécuriser la chaîne de preuves.

Checklist eDiscovery Premium

✅ Rôle eDiscovery Manager affecté (Gestionnaire + Administrateur), propagation 15–30 min.

✅ Cas eDiscovery-Demande-Acces-nLPD créé avec toggle Premium activé

✅ 2 sources de données ajoutées (boîte aux lettres + site OneDrive admin@demo.ch)

✅ Recherche Recherche-Beraz-art25 exécutée : 275 correspondances estimées (tenant entier)

✅ 359 éléments collectés dans le jeu à réviser Révision-Beraz (49,57 Mo)

✅ Export Export-Beraz-art25 réalisé : 3 packages téléchargés (PST + fichiers natifs + rapports CSV)

Les chiffres dépendent évidement du nombre d’éléments détenus par l’utilisateur concerné.

## 8.4 Communication Compliance (Conformité des communications) Inclus Business Premium + Purview Suite

Concept

Communication Compliance surveille les communications (emails Exchange, messages Teams, Viva Engage) pour détecter des contenus problématiques, harcèlement, discrimination, contenu offensant, partage d'informations sensibles, violations de code de conduite. L'IA analyse le contenu et le soumet à des réviseurs humains désignés pour décision.

Ce que ça surveille

| **Politique de surveillance** | **Exemple de déclencheur** |
| --- | --- |
| Harcèlement / discrimination | Message Teams contenant des termes offensants ou menaçants |
| Fuite d'info confidentielle | Email mentionnant des données clients, chiffres financiers non publiés, mots-clés configurés |
| Conflits d'intérêts | Communication avec un concurrent nommé, mention de recrutement concurrent |

Accès dans Purview : Purview → Solutions → Conformité des communications → Stratégies → + Créer une stratégie.

Licence : inclus dans Microsoft Purview Suite pour Business Premium 

Prérequis :

**⚠️ Obligation légale nLPD avant activation : **la surveillance des communications est une donnée sensible au sens de l’art. 5 nLPD. Les collaborateurs doivent être informés de cette surveillance dans le règlement intérieur **avant activation** (art. 19 nLPD obligation d’information). Consultez un juriste avant déploiement en production. Voir Annexe G pour le modèle de clause.

Rôle requis : **Communication Compliance Admin** ajouter admin@demo.ch via Purview → Paramètres → Rôles et étendues → Groupes de rôles → Communication Compliance.

Stratégie 1 : CC-Surveillance-Données-Sensibles-nLPD

Objectif : détecter les communications contenant des données personnelles sensibles (AVS, données médicales, IBAN) circulant dans Exchange, Teams et Viva Engage — entrant, sortant et interne.

1. Purview → Conformité des communications → Stratégies → + Créer une stratégie → sélectionnez le modèle : Détecter les types d’informations sensibles.

2. Nom : **CC-Surveillance-Données-Sensibles-nLPD** → Utilisateurs concernés : Tous les utilisateurs, → Réviseurs : admin@demo.ch.

3. Informations sensibles à détecter → cliquez : Ajouter des informations sensibles ou un dictionnaire de mots clés → ajoutez les 4 SITs : AVS Suisse, Medical-RH-Suisse, International Banking Account Number (IBAN), Swiss Social Security Number AHV → **Créer une stratégie**.

Paramètres automatiques : Exchange + Teams + Viva Engage. Direction : Entrant, Sortant, Interne, Pourcentage : 100 %. Résultat : statut Activating / Intègre. La stratégie devient active dans les 24 heures.

Stratégie 2 : CC-Contenu-Inapproprié-nLPD

Objectif : détecter les contenus inappropriés (harcèlement, discrimination, violence, contenu sexuel) dans les messages Teams et Viva Engage internes, obligation de prévention de l’employeur (CO art. 328).

1. Purview → Conformité des communications → Stratégies → + Créer une stratégie → sélectionnez le modèle : Signaler un contenu inapproprié.

2. Nom : **CC-Contenu-Inapproprie-nLPD** | Utilisateurs concernés → Tous les utilisateurs → Réviseurs : admin@demo.ch → **Créer une stratégie**.

Paramètres automatiques : Teams + Viva Engage.  Direction : Interne, Classifieurs : Sexual, Violence, Hate, Self-harm. Pourcentage : 100 %. Résultat : statut Activating / Intègre.

Vue d’ensemble au moment de la configuration

Avant même toute configuration, la vue d’ensemble de Communication Compliance affichait déjà des alertes sur le tenant demo.onmicrosoft.com : 1 message Teams avec contenu potentiellement inapproprié + 214 emails et 32 messages Teams contenant des informations sensibles (dont Medical-RH-Suisse : 19 occurrences, Swiss Social Security Number AHV : 3 occurrences). Argument POC : *« avant même de configurer quoi que ce soit, l’outil voit déjà ce qui circule dans votre tenant »*.

Checklist Communication Compliance

✅ Information préalable des collaborateurs documentée (règlement intérieur, art. 19 nLPD)

✅ Stratégie CC-Surveillance-Donnees-Sensibles-nLPD créée. 4 SIT configurés

✅ Stratégie CC-Contenu-Inapproprié-nLPD créée. classifieurs Sexual/Violence/Hate/Self-harm

*⚠️ Note nLPD — La surveillance des communications est une donnée sensible au sens de l’art. 5 nLPD. Les collaborateurs doivent être informés de cette surveillance dans le règlement intérieur avant activation (art. 19 nLPD, obligation d’information). Consultez un juriste avant déploiement en production.*

## 8.5 Gestionnaire de conformité

Le Gestionnaire de conformité est un tableau de bord transversal de Purview qui mesure la progression globale de votre organisation vers les bonnes pratiques de sécurité et de conformité réglementaire. Il ne remplace pas les configurations effectuées dans ce guide : il les mesure et les contextualise par rapport aux réglementations actives dans votre tenant.

Navigation : Purview → Gestionnaire de conformité → Vue d’ensemble.

Score global et répartition

Le score global affiché dans le tenant Axonix SA est de 63 % (14 136,67 / 22 416 points). Il se compose de deux parties distinctes : vos points gagnés (1 728,67 / 9 915) correspondent aux actions que vous avez configurées vous-même ; les points gérés par Microsoft (12 408 / 12 501) correspondent aux protections que Microsoft applique nativement dans sa plateforme (chiffrement en transit, redondance des datacenters, contrôles d’accès infrastructure). Ces deux composantes sont indépendantes : vous ne pouvez pas agir sur les points Microsoft, uniquement sur vos propres points.

📊 **Score et mise en page du tenant : **le score de 63 % est spécifique au tenant Axonix SA au moment du POC. Il progressera automatiquement à mesure que les configurations de ce guide sont déployées, sans aucune action manuelle dans le Gestionnaire de conformité.

Évaluations actives

Le Gestionnaire de conformité contient 3 évaluations actives dans le tenant Axonix SA. Navigation : Purview → Gestionnaire de conformité → Évaluations.

| **Évaluation** | **Progression** | **Réglementation** |
| --- | --- | --- |
| Data Protection Baseline for Microsoft 365 | 64 % — 58 / 489 vos actions terminées, 729 / 734 actions Microsoft terminées | Ligne de base de protection des données |
| LPD SUISSE | 33 % — 3 / 34 vos actions terminées, 12 / 12 actions Microsoft terminées | Switzerland FADP |
| AI Baseline Assessment | 64 % — 1 / 80 vos actions terminées, 96 / 96 actions Microsoft terminées | AI Baseline |

L’évaluation LPD SUISSE (Switzerland FADP) est la plus pertinente pour une PME suisse. Elle est incluse dans la souscription Purview Suite (1 licence réglementaire utilisée sur 3 disponibles). Son score de 33 % reflète un état de début de déploiement : les 12 actions Microsoft sont toutes terminées, mais 31 de vos 34 actions restent à compléter. Plusieurs de ces actions sont déjà couvertes par ce guide (DLP, étiquettes de sensibilité, rétention) mais le Gestionnaire de conformité peut ne pas les détecter automatiquement si elles sont en mode simulation ou non encore validées.

Lire et utiliser une action d’amélioration

Chaque action d’amélioration contient trois onglets : Détails (procédure d’implémentation et lien vers la solution), Preuve (fichiers ou liens que vous chargez pour documenter votre implémentation), Commandes associées (liste des articles réglementaires que cette action adresse dans chaque évaluation active).

Exemple : l’action Search audit logs dans l’évaluation LPD SUISSE est liée à l’art. 21 FADP (Offering documents to the Federal Archives) et à trois contrôles AI Baseline (Transparency, Logging and Traceability, AI System Logging). Pour la marquer comme implémentée, chargez une preuve (export du journal d’audit, capture d’écran) dans l’onglet Preuve, puis passez l’état d’implémentation à Implémenté.

ℹ️ **Bouton Launch Now : **certaines actions d’amélioration contiennent un bouton Launch Now dans l’onglet Détails. Il redirige vers l’outil Microsoft approprié pour implémenter l’action : portail Defender (security.microsoft.com) pour les actions liées à la sécurité des appareils, portail Intune pour la gestion des appareils mobiles, etc. Ces redirections sont normales et attendues. Elles ne nécessitent pas d’action supplémentaire dans Purview.

Actions LPD SUISSE pertinentes pour ce POC

Parmi les 34 actions de l’évaluation LPD SUISSE, plusieurs sont directement couvertes par les configurations de ce guide. Les actions automatiques (testées automatiquement par Purview) progresseront d’elles-mêmes ; les actions manuelles nécessitent de charger une preuve.

| **Action** | **État actuel** | **Couverte par ce guide** |
| --- | --- | --- |
| Create and deploy data loss prevention policies | 0 / 27 — Aucun | Oui — sections 4.1 à 4.5 (manuel : charger preuve) |
| Create customized or use default DLP policies for endpoints | 0 / 27 — Aucun | Oui — section 9.1 (manuel : charger preuve) |
| Automatically apply retention labels | 0 / 27 — Risque élevé d’échec | Oui — section 8.1 (automatique) |
| Search audit logs | 0 / 9 — Aucun | Oui — section 0.3 Audit (manuel : charger preuve) |
| Digitally sign secure channel data | 27 / 27 — Réussite | Déjà validée automatiquement |
| Encrypt or sign secure channel data | 27 / 27 — Réussite | Déjà validée automatiquement |
| Display privacy statements for end users | 0 / 27 — Aucun | Non : nécessite Intune MDM |
| Remove TLS 1.0/1.1 and 3DES dependencies | 0 / 27 — Aucun | Non : hors périmètre Business Premium |

Périmètre et recommandations pour une PME suisse

La majorité des 537 actions d’amélioration visibles dans le Gestionnaire de conformité concernent Microsoft Defender for Endpoint complet, Intune MDM, Microsoft Entra ID Governance ou des licences E3/E5. Ces actions sont hors périmètre d’une configuration Business Premium + Purview Suite. Ne pas chercher à tout implémenter : le score n’est pas une course. L’objectif est de couvrir les actions pertinentes pour la nLPD, pas d’atteindre 100 %.

⚠️ **Jauge verte ≠ objectif : **un score de 63 % sur un tenant Business Premium bien configuré est un bon résultat. Les points manquants correspondent majoritairement à des fonctionnalités nécessitant des licences supplémentaires. Se concentrer sur l’évaluation LPD SUISSE et viser la complétion des actions couvertes par ce guide.

Suivi et routine de maintenance

Le Gestionnaire de conformité se met à jour automatiquement au fur et à mesure des déploiements. Une revue trimestrielle est suffisante pour une PME : consulter l’évaluation LPD SUISSE, vérifier les nouvelles actions apparues, charger les preuves des configurations réalisées. Navigation : Gestionnaire de conformité → Évaluations → LPD SUISSE → Vos actions d’amélioration → filtrer par État : À terminer.

---

[← Partie 7 — Gestion opérationnelle et démos](08-gestion-operationnelle.md) | [Partie 9 — Endpoint DLP, Edge for Business et AI Hub →](10-endpoint-dlp-aihub.md)
