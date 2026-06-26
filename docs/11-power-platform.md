---
title: "Partie 10 — Gouvernance Copilot Studio et Power Platform | Configuration Microsoft Purview 2026"
description: "Gouvernance des agents Copilot Studio et des connecteurs Power Platform : politique DLP connecteur, inventaire des agents et flux existants, Managed Environments."
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

# Partie 10 — Gouvernance Copilot Studio et Power Platform

Cette partie couvre la gouvernance des agents Copilot Studio et des connecteurs Power Platform. Commencez par la section 10.2 (inventaire) avant d’appliquer la section 10.1 (blocage), la politique DLP connecteur s’active immédiatement et sans mode simulation. Les sections 10.2 et 10.3 s’appliquent aussi si des agents sont déjà publiés ou en cours de construction.
## 10.1 Politiques DLP connecteur : inclus Business Premium

La politique DLP connecteur Power Platform définit quels connecteurs sont autorisés dans les flux Power Automate et agents Copilot Studio. Elle est indépendante de Purview et se configure dans le Power Platform Admin Center.

🚨 **PRÉREQUIS OBLIGATOIRE : **effectuez l’inventaire de la section 10.2 AVANT de créer cette politique. Elle est active immédiatement et sans mode simulation : tout flux utilisant un connecteur bloqué cessera de fonctionner dès la création.

Accéder aux stratégies de données

admin.powerplatform.microsoft.com → icône Sécurité (cadenas) → Données et confidentialité → Stratégie de données → + Nouvelle stratégie.

ℹ️ **Accès refusé : **attribuez-vous le rôle Administrateur de Power Platform via entra.microsoft.com → Rôles et administrateurs → Administrateur de Power Platform → + Ajouter des affectations. Attendez 5 à 10 minutes.

Procédure : créer la stratégie DLP-ConnecteursProd

Page 1 : Nom : saisissez DLP-ConnecteursProd → Suivant.

Page 2 : Connecteurs prédéfinis : Passer à Entreprise (via 3 points ⋮ de chaque connecteur) : SharePoint, OneDrive Entreprise, Microsoft Teams, Microsoft Exchange, Office 365 Outlook, Groupes Office 365, Office 365 Groups Mail, Utilisateurs d’Office 365, Microsoft Entra ID, OneNote (professionnel).

⚠️ **Blocable = Non : **SharePoint, OneDrive Entreprise, Microsoft Teams, Office 365 Outlook, Groupes Office 365, Office 365 Groups Mail, Utilisateurs d’Office 365 et OneNote ne peuvent pas être bloqués. Le bouton Bloquer est grisé pour ces connecteurs : utilisez uniquement Passer à Entreprise.

Page 2 : Bloquer (via 3 points ⋮) : HTTP, HTTP Webhook, HTTP with Microsoft Entra ID (pré-autorisé), When a HTTP request is received, Knowledge source with public websites and data in Copilot Studio, Knowledge source with SharePoint and OneDrive in Copilot Studio.

ℹ️ **Tout le reste : **les 1620+ connecteurs restants (Dynamics 365, Azure, Salesforce, canaux Copilot Studio, etc.) restent en Non professionnel par défaut. Ne les touchez pas.

Page 3 : Connecteurs personnalisés : laissez la règle par défaut Ordre 1 / Ignorer / * → Suivant.

Page 4 : Étendue : sélectionnez Ajouter tous les environnements → Suivant.

Page 5 : Revoir : vérifiez (10) Entreprise, (1620+) Hors entreprise, (6) Bloqué, Étendue = Ajouter tous les environnements. Cliquez sur Créer une stratégie. La politique est active immédiatement.

## 10.2 Inventaire des agents et flux existants, inclus Business Premium

Avant d’appliquer la politique DLP connecteur, inventoriez les flux Power Automate et agents Copilot Studio existants pour confirmer qu’aucun n’utilise un connecteur qui sera bloqué.

Flux Power Automate

admin.powerplatform.microsoft.com → icône Gérer → Power Automate → onglet Inventaire. Pour chaque flux : cliquez sur son nom pour vérifier si le connecteur HTTP est utilisé.

ℹ️ **Tenant Axonix SA : **1 flux existant : « Avertir un canal que… » (propriétaire demo, créé le 22/03/2025, Suisse). Ce flux n’utilise pas le connecteur HTTP. Aucun risque.

Agents Copilot Studio

admin.powerplatform.microsoft.com → icône Gérer → Copilot Studio → onglet Assistants. Consultez également l’onglet Flux d’assistant.

ℹ️ **Tenant Axonix SA : **0 assistant Copilot Studio. Appliquez la politique de la section 10.1 maintenant : elle protège le tenant avant que le premier agent soit créé.

⚠️ **Si un flux ou agent utilise HTTP : **contactez son propriétaire. Demandez si une alternative Microsoft 365 (SharePoint, Exchange, Teams) peut le remplacer. Appliquez la politique après migration.

## 10.3 Managed Environments. Power Apps Premium requis

Les Managed Environments nécessitent une licence Power Apps Premium ou Power Automate Premium : hors périmètre Business Premium + Purview Suite. Avec Business Premium, vous pouvez néanmoins restreindre qui peut créer des environnements.

Restreindre la création d’environnements

admin.powerplatform.microsoft.com → icône Gérer → Paramètres du client → section Qui peut créer des environnements de production et sandbox → sélectionnez Uniquement les administrateurs spécifiques → Enregistrer.

ℹ️ **Environnement par défaut Axonix SA : **Répertoire par défaut (Suisse, type Par défaut, Dataverse actif, non géré). DLP-ConnecteursProd s’applique à cet environnement car l’étendue a été configurée sur Ajouter tous les environnements.

---

[← Partie 9 — Endpoint DLP, Edge for Business et AI Hub](10-endpoint-dlp-aihub.md) | [Partie 11 — Gouvernance des accès SharePoint →](12-sharepoint.md)
