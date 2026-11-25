---
title: "Partie 4 : Politiques DLP (Data Loss Prevention) | Configuration Microsoft Purview 2026"
description: "Créer les politiques DLP Microsoft Purview : politique principale, protection de la transmission externe et protection Copilot, pour bloquer la fuite de données sensibles."
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

# Partie 4 : Politiques DLP (Data Loss Prevention)

## 4.1 Comprendre la DLP

Les étiquettes classifient les documents, la DLP agit sur les actions. Elle empêche concrètement un utilisateur d'envoyer un document confidentiel par email à l'extérieur, de le partager via Teams, ou de le coller dans un chat.

| **Sans DLP** | **Avec DLP** |
| --- | --- |
| Utilisateur peut envoyer contrat RH à n'importe qui | Envoi bloqué si destinataire hors domaine autorisé |
| Copilot accède aux données selon les permissions utilisateur, sans restriction supplémentaire | DLP alerte ou bloque certains usages ; le chiffrement RMS reste le mécanisme principal de restriction |
| Aucune trace des tentatives de fuite | Chaque tentative loggée dans Activity Explorer |

### 4.1.1 Politique DLP spécifique Copilot

La protection Copilot nécessite une **politique DLP distincte** car c’est le seul emplacement DLP où les étiquettes de sensibilité peuvent être utilisées directement comme condition de déclenchement. Dans les politiques cloud classiques (Exchange, SharePoint, OneDrive, Teams) cette condition n’est pas disponible. Cette politique vise à limiter l’extraction, la synthèse et la restitution de contenus sensibles par Microsoft Copilot, même lorsque l’utilisateur dispose des autorisations d’accès au fichier. Elle ne remplace pas les permissions SharePoint ni le chiffrement RMS, elle constitue une mesure complémentaire spécifique à la gouvernance des usages d’intelligence artificielle. Sa configuration est détaillée à la section 4.4.

## 4.2 Créer la politique DLP principale

1. Dans Purview → menu de gauche → Protection contre la perte de données → Stratégies.

2. Cliquez sur + Créer une stratégie puis Appareils et applications d’entreprise

3. Page 1 : choisissez Personnalisé → Personnalisé (tout en bas de liste) → **Suivant.**

4. Page 2 : Nom : **DLP-Protection-nLPD-demo** | Description : Protège les données personnelles sensibles nLPD suisse. → **Suivant.**

5. Page 3 : Unités administratives : Répertoire complet → **Suivant.**

6. Page 4 : Emplacements : activez uniquement les 4 emplacements Microsoft 365 ci-dessous :

| **Emplacement** | **Inclure** |
| --- | --- |
| ✅ Courrier Exchange | Tous les utilisateurs |
| ✅ Sites SharePoint | Tous les sites sauf **Transmissionexterne** → Modifier → Exclure → saisir https://demo.sharepoint.com/sites/Transmissionexterne → + → **Terminé.** Ce site est géré par la politique dédiée DLP-Protection-Transmission-Externe (section 4.3). |
| ✅ Comptes OneDrive | Tous les comptes |
| ✅ Messages de conversation et de canal Teams | Tous les utilisateurs |

⚠️ **Appareils, Instances, Référentiels locaux : **ces emplacements apparaissent dans la liste mais ne doivent pas être activés pour ce POC. Appareils nécessite Endpoint DLP + onboarding Intune (section 9). Instances nécessite une licence supplémentaire. Référentiels locaux nécessite un scanner Purview on-premise.

7. Cliquez sur **Suivant.**

8. Page 5 : Définir les paramètres de stratégie : choisissez Créer ou personnaliser des règles DLP avancées → **Suivant.**

9. Page 6 : Règles personnalisées : cliquez sur + Créer une règle. Créez les 3 règles suivantes :

Règle 1 : Bloquer partage externe AVS

| **Paramètre** | **Ce que vous devez configurer** |
| --- | --- |
| Nom de la règle | Bloquer-AVS-Externe |
| Conditions : Ajouter une condition | Le contenu inclut → + Ajouter → Types d’informations confidentielles → tapez AVS-Suisse → sélectionnez AVS-Suisse │ Confiance moyenne │ nombre min : 1. (SIT personnalisé créé en section 2.3bis.) |
| ET : Ajouter une condition | Du contenu est partagé à partir de Microsoft 365 → Avec des personnes externes à mon organisation |
| Actions : Ajouter une action | Restreindre l'accès ou chiffrer le contenu dans les emplacements Microsoft 365 → Bloquer tout le monde |
| Notifications utilisateur | Activez. Message : Ce document contient un numéro AVS suisse. Partage externe bloqué nLPD art. 8. |
| Remplacements utilisateur | Section Notifications utilisateur : activer “Afficher le conseil de stratégie sous forme de boîte de dialogue avant l’envoi” (Exchange uniquement). Section Remplacements par l’utilisateur : (1) ☑ Permettre aux utilisateurs d’outrepasser les restrictions ; (2) ☑ Exiger une justification pour ignorer ; (3) ☑ Exiger que l’utilisateur final reconnaisse explicitement le remplacement (Exchange uniquement). Laisser décoché : Ignorer la règle si faux positif. |
| Rapports d'incident | Niveau de gravité : Élevé │ Envoyer une alerte aux administrateurs : votre email admin |

Cliquez sur Enregistrer pour sauvegarder la règle

Exemple Notifications utilisateur.

Règle 2 : Bloquer partage externe IBAN

| **Paramètre** | **Ce que vous devez configurer** |
| --- | --- |
| Nom de la règle | Bloquer-IBAN-Externe |
| Condition 1 | Le contenu contient > IBAN > confiance élevée > min : 1 |
| Condition 2 | Du contenu est partagé à partir de Microsoft 365 > Avec des personnes externes à mon organisation |
| Action | Bloquer tout le monde sans possibilité de remplacement |
| Notifications utilisateur | Message : Ce document contient un IBAN. Partage externe bloqué. |
| Remplacements | NE PAS activer les remplacements : blocage strict |

**Pourquoi ce choix est conforme nLPD**

La nLPD n’interdit pas la communication de données personnelles à des tiers : elle exige que le traitement respecte les principes de l’art. 6 (proportionnalité, finalité) et, s’il porte atteinte à la personnalité, qu’il repose sur un motif justificatif (art. 31). En pratique, la communication doit donc être **justifiée, proportionnée et traçable**. Un blocage strict sans override est contre-productif en PME, les utilisateurs contournent systématiquement la restriction via des canaux non surveillés (Gmail personnel, WeTransfer, WhatsApp), ce qui fait perdre toute traçabilité. Le blocage avec override justifié maintient l’utilisateur dans le canal M365 surveillé et génère automatiquement une preuve documentaire utile en cas de contrôle du PFPDT : justification horodatée, identité de l’expéditeur, fichier concerné, tout est enregistré dans l’Explorateur d’activités Purview. La Règle 2 (IBAN) reste en blocage strict sans override, car aucun motif légitime ne justifie l’envoi d’un IBAN en clair par email.

**⚠️ Remplacement avec justification : portée et production**

Le blocage reste la règle : le remplacement avec justification est une exception volontaire, enregistrée avec sa justification. Dans ce POC, il est ouvert à tous les utilisateurs, et cela vaut pour les quatre emplacements de la politique (Exchange, SharePoint, OneDrive, Teams). Ce choix garde les utilisateurs dans l’environnement Microsoft 365 surveillé plutôt que de les pousser vers des services personnels. Pour les fichiers du site RH, les permissions SharePoint limitent en pratique le remplacement aux membres RH et Direction, seuls à pouvoir partager ces fichiers. Pour Exchange, OneDrive et Teams, en revanche, tout utilisateur bloqué peut l’utiliser. En production : examinez régulièrement les justifications dans l’Explorateur d’activités ; orientez les partages externes de données sensibles vers le site Transmission externe (section 4.3) ; et, si votre licence le permet, réservez le remplacement à un **groupe de sécurité DLP-Exceptions** (Direction + RH uniquement). Lors de la rédaction, la condition par groupe n’était pas disponible avec Business Premium : vérifiez sa disponibilité dans votre tenant. Voir Annexe E pour le détail.

> 📌 **RÈGLE D’USAGE ENTRE LES DEUX CANAUX DE SORTIE : **le guide prévoit deux chemins pour faire sortir une donnée sensible, à ne pas confondre. **Le site Transmission externe** (section 4.3) est le canal normal pour tout envoi prévu de documents sensibles à un partenaire (fiduciaire, réviseur, avocat, client) : copie du document, étiquette 6 appliquée manuellement, lien nominatif, rétention de 2 jours. **Le remplacement avec justification** de la règle AVS est réservé aux cas ponctuels et imprévus, par exemple un e-mail urgent contenant un numéro AVS. Chaque justification est enregistrée et doit être examinée ensuite par l’administrateur (Explorateur d’activités, filtre Remplacement de la stratégie DLP par l’utilisateur). Un recours répété au remplacement pour le même type d’échange signale un besoin qui doit passer par le site Transmission externe. Inscrivez cette règle dans la procédure interne ou la charte informatique.

Cliquez sur **Enregistrer.**

Règle 3 : Avertir pour données sensibles en interne

| **Paramètre** | **Ce que vous devez configurer** |
| --- | --- |
| Nom de la règle | Avertir-Données-Sensibles-Interne |
| Condition 1 | Le contenu contient → AVS OU IBAN OU données médicales tapez AVS-Suisse et sélectionner : AVS-Suisse, confiance moyenne. Tapez IBAN et sélectionner : International Banking Account Number (IBAN), confiance élevée. Tapez Medical-RH-Suisse et sélectionner : Medical-RH-Suisse, confiance moyenne │ Nb d’instance 1 |
| Condition 2 | Partagé avec des personnes à l'intérieur de mon organisation |
| Action | Aucun blocage : ne cochez aucune action |
| Notifications utilisateur | Activez les conseils de stratégie : Message : Ce document contient des données sensibles. Vérifiez que le destinataire est autorisé. |

Cliquez sur **Enregistrer.**

10. Une fois les 3 règles créées (Bloquer-AVS-Externe, Bloquer-IBAN-Externe, Avertir-Données-Sensibles-Interne), cliquez sur **Suivant.**

11. Page 7 : Mode de la stratégie : choisissez Exécuter la stratégie en mode simulation → **Suivant.**

12. Page 8 : Vérifiez et cliquez sur **Soumettre.**

> ⚠️ **ATTENTION : **Testez toujours en simulation 1 semaine avant d'activer. Regardez les résultats dans Activity Explorer, si trop de faux positifs, augmentez le niveau de confiance dans les règles.

## 4.3 DLP-Protection-Transmission-Externe

Politique dédiée au site Transmission externe, séparée de la politique principale DLP-Protection-nLPD-demo car ce site a des règles spécifiques (partage vers l’extérieur autorisé sous conditions + surveillance interne). Contient 2 règles : Règle 1 (surveillance traçabilité partages externes), Règle 2 (avertissement interne pour données sensibles). Note : l’étiquette 6 est appliquée manuellement par l’utilisateur (section 2.4) ; cette politique surveille et alerte, elle ne bloque pas le partage. Ce site est le canal normal des envois prévus ; le remplacement justifié de la règle AVS reste réservé aux cas ponctuels (voir la règle d’usage en section 4.2). La logique d’ensemble du site et le parcours d’un document sont décrits dans la préparation du tenant (section 4.7, encadré « Pourquoi un site dédié à la transmission externe ? »). Toutes les règles sont documentées ci-dessous.

1. Purview → Protection contre la perte de données → Stratégies → **+ Créer une stratégie** → Personnalisé → Personnalisé → Suivant.

2. Page Nommer votre stratégie → Nom : **DLP-Protection-Transmission-Externe** | Description : *Protège le site Transmission externe. Surveille les partages externes (traçabilité nLPD) et les accès internes aux données sensibles* → **Suivant**.

3. Unités administratives → Répertoire complet → **Suivant.**

4. Page Emplacements → activez **uniquement** l’emplacement suivant :

**Sites SharePoint : **Modifier → Inclure → saisir **https://demo.sharepoint.com/sites/Transmissionexterne** → + → Terminé. **Ce site uniquement** ne pas activer Exchange ni OneDrive (déjà couverts par la politique principale DLP-Protection-nLPD-demo, doublon évité). → **Suivant.**

6. Page Règles de DLP avancées → créez les 2 règles suivantes une par une en cliquant **+ Créer une règle** après chaque règle enregistrée.

Règle 1 : Surveiller les partages externes (traçabilité nLPD)

| **Paramètre** | **Ce que vous devez configurer** |
| --- | --- |
| Nom de la règle | **Surveiller-Partage-Externe** |
| Condition 1 | Le contenu contient → opérateur du groupe : **N’importe lequel de ces éléments** (OU) → tapez **AVS-Suisse** et sélectionner : **AVS-Suisse**, confiance moyenne. Tapez **IBAN** et sélectionner : **International Banking Account Number (IBAN)**, confiance élevée. Tapez **Medical-RH-Suisse** et sélectionner : **Medical-RH-Suisse**, confiance moyenne, min 3. Un seul de ces types suffit à déclencher la règle. |
| Condition 2 | Du contenu est partagé à partir de Microsoft 365 → Avec des personnes externes à mon organisation |
| Action | Aucun blocage : surveillance uniquement. |
| Notifications utilisateur | Activez par Message : Ce partage externe contient des données sensibles détectées. Il fait l’objet d’une alerte et d’un contrôle interne. |
| Rapports d’incident | Niveau de gravité : Faible │ Alerte admin : Activé │ Email : admin@demo.ch |

*Objectif : détecter et signaler les partages externes de contenus sensibles reconnus par les SIT. Un document sans correspondance SIT peut être partagé sans alerte DLP : la traçabilité de tous les partages, y compris ceux-là, repose sur le journal d’audit Microsoft 365 (activités de partage). Une alerte DLP ne démontre pas à elle seule la conformité juridique de la transmission. L’étiquette 6 appliquée manuellement matérialise la décision de transmettre. Cliquez sur **Enregistrer**.*

**⚠️ Si l’étiquette 6 n’est pas appliquée : **la condition « étiquette n’est pas appliquée » n’existe pas dans les politiques DLP SharePoint. Les garde-fous sont : (1) les permissions SharePoint (seuls les membres des groupes RH, Finances et Direction peuvent déposer, chacun dans sa bibliothèque) ; (2) la détection des SIT par cette politique, qui alerte l’administrateur en cas de partage externe de données sensibles ; (3) la procédure interne qui impose l’application manuelle de l’étiquette 6 avant tout partage. Un document partagé sans étiquette part sans marquage visuel.

Règle 2 : Avertir pour données sensibles en interne

| **Paramètre** | **Ce que vous devez configurer** |
| --- | --- |
| Nom de la règle | Avertir-Données-Sensibles-Interne |
| Condition 1 | Le contenu contient → AVS OU IBAN OU données médicales tapez AVS-Suisse et sélectionner : AVS-Suisse, confiance moyenne. Tapez IBAN et sélectionner : International Banking Account Number (IBAN), confiance élevée. Tapez Medical-RH-Suisse et sélectionner : Medical-RH-Suisse, confiance moyenne Nb d’instance 3 |
| Condition 2 | Partagé avec des personnes à l'intérieur de mon organisation |
| Action | Aucun blocage : ne cochez aucune action |
| Notifications utilisateur | Activez les conseils de stratégie par Message : Ce document contient des données sensibles. Vérifiez que le destinataire est autorisé. |

Cliquez sur Enregistrer.

10. Une fois les 2 règles créées (Surveiller-Partage-Externe, Avertir-Données-Sensibles-Interne), cliquez sur **Suivant**.

11. Page 7 : Mode de la stratégie → Mode : Exécuter la stratégie en mode simulation → **Suivant**.

12. Page 8 : Vérifiez et cliquez sur **Soumettre**.

La politique **DLP-Protection-Transmission-Externe** contient **2 règles** couvrant l'ensemble des vecteurs de risque du site : surveillance traçabilité partages externes (Règle 1), avertissement interne données sensibles (Règle 2).

> ⚠️ **ATTENTION : **Testez toujours en simulation 1 semaine avant d'activer. Regardez les résultats dans Activity Explorer. Si trop de faux positifs, augmentez le niveau de confiance dans les règles. Si après 1 semaine le résultat est correct activez la stratégie.

## 4.4 DLP-Protection-Copilot

Politique distincte : Protéger les documents étiquetés dans Copilot

**ℹ️ INFO : **La protection Copilot via DLP ne peut pas être ajoutée comme simple règle à la politique DLP-Protection-nLPD-demo. L’emplacement **Copilot (expériences Microsoft Copilot)** doit être configuré dans une **politique DLP dédiée**. C’est uniquement dans cette politique qu’apparaissent la condition *Étiquettes de sensibilité* et l’action *Restreindre l’accès dans Copilot*. 

Créez une nouvelle politique distincte en suivant ces étapes :

1. Purview → Protection contre la perte de données → Stratégies → + Créer une stratégie → Personnalisé → Personnalisé → **Suivant.**

2. Nom : DLP-Protection-Copilot | Description : Bloque l’accès Copilot aux documents confidentiels étiquetés. → **Suivant.**

3. Unités administratives : Répertoire complet → **Suivant.**

4. Emplacements : activez **uniquement** : ✅Microsoft 365 Copilot et Copilot Chat, Tous les utilisateurs. Laissez tous les autres emplacements désactivés. → **Suivant.**

5. Paramètres : Créer ou personnaliser des règles DLP avancées → **Suivant.**

6. Règles > + Créer une règle. Configurez la règle unique comme suit :

| **Paramètre** | **Ce que vous devez configurer** |
| --- | --- |
| Nom de la règle | Bloquer-Copilot-Confidentiel |
| Condition | Le contenu inclut → + Ajouter → Étiquettes de sensibilité → sélectionnez : 3b - Direction – Documents, 4 - RH-Confidentiel, 5 - Finances-Confidentiel, 6 - Transmission-Externe → **Ajouter**. (Cette condition n’est visible que si l’emplacement Copilot est activé dans la politique.) |
| Action | Restreindre Copilot au traitement du contenu →✅ Accès aux sources de connaissances |
| Rapports d’incident | Alerte admin : Activé │ Email : admin@demo.ch |

7. Cliquez sur Enregistrer. Puis Suivant → Mode simulation → Suivant → Soumettre.

Limites connues

**Limite 1 : SIT non supportés sur l’emplacement Copilot.**
L’emplacement Copilot (expériences Microsoft Copilot) accepte uniquement les étiquettes de confidentialité comme condition, les Types d’infos confidentielles (SIT) ne sont pas disponibles dans ce menu. L’option « Types d’infos confidentielles » n’apparaît pas dans les conditions de la règle lorsque l’emplacement Copilot est activé. Conséquence : un fichier non étiqueté contenant un numéro AVS ou un IBAN n’est pas protégé par cette politique ce qui renforce l’importance critique de l’auto-labelling (sections 6.1 et 6.2).

Limite 2 : Copilot standalone (copilot.microsoft.com) protection partielle.
La politique DLP-Protection-Copilot couvre les expériences Copilot intégrées à Microsoft 365 (Copilot dans Word, Excel, Teams, SharePoint). L'application Copilot standalone accessible via copilot.microsoft.com ou l'application Windows est hors périmètre de cette politique. La DLP-Protection-Endpoint (créée plus loin) assure une protection complémentaire sur les postes intégrés : l'upload de fichiers vers copilot.microsoft.com est bloqué via le groupe Sites web d'IA générative, et le copier-coller de contenu sensible est bloqué sur Edge natif via l'action Coller dans les navigateurs pris en charge. Ces deux vecteurs sont donc couverts. Cette protection ne s'applique pas sur Firefox et Chrome sans extension Purview installée, ni sur les postes non intégrés dans Purview. La saisie manuelle de contenu sensible dans Copilot standalone (frappe directe au clavier, sans copier-coller ni upload) reste le seul angle mort techniquement non blocable par la DLP-Protection-Endpoint, aucune opération de presse-papiers ou de fichier n'est interceptable dans ce cas. Pour une couverture complète de ce vecteur résiduel, une politique Shadow AI complémentaire (Web Content Filtering + MDCA) est recommandée.

**Limite 3 : Bandeau « Protection des données d’entreprise » ≠ protection Purview.**
Le bandeau affiché dans Copilot standalone indique une protection de session Entra/Intune (les données de la conversation ne sont pas partagées avec des tiers pour l’entraînement). Il ne signifie pas que Purview DLP inspecte ou bloque le contenu saisi. Ces deux mécanismes sont indépendants : ne pas les confondre en démonstration client.

---

[← Partie 3 : Auto-labelling (étiquetage automatique)](04-auto-labelling.html) | [Partie 5 : Audit et DSPM for AI (surveillance Copilot) →](06-audit-dspm-ai.html)
