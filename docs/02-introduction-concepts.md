---
title: "Partie 1 — Introduction et concepts Purview | Configuration Microsoft Purview 2026"
description: "Qu'est-ce que Microsoft Purview, pourquoi il est indispensable avec Copilot M365, les 4 piliers de la configuration (étiquettes, auto-labelling, DLP, audit), l'ordre de configuration obligatoire et l'activation de l'audit Microsoft 365."
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

# Partie 1 — Introduction et concepts Purview

## 1.1 Qu'est-ce que Microsoft Purview ?

Microsoft Purview est la suite de conformité et de protection des données de Microsoft, accessible via le portail https://compliance.microsoft.com. Elle regroupe tous les outils nécessaires pour classifier, protéger, surveiller et gouverner vos données dans Microsoft 365, emails, fichiers SharePoint, OneDrive, Teams, et les applications IA comme Copilot.

> ℹ️ **INFO : **Purview ne crée pas de nouvelles données, il protège celles qui existent déjà. C'est une couche de gouvernance qui s'applique par-dessus vos services existants.

## 1.2 Pourquoi Purview est indispensable avec Copilot M365

Microsoft 365 Copilot respecte les permissions d’accès aux données (SharePoint, OneDrive, Exchange) ainsi que les protections appliquées via les étiquettes de sensibilité, notamment le chiffrement. Les politiques DLP viennent en complément pour détecter, alerter ou bloquer certains usages (partage, exfiltration), mais ne constituent pas le mécanisme principal de restriction de Copilot. En conséquence, toute donnée accessible à un utilisateur peut potentiellement être exploitée par Copilot si elle n’est pas correctement protégée. C’est pourquoi la gouvernance des accès (principe du moindre privilège) est indispensable en complément de Purview.

⚠️ Point critique : Permissions et accès aux données

Microsoft Purview ne modifie pas les permissions d’accès aux données. Si un document est accessible à un utilisateur (via SharePoint, OneDrive ou Teams), il reste accessible indépendamment de son étiquette. En présence de sur-partage (ex : site SharePoint ouvert à tous), les données peuvent être exploitées par les utilisateurs et par Copilot, même si elles sont classifiées. Une gouvernance des accès (principe du moindre privilège) est indispensable en complément de Purview. La Partie 11 de ce guide explique comment la mettre en place dans SharePoint.

⚠️ Limites connues de la solution

L’auto-labelling peut générer des faux positifs ou manquer certains contenus. Les politiques peuvent mettre du temps à s’appliquer (délai variable selon le tenant). Certaines actions peuvent contourner les contrôles notamment la capture d’écran ou la photo d’écran avec un téléphone. Purview doit être considéré comme un composant de gouvernance et non comme une protection absolue.

## Positionnement de Microsoft Purview

Microsoft Purview est une solution de gouvernance, de conformité et de protection des données. Elle ne remplace pas les solutions de sécurité active (protection contre les attaques, détection des menaces), qui relèvent d’autres services Microsoft tels que Microsoft Defender. Ces deux couches sont complémentaires et non substituables.

> 🚨 **IMPORTANT : ****Point d’attention nLPD, si des mesures de protection adaptées ne sont pas mises en place (art. 8 nLPD), la responsabilité peut engager la personne en charge et, en cas de manquement grave, conduire à une amende pouvant aller jusqu’à CHF 250'000.**

 

## 1.3 Les 4 piliers de Purview que vous allez configurer

| **Pilier** | **Ce que ça fait** | **Partie de ce guide** |
| --- | --- | --- |
| **1 : Étiquettes de sensibilité** | Classifier les documents selon leur niveau de confidentialité | Partie 2 |
| **2 : Auto-labelling** | Détecter et classifier automatiquement les données sensibles (AVS, IBAN) | Partie 3 |
| **3 : DLP** | Bloquer la fuite des données sensibles vers l'extérieur | Partie 4 |
| **4 : Audit ****&**** DSPM for AI** | Tracer qui accède à quoi, surveiller Copilot | Partie 5 |

Les Parties 6 à 11 (scénarios de test, gestion opérationnelle, fonctionnalités avancées, Endpoint DLP, gouvernance Power Platform et accès SharePoint) complètent ces 4 piliers fondamentaux et peuvent être déployées progressivement après le socle.

## 1.4 Ordre de configuration obligatoire

1. **Prérequis rôles et PowerShell (Partie 0)** À faire AVANT TOUT : sans les bons rôles, rien ne fonctionnera.

2. **Activer l'audit (section 1.5)** À faire EN DEUXIÈME : sans audit actif, vous ne pouvez pas vérifier que vos politiques fonctionnent.

3. **Créer les étiquettes (Partie 2)** Créer les SIT personnalisés. Définir vos niveaux de classification avant tout le reste.

4. **Configurer l'auto-labelling (Partie 3)** Détecter automatiquement AVS, IBAN, données médicales.

5. **Créer les politiques DLP (Partie 4)** Bloquer la fuite des données une fois les étiquettes en place.

6. **Configurer DSPM for AI (Partie 5.1)** Surveiller Copilot : dernière étape.

## 1.5 Activer l'audit : À faire EN PREMIER

> 🚨 **IMPORTANT : **L’audit Microsoft 365 (Unified Audit Log) n’est pas systématiquement activé selon les environnements notamment sur les tenants de test ou récents. La méthode la plus fiable pour garantir son activation reste l’utilisation de PowerShell via Exchange Online. L’audit doit être activé AVANT toute autre configuration sans audit actif, vous ne pouvez pas vérifier que vos politiques fonctionnent et vous n’avez aucune trace en cas d’incident nLPD.

### 1.5.1 Installer et connecter le module Exchange Online

Ouvrez PowerShell en administrateur (voir section 0.2) et tapez ces commandes une par une :

**Étape 1 : Installer le module (une seule fois)**

Install-Module -Name ExchangeOnlineManagement -Force

*→ Si on vous demande de faire confiance au dépôt PSGallery, tapez O et Entrée.*

**Étape 2 : Se connecter à Exchange Online**

Connect-ExchangeOnline

*→ Une fenêtre de connexion Microsoft s'ouvre. Connectez-vous avec votre compte admin. Attendez le message de confirmation dans PowerShell.*

**Étape 3 : Activer l'audit**

Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true

*→ Aucune réponse = succès. Si vous voyez une erreur rouge, vérifiez votre rôle Organization Management Exchange (section 0.3).*

**Étape 4 : Vérifier que l'audit est actif **

Get-AdminAuditLogConfig | FL UnifiedAuditLogIngestionEnabled

**→ Résultat ****attendu :**** ****UnifiedAuditLogIngestionEnabled :**** True**

*→ Si vous voyez False : déconnectez-vous, reconnectez-vous et réessayez l'étape 3.*

**Étape 5 : Se déconnecter proprement **

Disconnect-ExchangeOnline -Confirm:$false

> ✅ **CONSEIL : **Si la valeur est True → vous pouvez continuer. Si elle est encore False après 3 tentatives → vérifiez que vous êtes bien membre du groupe Organization Management dans le Centre d'administration Exchange (section 0.3).

### 1.5.2 Vérifier l'audit dans le portail Purview

1. Dans Purview → menu de gauche → Audit.

2. Statut attendu : un bandeau vert indique que l'audit est activé.

3. Si vous voyez un bouton Démarrer l'enregistrement → cliquez dessus en complément de PowerShell.

| **Ce qui est audité** | **Rétention avec Purview Suite** |
| --- | --- |
| Connexions utilisateurs | 1 an (vs 90 jours sans add-on) |
| Accès aux fichiers SharePoint/OneDrive | 1 an |
| Interactions Copilot, prompts et réponses | 1 an |
| Déclenchements DLP | 1 an |
| Modifications d'étiquettes | 1 an |

Les journaux d’audit sont conservés pendant une durée maximale pouvant aller jusqu’à 12 mois selon la licence et la configuration du tenant. Cette durée est définie conformément au principe de proportionnalité de la nLPD et permet la détection d’incidents de sécurité, l’analyse des accès et la constitution de preuves en cas de violation avant suppression ou anonymisation automatique des données.

La durée de conservation des journaux fait l’objet d’une réévaluation périodique en fonction de l’évolution des risques, des obligations légales et des recommandations des autorités compétentes.

---

[← Partie 0 — Avant de commencer](01-avant-de-commencer.md) | [Partie 2 — Les Étiquettes de sensibilité (Labels) →](03-etiquettes-sensibilite.md)
