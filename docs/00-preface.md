---
title: "Préface | Configuration Microsoft Purview 2026"
description: "Préface du guide de configuration Microsoft Purview 2026 : contexte du projet, chronologie de déploiement en 4 semaines, positionnement de Purview face à l'antivirus, au SIEM et aux permissions, et limites du dispositif."
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

# Préface

J’ai entrepris cette procédure dans le cadre de ma préparation à l’examen SC-401, avec l’intention de déployer et d’étudier Microsoft Purview en conditions réelles.

Au départ, venant du monde de l’infrastructure, je n’avais aucune expérience de Purview. L’ensemble de la solution m’a d’abord semblé complexe et peu intuitif, puis, à mesure que la configuration avançait, les concepts se sont progressivement mis en place.

Le résultat est une procédure exhaustive couvrant l’ensemble des fonctionnalités pertinentes pour une PME Suisse, avec pour ambition de représenter au plus près les exigences réelles du terrain. Toutes les configurations ont été déployées et validées sur un tenant de test en mode hybride entre mars et avril 2026, directement dans l’interface Purview.

Pour toute organisation souhaitant déployer Microsoft Purview, vous trouverez ici tout ce dont vous aurez besoin. Si vous débutez, sachez qu’il est possible de créer un environnement de preuve de concept avec des licences d’évaluation Microsoft 365. Vous disposerez de deux mois pour déployer l’ensemble des configurations, les tester et acquérir la maîtrise opérationnelle de la solution.

## Chronologie de déploiement

**Semaine 1 : **Socle tenant : groupes de rôles, licences Purview Suite, audit, SITs personnalisés (AVS-Suisse-PME, Medical-RH-Suisse).

**Semaine 2 : **Étiquettes de sensibilité (7 étiquettes, publication) - Auto-labellisation (simulation + activation) - Politiques de rétention (5 politiques).

**Semaine 3 : **Politiques DLP (Exchange, SharePoint, Copilot, Transmission externe) - Endpoint DLP + Edge for Business.

**Semaine 4 : **DSPM for AI - Insider Risk Management (3 stratégies) - eDiscovery Premium - Communication Compliance - Étiquette 6 « Transmission-Externe ».

## Positionnement de Microsoft Purview

Microsoft Purview est un outil de gouvernance et de protection des données, pas un outil de sécurité périmétrique. Il est fondamental de comprendre ce qu’il fait et ce qu’il ne remplace pas. Purview suppose des fondations saines.

**Antivirus / EDR : **Purview ne détecte pas les logiciels malveillants ni les menaces sur les postes. Microsoft Defender ou un équivalent reste un prérequis indépendant.

**SIEM : **La corrélation des événements de sécurité et la détection des attaques relèvent d’une couche distincte (Microsoft Sentinel ou équivalent).

**Structure de permissions : **Si un utilisateur a accès à un fichier qu’il ne devrait pas voir, Purview ne corrige pas cette erreur de gouvernance. Il peut alerter, mais il ne bloque pas un accès considéré comme légitime selon les permissions SharePoint en place. Une structure de droits cohérente sur les sites, bibliothèques et dossiers est un préalable indispensable.

**Pare-feu et proxy réseau : **Le contrôle du trafic sortant vers des sites non autorisés relève d’une couche réseau dédiée. Purview complète cette protection au niveau des données, il ne la remplace pas.

**Sauvegarde et continuité : **La rétention Purview n’est pas une solution de backup. Elle garantit la conservation légale des données, pas leur restauration en cas de sinistre.

Ce que Purview apporte c’est une couche de protection centrée sur la donnée elle-même. Les étiquettes et les contrôles voyagent avec le document, quel que soit l’endroit où il se trouve. C’est une protection complémentaire, pas de substitution. Déployé sur des fondations fragiles, Purview donnera une fausse impression de sécurité. Déployé correctement, il constitue une défense en profondeur efficace et démontrable face au PFPDT.

## Note de limitation de responsabilité

Ce guide a été rédigé et validé terrain sur la base des interfaces Microsoft 365 et Microsoft Purview telles qu'elles se présentaient au moment de sa rédaction (avril 2026). Microsoft déploie des mises à jour d'interface de manière continue et sans préavis : les chemins de navigation, les libellés de boutons, la disposition des menus et la disponibilité de certaines fonctionnalités sont susceptibles d'évoluer à tout moment, indépendamment de la volonté de l'auteur.

L'auteur de ce document ne saurait être tenu responsable des divergences entre les procédures décrites et les interfaces effectivement rencontrées lors de l'exécution. En cas de doute sur une étape spécifique, il est recommandé de consulter la documentation officielle Microsoft disponible sur docs.microsoft.com, de contacter le support Microsoft, ou de faire appel à un consultant Microsoft 365 certifié.

Ce guide est fourni à titre informatif et pédagogique uniquement. Il ne constitue pas un avis juridique et ne remplace pas un accompagnement professionnel adapté à votre organisation et à votre contexte réglementaire spécifique. La conformité à la nLPD relève de la responsabilité exclusive de l'organisation qui déploie ces configurations.

---

[← Retour à l'index](../) | [Partie 0 — Avant de commencer →](01-avant-de-commencer.md)
