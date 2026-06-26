---
title: "Partie 5 — Audit et DSPM for AI (surveillance Copilot) | Configuration Microsoft Purview 2026"
description: "Configurer DSPM for AI et l'audit Purview pour surveiller les interactions Copilot M365 : Activity Explorer, recherche d'accès spécifiques dans les journaux d'audit."
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

# Partie 5 — Audit et DSPM for AI (surveillance Copilot)

## 5.1 Configurer DSPM for AI

Microsoft a unifié en 2026 DSPM for AI (classique)** **et DSPM dans une interface unifiée appelée DSPM (aperçu)**.** Les deux expériences restent actuellement accessibles depuis la barre latérale gauche du portail Microsoft Purview. À ce jour, Microsoft n’a communiqué aucune date officielle de suppression ou de retrait de DSPM for AI (classique). Les expériences classiques (DSPM classique et DSPM for AI classique) demeurent disponibles en parallèle, mais n’évoluent plus fonctionnellement. Les investissements et nouvelles fonctionnalités sont désormais concentrés sur la nouvelle interface DSPM**.**

Ce guide documente exclusivement la nouvelle interface DSPM unifiée.

Périmètre de DSPM en Business Premium + Purview Suite

DSPM (aperçu) est avant tout un **tableau de bord de reporting et de surveillance** dans le cadre de la licence Business Premium + Purview Suite. La majorité des fonctionnalités d’action visibles dans l’interface nécessitent des licences supplémentaires non incluses dans ce périmètre. Connaître ce périmètre évite de se perdre dans une interface conçue pour montrer ce qui manque.

**Inclus et opérationnel dans ce guide : **Rapports (Exfiltration, DLP, Étiquettes, IA), Observabilité IA, Activity Explorer Copilot, Métriques de posture, Data Risk Assessments au niveau site.

**Nécessite des licences supplémentaires : **Politiques one-click (extension navigateur + Endpoint DLP requis), Analyse item-level des Data Risk Assessments (Fabric / Azure AKS), Capture des interactions Copilot (M365 Copilot par utilisateur). Ne pas chercher à activer ces fonctionnalités ici, elles sont documentées dans l’Annexe E.

Accès et navigation :

Purview → barre latérale gauche → **DSPM (aperçu)**. Structure du menu : Posture, Objectifs, Observabilité IA, Découvrir (Applications et agents, Explorateur d’activités, Explorateur de ressources, Évaluations des risques), Tâches et actions (Tâches de configuration, Action de correction), Rapports.

Posture : Métriques clés à surveiller

Trois indicateurs sur les 30 derniers jours : (1) **Ressources classées et étiquetées**, objectif : 100% via auto-labelling et campagnes manuelles (section 2.7). (2) **Activités couvertes par DLP**, objectif : 100%. 
(3) **Alertes de protection des données**, objectif : 0 alertes non traitées (voir Annexe F SLA 48h).

Objectifs : Protections Copilot en place

Objectifs de sécurité des données, état après configuration POC
La page Objectifs affiche 4 axes de surveillance. À date du POC Axonix SA, l'état est le suivant :
⚠️ Empêchez l'exposition de données dans les interactions Copilot : 5 interactions avec données sensibles détectées. Normal en phase de test : les fichiers du dataset POC (AVS, IBAN) ont été référencés par Copilot. Action : vérifier dans Activity Explorer quels fichiers sont concernés et confirmer que les étiquettes RH-Confidentiel et Finances-Confidentiel sont bien appliquées.
⚠️ Empêchez l'exfiltration vers des destinations à risque : 1 fichier à risque détecté. Vérifier dans le plan de correction quel fichier est concerné et s'il est correctement étiqueté. Si oui, le signal disparaîtra dans les 24h suivant l'activation de la DLP-Protection-Endpoint.
⚠️ Découvrez les sources de données non surveillées référencées par les applications IA : 1 interaction IA référençant une source non couverte par DSPM. Normal si des fichiers ont été consultés via Copilot avant l'activation des politiques.
⚠️ Découvrez les données sensibles au sein de votre entreprise : 0 fichiers à risque. C'est le seul objectif sans risque actif, les étiquettes couvrent bien le périmètre.

Observabilité IA, Applications et agents surveillés

Applications et agents surveillés : 3 applications détectées (30 derniers jours) : M365Copilot - Apps, M365Copilot - Bizchat, Word Drafting Agent, tous statut Actif, risque niveau Aucun. Trois métriques surveillées : assistants à risque élevé/moyen/faible (0/0/0) et agents avec interactions sensibles.
Partage excessif, exfiltration, contraire à l'éthique (tous à 0).

Tâches de configuration : État pour Business Premium + Purview Suite

Purview → DSPM (aperçu) → Tâches et actions → Tâches de configuration. Après configuration complète de ce guide : 2 terminées, 8 non démarrées.

**Terminées (incluses dans ce guide)**** : **Activer Microsoft Purview Audit (section 1.5) ; Protéger vos données avec étiquettes de sensibilité (Partie 2).

**Nécessitent Azure Pay-as-you-go (hors scope Business Premium + Purview Suite) : **Configurer la facturation à l’utilisation. Interactions sécurisées dans les expériences Microsoft Copilot. Étendre les insights données sensibles apps IA (détection réseau via SASE/SSE). Sécuriser les interactions apps IA d’entreprise.

**Optionnel : nécessite extension navigateur Purview : **Étendre vos insights de découverte de données. Configure 3 stratégies Edge for Business (détection invites IA, visites sites IA, contenus collés sur sites IA), mode audit uniquement, zéro impact utilisateur.

**Disponibles mais non prioritaires PME : **Intégrer des appareils à Microsoft Purview (section 7).
Intégrez les solutions partenaires (Varonis, Cyera, BigID, hors scope PME).

Extension navigateur Microsoft Purview : Valeur ajoutée et prérequis

Sans l’extension navigateur, la DLP Purview couvre Exchange, SharePoint, OneDrive, Teams et Copilot mais pas le navigateur web. Un utilisateur peut contourner l’ensemble du dispositif en ouvrant Gmail ou WeTransfer dans Chrome et en attachant directement un fichier RH depuis son poste. L’extension comble cet angle mort : elle surveille et contrôle ce qui se passe dans le navigateur indépendamment du canal utilisé.

**Ce que l’extension détecte et contrôle : **upload d’un fichier sensible vers un site externe (WeTransfer, Dropbox perso, Gmail web), copier-coller de contenu sensible dans un formulaire web ou une IA grand public (ChatGPT, Gemini), impression vers PDF depuis le navigateur, visite de sites IA non autorisés avec des données sensibles en mémoire.

**⚠️ Prérequis obligatoires avant installation : **(1) Endpoint DLP activé sur le poste (On le fera en section 9.1), sans Endpoint DLP, l’extension ne fait rien. (2) Déploiement via Intune ou GPO, l’installation manuelle poste par poste n’est pas viable en entreprise. (3) Phase audit obligatoire, activer en mode audit 2 à 4 semaines avant tout blocage pour identifier les usages légitimes à exclure (ex. : envoi de fichiers publics vers des partenaires via WeTransfer).

**Positionnement dans la roadmap PME : **étape de maturité avancée, à déployer après consolidation de la couche de base (classification + DLP Exchange/SharePoint + protection Copilot). Comptez 2 à 4 semaines de projet dédié incluant la configuration Intune, le déploiement de l’extension et la phase d’audit.

Action de correction : 12 recommandations

Purview → DSPM (aperçu) → Tâches et actions → Action de correction : 13 éléments, 12 non démarrés, 1 rejeté. La recommandation "Protéger les données sensibles dans les interactions avec Microsoft 365 Copilot" a été rejetée (Dismissed) car couverte par la DLP-Protection-Copilot configurée en section 4.4. Les recommandations "Empêchez l'exfiltration vers des destinations à risque", "Découvrez les données sensibles" et "Prévenez le partage excessif" seront adressées par Endpoint DLP (section 9.1). Ne traitez aucune action de correction à cette étape.

Politiques one-click DSPM : Information

DSPM propose des actions de correction accessibles depuis **DSPM (aperçu) → Objectifs → Afficher le plan de correction** sur chaque objectif. Ces actions varient selon l’objectif sélectionné et l’état de votre tenant, elles ne correspondent pas à une liste fixe de politiques nommées. Pour chaque objectif vous trouverez des blocs d’action : créer une stratégie DLP, appliquer une stratégie existante, configurer l’IRM, lancer un Data Risk Assessment, ou améliorer la conformité IA. Chaque bloc indique clairement si la stratégie est déjà en place, en cours, ou non démarrée. **Ne pas activer ces actions à cette étape.** La plupart nécessitent l’extension navigateur et/ou Endpoint DLP (section 9). Les actions liées à Copilot sont déjà couvertes par votre politique DLP-Protection-Copilot configurée en section 4.4. Attention à ne pas créer de doublon.

Data Risk Assessments : Cartographier les données surexposées

Les Data Risk Assessments identifient les fichiers SharePoint, OneDrive et Teams accessibles à un périmètre plus large que nécessaire, liens de partage ouverts, fichiers sensibles surexposés.
Navigation : **DSPM (aperçu) → Découvrir → Évaluations des risques liées aux données**. L’interface propose un workflow en 3 étapes : (1) Identifier, examiner les résultats pour les utilisateurs accédant à des éléments sensibles. (2) Protéger, limiter l’accès Copilot aux données sensibles et appliquer des stratégies d’étiquette. (3) Surveiller, effectuer des examens des accès SharePoint. Un bandeau rouge « Analyse au niveau de l’élément non configurée : Configurer la connexion » peut apparaître : **ne pas cliquer sur ce bouton !** Il concerne Microsoft Fabric / Azure AKS, hors périmètre Business Premium. Le scan niveau site est suffisant pour une PME et les résultats permettent de désactiver en masse les liens « Anyone with the link » et d’identifier les fichiers sensibles mal protégés.

Routine de surveillance mensuelle

Revue mensuelle suffisante en régime de croisière, revue hebdomadaire recommandée pendant les 3 premiers mois. Checklist : (1) Consulter les nouvelles recommandations DSPM, traiter celles classées « Élevée ». (2) Vérifier l’évolution du taux de couverture des sensitivity labels (objectif > 80 %). (3) Analyser les pics d’activité DLP (augmentation anormale de correspondances). (4) Relancer un Data Risk Assessment (trimestriel) : DSPM → Découvrir → Évaluations des risques liées aux données → + Créer une évaluation personnalisée. Le scan global se relance automatiquement, mais le trimestriel ciblé passe par ce bouton. (5) Documenter les actions correctives dans le registre nLPD. Le rapport DSPM (aperçu) → Rapports → Microsoft 365 Copilot se peuple automatiquement au fil de l’utilisation et constitue l’outil de surveillance principal des interactions Copilot avec les données sensibles, aucune configuration supplémentaire requise. Les autres rapports disponibles (Exfiltration, Utilisation de l’IA et risque, Étiquettes) suivent la même logique et sont à explorer selon les besoins.

## 5.2 Utiliser Activity Explorer

Activity Explorer est votre tableau de bord principal pour vérifier que tout fonctionne.

1. Purview → DSPM (aperçu) → Découvrir → Explorateur d’activités → onglet Activités d’IA.

2. Plage de dates : Derniers 7 jours.

3. Filtres utiles :

| **Ce que vous cherchez** | **Filtre à appliquer dans Activity Explorer** |
| --- | --- |
| Vérifier que les labels s'appliquent | Type d'activité = Étiquette appliquée |
| Labels appliqués automatiquement | Type d'activité = Étiquette appliquée automatiquement |
| Déclenchements DLP | Type d'activité = Correspondance de règle DLP |
| Accès Copilot | Type d'activité = Interaction Copilot (si visible) |
| Tentatives bloquées | Type d'activité = Remplacement de la stratégie DLP par l'utilisateur |
| Par utilisateur spécifique | Filtre Utilisateur = tapez le nom de votre utilisateur de test |

## 5.3 Rechercher un accès spécifique dans les logs

Pour rechercher qui a accédé à un fichier précis (ex : le contrat de Valérie Beraz) :

1. Purview → Audit → Recherche.

2. Plage de dates : choisissez la période souhaitée.

3. Activités : cliquez sur la liste déroulante→ Fichiers et pages → cochez Fichier consulté.

4. ObjectId (fichier, dossier ou site): tapez le nom du fichier (ex : Beraz).

5. Cliquez sur Rechercher.

6. Les résultats montrent : qui a accédé, quand, depuis quelle IP, depuis quel outil (Copilot, navigateur, etc.).

7. Cliquez sur un résultat pour voir le détail complet de l'événement.

**Limitation Purview, fichiers sans étiquette : **Microsoft Purview ne propose pas de filtre natif « étiquette de sensibilité = aucune » dans l’Explorateur de données (Content Explorer). La vue est toujours positive , elle affiche ce qui est étiqueté, pas ce qui ne l’est pas. C’est un comportement attendu du produit, non un défaut de configuration ou de licence. La démarche pour identifier et piloter cette dette de labellisation est documentée en section 7.5.

---

[← Partie 4 — Politiques DLP (Data Loss Prevention)](05-politiques-dlp.md) | [Partie 6 — Scénarios de test et validation →](07-tests-validation.md)
