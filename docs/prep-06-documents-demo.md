---
title: "Préparation — Documents de démonstration | Configuration Microsoft Purview 2026"
description: "Répartition et dépôt des fichiers de démonstration (contrats, fiches de salaire, registre du personnel, IBAN) dans les sites SharePoint pour tester l'auto-labelling Purview."
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

# Partie 6 — Déposer les documents de démonstration dans SharePoint

Le guide Purview utilise des documents contenant des données sensibles réelles pour tester la détection automatique (auto-labelling). Les fichiers du dossier FICHIERS_DEMO_AXONIX.zip sont prêts à l'emploi.
Ils contiennent des numéros AVS, des IBAN et des informations confidentielles fictives d'une société fictive (Axonix SA).

> **ℹ️  CE QUE PURVIEW VA DÉTECTER DANS CES FICHIERS** Règle AVS → étiquette 4 - RH-Confidentiel : déclenche sur les fichiers 11, 12, 13, 14, 15, 16 (numéros AVS 756.xxxx). Règle IBAN → étiquette 5 - Finances-Confidentiel : déclenche sur les fichiers 14, 17, 18 (IBAN CH xx xxxx). Le fichier 14 (Registre Personnel) déclenche LES DEUX règles, il est particulièrement intéressant pour la démo.

## 6.1 Répartition des fichiers par site SharePoint

Déposez chaque fichier dans le site correspondant à son niveau de confidentialité. Cette répartition est volontairement représentative d'une vraie PME suisse.

Site Communication : accès tous les employés

Fichiers sans données personnelles sensibles, à portée générale.

| **Fichier** | **Contenu** | **Données sensibles** |
| --- | --- | --- |
| 01_Presentation_Axonix_SA.docx | Présentation société, IDE, adresse, capital social | Aucune (public) |
| 03_Reglement_Interieur_2025.docx | Règlement intérieur, horaires, congés, code de conduite | Aucune (tous employés) |
| 07_Guide_Depannage_Helpdesk.docx | Guide support N1/N2, procédures techniques helpdesk | Aucune (usage interne) |

Site Direction : accès GRP-Direction uniquement

Fichiers stratégiques et confidentiels réservés à la direction.

| **Fichier** | **Contenu** | **Données sensibles** |
| --- | --- | --- |
| 04_Reponse_AO_Migration_Azure_Baumont.docx | Réponse appel d'offres Baumont Industries, tarifs, architecture proposée | Confidentiel commercial |
| 05_Politique_Securite_Gestion_Incidents.docx | Politique sécurité, classification incidents P1/P2, procédures | Confidentiel interne |
| 08_CR_Comite_Direction_18mars2025.docx | Compte rendu comité de direction, décisions stratégiques, budgets | Confidentiel direction |

Site RH : accès GRP-RH + GRP-Direction (lecture)

Fichiers RH contenant des données personnelles sensibles au sens de la nLPD. Ce sont ces fichiers qui déclenchent la règle AVS dans l'auto-labelling Purview.

| **Fichier** | **Contenu** | **Données sensibles détectées par Purview** |
| --- | --- | --- |
| 11_Contrat_Travail_Beraz_Valerie_RH.docx | Contrat de travail, Valérie Beraz, Resp. RH | 🔴 AVS : 756.4521.8834.09 |
| 12_Contrat_Travail_Tanner_Kevin_Finances.docx | Contrat de travail, Kevin Tanner, Resp. Finances | 🔴 AVS : 756.8821.4467.23 |
| 13_Fiches_Salaire_Avril_2025.docx | Fiches de salaire avril 2025, montants nets et bruts | 🔴 AVS : 756.4521.8834.09 |
| 14_Registre_Personnel_2025.docx | Registre complet du personnel, 7 employés, salaires | 🔴🔴 AVS (×7) + IBAN (×7) - FICHIER CLÉ DÉMO |
| 15_Evaluation_Annuelle_Nguyen_Linh_2024.docx | Évaluation annuelle, Linh Nguyen, Dev Senior | 🔴 AVS : 756.9988.7766.12 |
| 16_Note_Disciplinaire_Favre_Laurent.docx | Note disciplinaire, L. Favre, fuite tarifaire client | 🔴 AVS : 756.5544.3322.87 + Strictement confidentiel |

> **⚠️  LE FICHIER 14 EST LE PLUS IMPORTANT POUR LA DÉMO** 14_Registre_Personnel_2025.docx contient à la fois 7 numéros AVS ET 7 IBAN partiels. Il déclenchera simultanément la règle AVS (→ RH-Confidentiel) et la règle IBAN (→ Finances-Confidentiel). C'est le cas de test idéal pour montrer comment Purview gère les conflits de règles d'auto-labelling.

Site Finances : accès GRP-Finances + GRP-Direction (lecture)

Fichiers financiers contenant des IBAN et données comptables. Ces fichiers déclenchent la règle IBAN dans l'auto-labelling Purview.

| **Fichier** | **Contenu** | **Données sensibles détectées par Purview** |
| --- | --- | --- |
| 06_Contrat_Maintenance_Etude_Rochat.docx | Contrat maintenance client, Étude Rochat & Associés, Vevey | Confidentiel client |
| 10_Comptabilite_Axonix_2024-2025.xlsx | Comptabilité annuelle, bilan, compte de résultat 2024-2025 | Confidentiel finances |
| 17_Grand_Livre_Comptable_Axonix_2025.xlsx | Grand livre comptable 2025, toutes les écritures détaillées | 🔴 IBAN (écritures bancaires) |
| 18_Rapport_Cloture_S1_2025.docx | Rapport clôture S1 2025, CA CHF 699'300, IBAN société | 🔴 IBAN : CH12 0076 2011 0000 1111 2 |

Site Intranet : accès tous les employés

Document de référence technique interne.

| **Fichier** | **Contenu** | **Données sensibles** |
| --- | --- | --- |
| 02_Procedure_AD_EntraID_Hybride.docx | Procédure gestion Active Directory et Entra ID hybride, usage techniciens | Confidentiel interne IT |

## 6.2 Procédure de dépôt dans SharePoint

1. Décompressez le fichier FICHIERS_DEMO_AXONIX.zip sur votre PC. Vous obtenez un dossier avec les 18 fichiers.

2. Ouvrez le site SharePoint de destination dans votre navigateur (ex : https://demo.sharepoint.com/sites/RH).

3. Cliquez sur Documents dans le menu gauche du site.

4. Cliquez sur Télécharger (upload) en haut de la bibliothèque, puis sélectionnez les fichiers du groupe correspondant.

5. Répétez pour chaque site (RH, Finances, Direction, Communication, Intranet).

> **✅  CONSEIL**** :**** UPLOADER AVEC LE BON COMPTE** Connectez-vous avec un compte membre du bon groupe pour uploader : rh1@demo.ch pour le site RH, finance1@demo.ch pour Finances, direction1@demo.ch pour Direction. Cela simule un comportement réaliste et ancre correctement l'historique d'activité pour la démo Purview. Pour Communication et Intranet, n'importe quel compte peut uploader.

> **ℹ️  DÉLAI AVANT DÉTECTION PAR PURVIEW** Après le dépôt des fichiers, l'auto-labelling Purview analyse les contenus de façon asynchrone. Comptez 1 à 24 heures avant que les premières étiquettes automatiques apparaissent sur les fichiers. Déposez les fichiers AVANT de commencer la configuration Purview pour que l'analyse soit déjà en cours.

---

[← Préparation — Activer l'audit Microsoft 365](prep-05-audit.html) | [Préparation — Annexes →](prep-annexes.html)
