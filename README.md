# Diagnostic Maturité Automatisation PME

Outil de diagnostic en ligne évaluant la maturité d'une PME en matière d'automatisation. Le répondant complète un questionnaire de 22 questions, découvre immédiatement son score et son profil, puis reçoit par email un rapport personnalisé : analyse par axe et trois recommandations priorisées générées par IA.

Démo : https://aymardbruyere1-byte.github.io/diagnostic-automatisation/

## Parcours du répondant

1. **Questionnaire** : 22 questions fermées réparties sur quatre axes (process, outils et données, culture et appétit, enjeu business). Une réponse par question, enchaînement automatique après sélection, retour arrière possible à tout moment.
2. **Score immédiat** : dès la dernière question répondue, le score global sur 100 et le profil de maturité s'affichent à l'écran. Le scoring est déterministe et calculé côté client, sans appel réseau.
3. **Contact** : nom, entreprise, email professionnel et consentement explicite. C'est la seule étape où une donnée personnelle est demandée.
4. **Restitution** : l'analyse détaillée des quatre axes et les trois recommandations personnalisées arrivent par email en quelques minutes.

Ce découpage résulte d'un arbitrage assumé : restituer le score avant la demande d'email fait perdre l'effet de rétention du résultat, mais donne au répondant une contrepartie immédiate et rend la demande de coordonnées légitime plutôt que transactionnelle. La valeur différenciante, l'analyse et les recommandations, reste délivrée par email.

## Architecture

```
Formulaire (GitHub Pages, HTML/JS statique)
        │  POST JSON
        ▼
Webhook Make ── filtre anti-spam
        │
        ├─▶ Journal d'audit (data store)
        ▼
API Claude (génération du rapport personnalisé)
        ▼
CRM Airtable (création de la fiche prospect)
        ▼
Routeur
        ├─▶ Email de restitution au prospect (Gmail)
        ├─▶ Alertes internes "lead chaud" (push iOS + email)
        └─▶ Alerte interne en cas d'échec de génération
```

Un second scénario, indépendant du premier, assure la supervision :

```
Déclencheur planifié (quotidien)
        ▼
Lecture du journal d'audit
        ▼
Filtre : soumissions entrées mais non terminées depuis plus d'une heure
        ▼
Email récapitulatif interne (uniquement s'il y a des anomalies)
```

### Composants

- **Formulaire** : page statique HTML/CSS/JS sans dépendance, hébergée sur GitHub Pages. Le scoring (score global sur 100, quatre scores d'axe, profil, qualification du lead) est calculé de manière déterministe côté client, avant envoi. Le score global et le profil sont restitués à l'écran en fin de questionnaire ; le détail par axe et les recommandations sont réservés à l'email. Le bouton de progression est masqué tant qu'aucune réponse n'est sélectionnée, plutôt que grisé, afin de ne pas suggérer une action indisponible.
- **Orchestration** : scénario Make déclenché par webhook, en temps réel.
- **Génération du rapport** : appel à l'API Claude (Anthropic) avec un prompt structuré. La sortie est un JSON strict (synthèse, trois recommandations, objet d'email), parsé avant tout envoi. L'IA rédige, elle ne calcule ni ne modifie aucun score.
- **CRM** : une fiche Airtable par diagnostic (scores, profil, statut, réponses brutes, rapport généré).
- **Restitution** : email HTML au prospect. Alertes internes immédiates lorsque le diagnostic répond aux critères de qualification "lead chaud".
- **Supervision** : scénario Make planifié, distinct du scénario principal, qui contrôle quotidiennement le journal d'audit.

## Robustesse

Le pipeline est conçu pour ne perdre aucune soumission :

- **Anti-spam** : clé de validation côté formulaire, filtrée à l'entrée du scénario. Protection contre les soumissions parasites sur le webhook (le dépôt étant public) ; ce n'est pas une authentification forte.
- **Échec de la génération IA** : le scénario se poursuit avec des valeurs de substitution. La fiche prospect est créée avec un statut dédié permettant une relance manuelle ; le prospect ne reçoit aucun email tant qu'un rapport valide n'a pas été produit ; une alerte interne est envoyée.
- **Échec d'envoi email** : nouvelles tentatives automatiques, puis conservation de l'exécution pour reprise manuelle.
- **Journal d'audit** : chaque soumission légitime est tracée à l'entrée du scénario (data store dédié), indépendamment du CRM. Un statut final est écrit en fin de traitement.
- **Contrôle de cohérence quotidien** : un scénario planifié lit ce journal chaque jour et signale par email toute soumission entrée mais jamais menée à son terme au-delà d'une heure. Sans ce contrôle, le journal ne serait qu'une donnée consultable à la demande ; c'est lui qui transforme la traçabilité en supervision effective. Aucun email n'est émis lorsque rien n'est à signaler.
- **Notifications de plateforme** : alertes email en cas d'erreur, d'avertissement ou de désactivation d'un scénario.

**Limite assumée** : ce contrôle ne se surveille pas lui-même. S'il cessait de s'exécuter, aucune alerte spécifique ne serait émise. Les notifications Make sur désactivation de scénario couvrent partiellement ce risque, sans le couvrir entièrement. Un contrôle externe serait la réponse complète ; il n'est pas justifié aux volumes actuels.

## Données personnelles (RGPD)

- **Données collectées** : nom, entreprise, email professionnel, réponses au questionnaire.
- **Finalité** : production et envoi du rapport de diagnostic ; qualification commerciale des demandes.
- **Base légale** : consentement (case à cocher explicite avant soumission).
- **Destinataires** : le responsable du traitement uniquement. Sous-traitants techniques : Make (orchestration, hébergement UE), Anthropic (génération du rapport), Airtable (stockage), Google (envoi des emails).
- **Conservation** : 3 ans après le dernier contact, puis suppression.
- **Droits** : accès, rectification, effacement, opposition. Toute demande : bruyere.builder@gmail.com
- La mention d'information complète figure en pied de page du formulaire.

## Stack

| Brique | Rôle |
|---|---|
| GitHub Pages | Hébergement du formulaire statique |
| Make | Deux scénarios : orchestration temps réel (webhook, gestion d'erreurs, journal d'audit) et supervision planifiée (contrôle quotidien du journal) |
| Claude API (Anthropic) | Génération du rapport personnalisé |
| Airtable | CRM |
| Gmail | Emails transactionnels et alertes |

## Auteur

Conçu et opéré par Bruyère · Conseil AMOA & automatisation IA.
