---
title: "Partie 3 — Auto-labelling (étiquetage automatique) | Configuration Microsoft Purview 2026"
description: "Configurer l'étiquetage automatique des données sensibles (AVS, IBAN, données médicales) dans Microsoft Purview : politique d'auto-labelling, mode simulation et vérification des résultats."
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

# Partie 3 — Auto-labelling (étiquetage automatique)

## 3.1 Comprendre l'auto-labelling

L’auto-labelling analyse les contenus de manière asynchrone en arrière-plan dans SharePoint, OneDrive et Exchange. Cette analyse dépend des capacités du service et peut prendre un certain temps, elle n’est ni immédiate ni garantie à un instant donné. Si Purview détecte un numéro AVS correspondant aux conditions configurées (type d’information, niveau de confiance, seuil), il peut appliquer automatiquement le label RH-Confidentiel. Après évaluation par le service, la couverture doit être vérifiée via le mode simulation avant activation.

| **Type** | **Comment ça marche** | **Avec Purview Suite** |
| --- | --- | --- |
| Auto-labelling client | Appliqué quand l'utilisateur ouvre le doc dans Word/Excel, suggestion visible dans la barre d'état | ✅ Configuré en section 2.4, étape 8 |
| Auto-labelling service (côté serveur) | Purview analyse automatiquement les fichiers SharePoint et OneDrive de manière asynchrone, selon les capacités du service et les emplacements configurés. Cette analyse n’est ni instantanée ni garantie exhaustive à un instant donné. | ✅ C'est l'objet de cette partie |

## 3.2 Créer la politique d'auto-labelling

Vous allez créer deux stratégies distinctes, une par étiquette cible car l’interface Purview associe un label unique à chaque stratégie.
Accès : Purview → Protection des données → Stratégies → Stratégies d’étiquetage automatique → + Créer une stratégie d’étiquetage automatique.

**⚠️ Séquence recommandée : **les SITs personnalisés **AVS-Suisse-PME** et **Medical-RH-Suisse** créés en section 2.3bis sont utilisés directement ci-dessous. Attendez la propagation (15–30 min) avant de continuer.

Stratégie 1 : Détection AVS et données médicales → 4 - RH-Confidentiel

**Étape 1 : Choisissez les informations à protéger**

Faites défiler la liste des catégories → sélectionnez Personnaliser → Suivant.

**Étape 2 : Nommer votre stratégie**

Nom : **Auto-Label-RH-Sensible** | Description : Détecte automatiquement les numéros AVS et données médicales dans SharePoint, OneDrive et Exchange. → **Suivant.**

**Étape 3 : Choisir une étiquette à appliquer automatiquement**

Cliquez sur choisir une étiquette → sélectionnez 4 : RH-Confidentiel → Appliquer → **Suivant.** ⚠️ Étape critique : l’étiquette choisie ici s’appliquera à tout document détecté par les règles de cette stratégie.

**Étape 4 : Unités d’administration**

Laissez sur Répertoire complet → **Suivant.**

**Étape 5 : Choisir les emplacements**

Activez les trois emplacements : Courrier Exchange (Tous utilisateurs et groupes), Sites SharePoint : Modifier → Exclure → saisir https://demo.sharepoint.com/sites/transmissionexterne → + → Terminé (ce site est géré par DLP dédiée, l’étiquette 4 - RH-Confidentiel ne doit pas s’y appliquer automatiquement), Comptes OneDrive (Tous utilisateurs et groupes). Azure Storage : laisser désactivé. Cliquez sur **Suivant.**

**Étape 6 : Configurer des règles**

Sélectionnez Règles communes → **Suivant.**

**Étape 7 : Définir des règles (+ Nouvelle règle)**

***Règle 1 : AVS suisse :*** Nom : Règle-AVS-Suisse | Le contenu contient → Nom du groupe : Règle-AVS-Suisse → +Ajouter → Types d’infos confidentielles → tapez AVS-Suisse-PME → sélectionnez AVS-Suisse-PME | Confiance moyenne | Nombre d’instances min : 1 → **Enregistrer.**

***Règle 2 : Médical :*** Nom : **Règle-Medical** → Le contenu contient → Nom du groupe : Règle-Medical | Opérateur du groupe : N’importe lequel de ces éléments (OU).

**Condition unique : **+ Ajouter → Types d’infos confidentielles → tapez et sélectionnez : **Medical-RH-Suisse** | Confiance : Moyenne | min : 2. → **Enregistrer**.

→ Logique globale : la Règle 1 détecte les documents contenant un numéro AVS (min 1). La Règle 2 détecte les documents contenant des termes médicaux RH (min 2), même sans numéro AVS. Les deux règles fonctionnent indépendamment : un document déclenchant l’une ou l’autre reçoit l’étiquette 4 - RH-Confidentiel.

Une fois les 2 règles créées, cliquez sur Suivant.

**Étape ****8  :**** Paramètres d’étiquette supplémentaire**

Cochez uniquement : Tous les emplacements (préversion) : remplace les étiquettes de priorité inférieure sur tous les emplacements. Laissez décoché : Emails uniquement et Appliquer le chiffrement aux e-mails reçus en dehors de votre organisation. Cliquez sur Suivant.

**Étape 9 : Mode de la stratégie**

Sélectionnez Exécuter la stratégie en mode de simulation. Laissez décoché : Activez automatiquement la stratégie si elle n’est pas modifiée après 7 jours. Cliquez sur Suivant.

**Étape 10 : Finaliser**

Vérifiez le résumé et cliquez sur Créer une stratégie.

Stratégie 2 : Détection IBAN → 5 - Finances-Confidentiel

Répétez exactement les étapes 1 à 10 avec les adaptations suivantes :

**Étape 2 : **Nom : Auto-Label-Finances-IBAN | Description : Détecte automatiquement les IBAN dans SharePoint, OneDrive et Exchange.

**Étape 3 : **Étiquette : 5 - Finances-Confidentiel.

**Étape 7 : **Une seule règle : Nom : Règle-IBAN | Nom du groupe : Règle-IBAN | Type sensible : IBAN (International Banking Account Number) | Confiance élevée | min 1 | Enregistrer.

> 🚨 **IMPORTANT : **Démarrez toujours en mode simulation. Vérifiez les résultats dans l’onglet Éléments correspondants de chaque stratégie avant d’activer. Les délais d’analyse varient de quelques heures à plusieurs jours selon la taille du tenant.

## 3.3 Vérifier les résultats de simulation

Une fois la simulation terminée (délai variable selon le tenant), voici comment vérifier :

1. Purview → Protection des données → Stratégies → Stratégie d’étiquetage automatique → cliquez sur votre stratégie Auto-Label-RH-Sensible ou Auto-Label-Finances-IBAN.

2. Vous voyez le statut : En cours de simulation ou Simulation terminée.

3. Cliquez sur Afficher les résultats de simulation, vous voyez la liste des fichiers qui auraient reçu un label.

4. Vérifiez que vos documents RH (11 à 16) et Finances (17, 18) apparaissent dans la liste.

5. Si un fichier attendu manque → vérifiez qu'il est bien uploadé dans SharePoint et que le site est inclus dans les emplacements de la stratégie.

6. Si des fichiers inattendus apparaissent → augmentez le niveau de confiance (de Moyen à Haute) dans les règles.

7. Une fois satisfait → cliquez sur Activer la stratégie.

> ✅ **CONSEIL : **Pour votre lab : les fichiers 11_Contrat_Travail, 13_Fiches_Salaire, 14_Registre_Personnel, 17_Grand_Livre doivent obligatoirement apparaître dans les résultats. S'ils manquent, ils ne sont probablement pas encore uploadés dans SharePoint.

---

[← Partie 2 — Les Étiquettes de sensibilité (Labels)](03-etiquettes-sensibilite.html) | [Partie 4 — Politiques DLP (Data Loss Prevention) →](05-politiques-dlp.html)
