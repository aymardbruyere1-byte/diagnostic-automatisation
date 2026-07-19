[README.md](https://github.com/user-attachments/files/30170766/README.md)
# Diagnostic Maturité Automatisation — PME

Outil de diagnostic en ligne évaluant la maturité d'une PME en matière d'automatisation. Le répondant complète un questionnaire de 22 questions et reçoit par email un rapport personnalisé : score de maturité, profil, et trois recommandations priorisées générées par IA.

Démo : https://aymardbruyere1-byte.github.io/diagnostic-automatisation/

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

### Composants

- **Formulaire** : page statique HTML/CSS/JS sans dépendance, hébergée sur GitHub Pages. Le scoring (score global sur 100, quatre scores d'axe, profil) est calculé de manière déterministe côté client, avant envoi.
- **Orchestration** : scénario Make déclenché par webhook, en temps réel.
- **Génération du rapport** : appel à l'API Claude (Anthropic) avec un prompt structuré. La sortie est un JSON strict (synthèse, trois recommandations, objet d'email), parsé avant tout envoi. L'IA rédige, elle ne calcule ni ne modifie aucun score.
- **CRM** : une fiche Airtable par diagnostic (scores, profil, statut, réponses brutes, rapport généré).
- **Restitution** : email HTML au prospect. Alertes internes immédiates lorsque le diagnostic répond aux critères de qualification "lead chaud".

## Robustesse

Le pipeline est conçu pour ne perdre aucune soumission :

- **Anti-spam** : clé de validation côté formulaire, filtrée à l'entrée du scénario. Protection contre les soumissions parasites sur le webhook (le dépôt étant public) ; ce n'est pas une authentification forte.
- **Échec de la génération IA** : le scénario se poursuit avec des valeurs de substitution. La fiche prospect est créée avec un statut dédié permettant une relance manuelle ; le prospect ne reçoit aucun email tant qu'un rapport valide n'a pas été produit ; une alerte interne est envoyée.
- **Échec d'envoi email** : nouvelles tentatives automatiques, puis conservation de l'exécution pour reprise manuelle.
- **Journal d'audit** : chaque soumission légitime est tracée à l'entrée du scénario (data store dédié), indépendamment du CRM. Un statut final est écrit en fin de traitement, ce qui rend détectable toute exécution interrompue.
- **Supervision** : notifications email en cas d'erreur, d'avertissement ou de désactivation du scénario.

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
| Make | Orchestration temps réel, gestion d'erreurs, journal d'audit |
| Claude API (Anthropic) | Génération du rapport personnalisé |
| Airtable | CRM |
| Gmail | Emails transactionnels et alertes |

## Auteur

Conçu et opéré par Bruyère — Conseil AMOA & automatisation IA.
