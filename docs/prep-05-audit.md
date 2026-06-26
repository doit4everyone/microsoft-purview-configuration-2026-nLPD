---
title: "Préparation — Activer l'audit Microsoft 365 | Configuration Microsoft Purview 2026"
description: "Vérifier et activer l'audit Microsoft 365 (Unified Audit Log), première étape requise par le guide Purview."
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

# Partie 5 — Activer l'audit Microsoft 365

L'activation de l'audit est la première étape du guide Microsoft Purview (Partie 0 du guide). Elle est placée ici car elle doit être activée le plus tôt possible. Les logs ne sont enregistrés qu'à partir du moment où l'audit est actif.

> **⚠****️  ACTIVEZ**** L'AUDIT MAINTENANT ****-**** PAS PLUS TARD** Les logs d'audit ne sont pas rétroactifs. Tout ce qui se passe avant l'activation n'est pas enregistré. Le guide Purview vérifie que l'audit est actif dès sa Partie 0. Si ce n'est pas fait, certaines sections ne fonctionneront pas.

## 5.1 Vérifier et activer l'audit

1. Allez sur https://compliance.microsoft.com.

2. Dans le menu gauche, cherchez Audit et cliquez dessus.

3. Si vous voyez un bouton bleu « Démarrer l'enregistrement des activités utilisateur et admin », cliquez dessus.

4. Si vous voyez « L'enregistrement est activé », rien à faire, l'audit est déjà actif.

5. Attendez quelques minutes et rafraîchissez la page pour confirmer que le statut est Activé.

> **✅  VÉRIFICATION POWERSHELL (OPTIONNEL)** Pour confirmer l'activation via PowerShell :

Connect-ExchangeOnline

Get-AdminAuditLogConfig | Select-Object UnifiedAuditLogIngestionEnabled

`# Résultat attendu : True`

Disconnect-ExchangeOnline -Confirm:$false

---

[← Préparation — Création des sites SharePoint](prep-04-sites-sharepoint.html) | [Préparation — Documents de démonstration →](prep-06-documents-demo.html)
