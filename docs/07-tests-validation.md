---
title: "Partie 6 — Scénarios de test et validation | Configuration Microsoft Purview 2026"
description: "Scénarios de test pour valider le déploiement Purview : auto-labelling, blocage DLP du partage externe, protection Copilot et checklist de validation finale."
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

# Partie 6 — Scénarios de test et validation

> ⚠️ **ATTENTION : **Avant de commencer les tests : vérifiez que les politiques DLP et auto-labelling sont bien activées (pas en simulation) et que l'audit est actif. Attendez 24-48h après activation avant de tester.

> 📦 **Fichiers de démo requis :** les tests ci-dessous utilisent des fichiers fictifs (contrat de travail, fiches de salaire, registre du personnel, grand livre). Téléchargez `FICHIERS_DEMO_AXONIX.zip` depuis la [release v1.0](https://github.com/doit4everyone/microsoft-purview-configuration-2026-nLPD/releases/tag/v1.0) et uploadez-les dans les sites SharePoint correspondants (RH, Finances) avant de commencer. Le détail de la répartition par site est décrit dans la [page de préparation du tenant](prep-06-documents-demo.html).

## 6.1 Test 1 Vérifier l'auto-labelling

1. **Uploader un document** Ouvrez le site SharePoint RH dans votre navigateur → Documents → faites glisser le fichier 11_Contrat_Travail_Beraz_Valerie_RH.docx dans la bibliothèque.

2. **Attendre 24-48 heures** L'auto-labelling tourne en arrière-plan. Revenez le lendemain.

3. **Vérifier le label** Dans SharePoint RH → Documents → cliquez sur les trois points ... à droite du fichier → Détails. Dans le panneau Détails à droite, vous devez voir : Étiquette de sensibilité = 4 - RH-Confidentiel.

4. **Vérifier dans Activity Explorer** Purview → Activity Explorer → filtrez Type d'activité = Étiquette appliquée automatiquement. Le fichier doit apparaître dans la liste.

> ⚠️ **ATTENTION — **Si le label n'apparaît pas après ce délai : vérifiez que la stratégie d'auto-labelling est bien en mode Activé (pas Simulation) et que le site SharePoint RH est bien inclus dans les emplacements de la stratégie.

## 6.2 Test 2 — DLP bloque le partage externe

1. **Ouvrir Outlook Web** Allez sur https://outlook.office.com et connectez-vous avec votre compte de test (rh1@demo.ch).

2. **Créer un email de test** Cliquez sur + Nouveau message. Dans le champ À : entrez une adresse email externe, votre email personnel Gmail ou Outlook.com personnel.

3. **Joindre le contrat RH** Cliquez sur le trombone → Parcourir cet ordinateur (ou Joindre des fichiers) → sélectionnez le fichier 11_Contrat_Travail_Beraz_Valerie_RH.docx.

4. **Essayer d'envoyer** Cliquez sur Envoyer. Résultat attendu : un message de conseil de stratégie DLP apparaît dans l'email, indiquant que le document contient des données sensibles. Si le blocage est strict, l'envoi est bloqué.

5. **Vérifier dans Activity Explorer** Attendez quelques minutes puis allez dans Purview → Activity Explorer → filtrez Type = Correspondance de règle DLP. L'événement doit apparaître.

> ⚠️ **ATTENTION — **Si aucun blocage n'apparaît : vérifiez que la politique DLP est en mode Activé et non Simulation. En mode simulation, les règles ne bloquent pas.

## 6.3 Test 3 — Protection Copilot

1. **Ouvrir Copilot M365** Allez sur https://microsoft365.com → connectez-vous avec votre compte de test → cliquez sur l'icône Copilot en haut à droite ou dans le menu.

2. **Poser la question** Tapez exactement : Résume le contrat de travail 11_Contrat_Travail_Beraz_Valerie_RH.docx

3. **Résultat attendu avec Purview actif** Résultat attendu : Copilot limite ou refuse l’extraction du contenu sensible, conformément aux permissions et protections appliquées (chiffrement RMS ou politique DLP active). Le comportement exact peut varier, refus complet, résumé partiel sans données sensibles, ou message d’avertissement.

4. **Comparer avec sans Purview** Si vous voulez voir la différence : allez dans DLP → votre politique → passez en mode simulation → refaites la même question à Copilot. Il répondra librement avec les données sensibles. Puis réactivez la politique.

## 6.4 Checklist de validation finale

| **✓** | **Ce qui doit fonctionner** | **Où vérifier** |
| --- | --- | --- |
| ✅ | 7 étiquettes créées | Purview → Protection infos → Étiquettes |
| ✅ | Politique d'étiquettes publiée | Purview → Protection infos → Politiques d'étiquettes |
| ☐ | Étiquettes visibles dans Word après un délai variable | Ouvrir un doc Office → barre Sensibilité |
| ✅ | Stratégie auto-labelling activée | Purview → Étiquetage automatique → Statut |
| ✅ | Auto-labelling détecte les documents test | Résultats de simulation → fichiers 11-18 |
| ✅ | Politique DLP principale activée | Purview → DLP → Stratégies → Statut |
| ✅ | DLP-Protection-Copilot activée | Purview → DLP → Stratégies |
| ✅ | DLP-Protection-Transmission-Externe activée (2 règles) | Purview → DLP → Stratégies |
| ✅ | Audit actif PowerShell = True | Get-AdminAuditLogConfig │ FL |
| ☐ | DSPM (aperçu) — Tâches de configuration vérifiées | Purview → DSPM → Vue d'ensemble |
| ✅ | Test DLP email externe bloqué | Activity Explorer → Correspondance DLP |
| ✅ | Test Copilot protégé | Copilot ne résume pas les docs confidentiels |

---

[← Partie 5 — Audit et DSPM for AI (surveillance Copilot)](06-audit-dspm-ai.html) | [Partie 7 — Gestion opérationnelle et démos →](08-gestion-operationnelle.html)
