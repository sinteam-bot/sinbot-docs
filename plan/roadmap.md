# Roadmap — Intégration fonctionnalités type Draftbot

> **Date de création** : 2026-08-27
> **Périmètre** : fonctionnalités comparables à Draftbot (modération, engagement, support, statistiques)
> **Cible** : MVP en 6-10 jours, suite complète en 14-22 jours

## 1. Vision

Faire de **Chienne** un bot Discord modulaire comparable à Draftbot, avec :

- Un **registre de features** activables par serveur (multi-guild ready)
- Un **dashboard web** pour tout piloter sans redémarrer
- Des **permissions fines** par rôle Discord
- Une **rétrocompatibilité totale** avec la configuration existante

## 2. Décisions cadres (validées)

| Sujet | Décision |
|---|---|
| Priorité 1 | **Modération auto (automod)** |
| Architecture | **Multi-guild ready** dès le départ |
| Pilotage | **Dashboard Nuxt** (API key + IP allowlist) |
| Permissions | **Par rôle Discord**, par feature |
| Stack | **Aucune nouvelle dépendance majeure** (sauf `joi` pour la validation et `canvas`/`@napi-rs/canvas` pour la welcome avancée) |
| Rétrocompat | **YAML legacy toujours lu en fallback** |

## 3. Phases

### Phase 0 — Fondations (1-2 jours)

**Livrables** :
- Tables `guild_settings`, `feature_flags`
- Module `core/feature-registry`
- API REST `/api/features/*`
- Page `/features` (toggles on/off)
- Rétrocompatibilité YAML (cf. [`migration-yaml.md`](./migration-yaml.md))

**Détail** : [`feature-registry.md`](./feature-registry.md)

### Phase 1 — Automod (3-5 jours)

**Livrables** :
- Feature `feature_automod/` complète
- Règles : anti-spam, bad-words, anti-raid, anti-invite, anti-link, mass-mention, anti-caps
- Sanctions progressives (warn → mute → kick → ban)
- Tables `user_warnings`, `user_sanctions`, `mod_logs`
- Slash commands : `/mod warn|mute|kick|ban|history|clear|unban`
- Page `/features/automod` + dashboard modération
- Tests unitaires

**Détail** : [`../features/automod.md`](../features/automod.md)

### Phase 2 — XP enrichi (2-3 jours)

**Livrables** :
- Messages de level-up (configurables)
- Rôles auto par palier
- Leaderboard paginé
- Page `/leaderboard`
- Slash : `/rank`, `/leaderboard`, `/xp-set`, `/xp-add`, `/xp-reset`

**Détail** : [`../features/xp-level.md`](../features/xp-level.md)

### Phase 3 — Tickets (3-4 jours)

**Livrables** :
- Bouton persistant "Ouvrir un ticket"
- Catégories configurables
- Claim / unclaim / close
- Transcript auto en BDD
- Slash commands complètes
- Page `/tickets` (liste filtrable)

**Détail** : [`../features/tickets.md`](../features/tickets.md)

### Phase 4 — Logs & Stats (2-3 jours)

**Livrables** :
- 18 événements Discord écoutés
- Channels de logs paramétrables par type
- WebSocket `/ws/logs` pour le live
- Page `/logs` (filtres, recherche)
- Dashboard `/stats` (KPIs)

**Détail** : [`../features/logs-stats.md`](../features/logs-stats.md)

### Phase 5 — Giveaways & Polls (2-3 jours)

**Livrables** :
- `/giveaway start|end|reroll|list|cancel`
- Tirage CSPRNG + cron
- `/poll create|end|delete|list`
- Vote multi-choices, résultats live

**Détail** : [`../features/giveaways-polls.md`](../features/giveaways-polls.md)

### Phase 6 — Bienvenue avancée (1-2 jours)

**Livrables** :
- Carte d'image (canvas)
- Auto-rôles multiples
- Compteur de boosts
- Milestones

**Détail** : [`../features/welcome-advanced.md`](../features/welcome-advanced.md)

## 4. Tableau récapitulatif

| # | Phase | Effort | Cumul | Dépendances |
|---|---|---|---|---|
| 0 | Fondations (FeatureRegistry) | 1-2 j | 1-2 j | — |
| 1 | Automod | 3-5 j | 4-7 j | Phase 0 |
| 2 | XP enrichi | 2-3 j | 6-10 j | Phase 0 |
| 3 | Tickets | 3-4 j | 9-14 j | Phase 0 |
| 4 | Logs & Stats | 2-3 j | 11-17 j | Phase 0 |
| 5 | Giveaways & Polls | 2-3 j | 13-20 j | Phase 0 |
| 6 | Bienvenue avancée | 1-2 j | 14-22 j | Phase 0 |

## 5. MVP recommandé

Pour avoir un **produit utilisable** rapidement :

- Phase 0 (obligatoire)
- Phase 1 (modération : la plus demandée)
- Phase 2 (XP : déjà partiellement en place)

**Effort MVP** : 6-10 jours

## 6. Critères transversaux (qualité)

Toutes les phases doivent respecter :

- ✅ **Tests** : `node --test` (unitaire + intégration légère)
- ✅ **Documentation** : page dédiée dans `docs/features/`
- ✅ **Rétrocompat** : aucune régression sur les features existantes
- ✅ **Sécurité** : permissions vérifiées à chaque entrée (commande, API)
- ✅ **Performance** : pas d'appel DB synchrone dans une boucle d'event
- ✅ **Logs** : préfixés par emoji + scope, exploitables en debug
- ✅ **i18n-ready** : tous les textes utilisateur externalisés dans `config/messages.js`

## 7. Risques globaux

| Risque | Impact | Mitigation |
|---|---|---|
| Délais de revue / validation des PRs | Retard cumulé | Découper en PRs < 500 lignes |
| Discord rate limits sur les nouvelles features | Messages perdus | Cache + retry exponentiel déjà en place (OpenRouter) |
| Multi-guild : fuite de données | Critique | Audit systématique des requêtes DB, lint custom |
| Conflits avec les features existantes (counter, countdown) | Régression | Tests d'intégration avant chaque merge |
| Charge dashboard (WS) | Coût serveur | Throttle des events, batch côté serveur |

## 8. Hors périmètre (à discuter plus tard)

- Webhooks sortants (notifications externes : Slack, Discord webhook custom)
- Système de musique / vocal
- Intégration Twitch / YouTube alerts
- Système économique complet (banque, jobs, shop)
- IA conversationnelle par channel
- Web mobile / app native

## 9. Validation

Cette roadmap est validée sur les hypothèses suivantes :

- Le bot reste **mono-guild** pendant le développement (un seul `DISCORD_GUILD_ID`)
- L'infra actuelle (Docker, Express, Nuxt) **suffit** pour le dashboard
- Les utilisateurs finaux (staff du serveur) **acceptent** de saisir une clé API pour accéder au dashboard
- Aucune feature existante (counter, captcha, welcome…) n'est **cassée** par la migration

## 10. Prochaines étapes

1. **Découper** la Phase 0 en tickets Git (`/issues` ou backlog interne)
2. **Designer** la migration `feature_flags` ↔ `config.yml` (cf. [`migration-yaml.md`](./migration-yaml.md))
3. **Commencer** par la Phase 0 — créer `feature_registry` + DB schema + API minimale
4. **Itérer** : tester avec une feature existante (ex: `xp`) avant d'attaquer l'automod
