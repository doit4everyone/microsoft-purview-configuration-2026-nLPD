---
title: "Partie 7 — Gestion opérationnelle et démos | Configuration Microsoft Purview 2026"
description: "Gestion opérationnelle de Microsoft Purview au quotidien : suspendre les protections pour une démo, vérifications hebdomadaires, résolution des problèmes courants et pilotage de la dette de labellisation."
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

# Partie 7 — Gestion opérationnelle et démos

## 7.1 Suspendre les protections pour une démo

| **Méthode** | **Comment faire** | **Risque** |
| --- | --- | --- |
| Mode simulation DLP | DLP → votre politique → Modifier → Mode : Simulation | ✅ Aucun — recommandé |
| Exclure un utilisateur | DLP → politique → Modifier → Emplacements → Exclure → votre compte test | ⚠️ Faible : 1 utilisateur |
| Désactiver une règle | DLP → politique → Règles → toggle OFF sur la règle spécifique | ⚠️ Moyen |
| Désactiver la politique | DLP → politique → Mode : Désactivé | 🚨 Élevé — aucune protection |

L’accès aux documents protégés par chiffrement est évalué dynamiquement. Le retrait d’un utilisateur d’un groupe associé à une étiquette entraîne la perte immédiate d’accès à l’ensemble des documents protégés, y compris ceux reçus antérieurement par courrier électronique ou déjà téléchargés.

## 7.2 Vérifications hebdomadaires recommandées

| **Vérification** | **Où dans Purview** |
| --- | --- |
| Nouveaux événements DLP | Prévention des pertes de données → Activity Explorer |
| Interactions Copilot inhabituelles | DSPM pour l'IA → Activités de l'IA |
| Documents non étiquetés dans SharePoint RH/Finances | Protection des informations → Content Explorer |
| Rapport hebdomadaire DSPM | Email automatique si configuré en 5.1 |

## 7.3 Résoudre les problèmes courants

| **Problème** | **Cause probable** | **Solution** |
| --- | --- | --- |
| Page Purview inaccessible | Rôle manquant | Vérifiez les 5 groupes Purview RBAC (section 0.3) |
| Bouton audit grisé | Rôle Organization Management manquant | Ajoutez-vous au groupe Exchange (section 0.3) |
| PowerShell = False malgré la commande | Rôle Audit Logs manquant | Vérifiez Organization Management Exchange |
| Étiquettes invisibles dans Word après un délai variable | Politique d'étiquettes non publiée | Purview → Politiques d'étiquettes → vérifiez le statut |
| Auto-labelling ne détecte rien | Fichiers pas dans les emplacements configurés | Vérifiez les sites SharePoint dans la stratégie |
| DLP ne bloque pas | Politique en mode simulation | Changez le mode en Activé |
| Copilot non bloqué malgré DLP | Documents pas encore étiquetés | Attendez que l'auto-labelling applique les labels (délai variable selon le tenant) |
| Content Explorer vide | Rôle Content Viewer manquant | Ajoutez-vous aux 2 groupes Content Explorer (section 0.3) |

## 7.4 Support : Comprendre l’accès pour les partenaires externes

Le chiffrement Purview ne nécessite pas que vous transmettiez un mot de passe au partenaire. L’accès est géré automatiquement par Microsoft via le authentification sécurisée (compte Microsoft/Entra ID ou code email). Voici les points essentiels pour répondre aux questions des utilisateurs et éviter les contournements de sécurité.

| **Question / Situation** | **Réponse / Conseil support** |
| --- | --- |
| **Accès du partenaire externe** | Le code est généré automatiquement par Microsoft au moment où le partenaire clique sur le lien, uniquement s’il ne possède pas de compte Microsoft ou Entra ID. |
| **Durée de validité** | Si un code email est utilisé, il est valable 15 minutes. Après expiration, le partenaire doit cliquer à nouveau sur Envoyer le code. |
| **Le partenaire ne reçoit pas le code** | Vérifiez les spams. L’email provient de microsoft.com. Si le problème persiste, demandez au partenaire d’essayer avec un autre navigateur ou de vider le cache. |
| **Conseil support général** | Recommandez la version Web (navigateur) plutôt que l’application de bureau pour une authentification invité (navigateur recommandé). Rappel : ne jamais transmettre un code de vérification reçu par email via un autre canal — il est à usage unique et lié à la session. |

## 7.5 Identifier et piloter la dette de labellisation

Dans un tenant Microsoft 365 actif depuis plusieurs années et sans gouvernance initiale des données, il est normal qu’une dette de labellisation existe : documents historiques, fichiers métier, contenus transférés ou hérités. L’objectif de cette section est de décrire la méthode pour identifier et piloter cette dette, en tenant compte des limitations connues de Purview.

**Rappel de la limitation Purview :**** **l’Explorateur de données (Content Explorer) n’offre pas de filtre 
« étiquette de sensibilité = aucune ». L’identification des fichiers non étiquetés repose donc sur une approche combinée, décrite ci-dessous en 4 méthodes.

Méthode 1 : Analyse des contenus déjà étiquetés (vue positive)

Purview → Protection des données → Explorateur de données : analysez chaque étiquette de sensibilité existante (RH-Confidentiel, Interne, Direction, etc.) individuellement. Observez les volumes de fichiers étiquetés par emplacement (SharePoint, OneDrive), par site et par bibliothèque. Cette étape fournit une cartographie fiable des données déjà gouvernées.

Méthode 2 : Comparaison avec l’inventaire réel des emplacements

Comparez les volumes observés dans l’Explorateur de données avec l’inventaire réel des sites et bibliothèques SharePoint (rapports SharePoint/OneDrive), en priorité sur les emplacements sensibles : sites RH, Finances, Direction et OneDrive des fonctions à risque. Tout écart significatif entre le volume total de fichiers et le volume visible comme étiqueté constitue de la dette de labellisation à qualifier.

Méthode 3 : Auto-labellisation en mode simulation (rattrapage ciblé)

Pour identifier précisément les fichiers sensibles non étiquetés, relancez les stratégies d’auto-labellisation existantes en mode **Simulation** (données RH : AVS, données personnelles ; données financières : IBAN ; données médicales). La simulation identifie les fichiers qui devraient être étiquetés selon les règles définies, sans modifier les documents et sans impact utilisateur. Tout fichier détecté en simulation sans étiquette existante est un candidat prioritaire au rattrapage.

Méthode 4 : Gouvernance par emplacement

Certains documents sensibles ne contiennent pas de motifs détectables automatiquement (contenu stratégique, juridique, direction). Pour ces cas, la gouvernance par emplacement s’applique : sur les sites identifiés comme sensibles, tout fichier sans étiquette est considéré non conforme par défaut, un étiquetage manuel ou une confirmation utilisateur est exigé. Cette approche complète les méthodes automatisées pour les données non structurées.

Résultat attendu et fréquence

Cadence recommandée : revue trimestrielle. À l’issue de chaque cycle, la dette de labellisation est identifiée, quantifiée et localisée ; les zones à risque sont priorisées ; les actions correctives (auto-labellisation, étiquetage manuel, gouvernance renforcée) sont planifiées et tracées dans le registre nLPD. L’objectif mesurable à atteindre est un taux de couverture des sensitivity labels **supérieur à 80 %** sur les emplacements sensibles, vérifiable via l’Explorateur de données et corrélé avec les rapports SharePoint/OneDrive.

Cette démarche constitue le point de transition vers la mise en oeuvre progressive de stratégies d’auto-labellisation complémentaires et la réduction continue de la dette dans le temps cohérente avec l’approche itérative documentée en section 7.2.

Indicateurs minimaux de suivi

Ces trois indicateurs constituent le tableau de bord minimal pour piloter la dette de labellisation dans le temps. Ils sont mis à jour lors de chaque revue trimestrielle et documentés dans le registre nLPD.

| **Indicateur** | **Fréquence** | **Seuil d’alerte** |
| --- | --- | --- |
| % de fichiers sans étiquette sur les sites sensibles (RH, Finances, Direction) | Trimestrielle | À définir par l’organisation (objectif recommandé : couverture > 80 %) |
| Évolution vs trimestre précédent (hausse / stable / baisse) | Trimestrielle | Hausse > 5 % → action corrective immédiate |
| Taux de couverture global des sensitivity labels | Trimestrielle | < 70 % → revue prioritaire à planifier |

---

[← Partie 6 — Scénarios de test et validation](07-tests-validation.html) | [Partie 8 — Fonctionnalités avancées Purview →](09-fonctionnalites-avancees.html)
