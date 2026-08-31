# Plan: Réimplémentation Rust du Discord Bot

> **Date** : 2026-08-31
> **Objectif** : Réimplémenter le bot Discord en Rust avec architecture multi-agents
> **Dossier cible** : `./backend/`

## 1. Architecture

```
backend/
├── Cargo.toml              # Workspace manifest
├── crates/
│   ├── core/               # Types partagés, traits, erreurs
│   ├── config/             # Configuration (YAML/env)
│   ├── db/                 # Database (PostgreSQL via sqlx)
│   ├── discord/             # Client Discord (twilight)
│   ├── web/                # Serveur web dashboard (axum)
│   ├── scheduler/          # Tâches planifiées (tokio-cron)
│   └── features/           # Modules fonctionnels
│       ├── automod/
│       ├── tickets/
│       ├── xp/
│       ├── economy/
│       ├── welcome/
│       ├── reaction-roles/
│       └── ...
```

## 2. Stack technique

| Composant | Library | Raison |
|---|---|---|
| Discord API | `twilight` | Moderne, performant, bien maintenu |
| Async runtime | `tokio` | Standard Rust async |
| Database | `sqlx` + `postgres` | Compile-time checked queries |
| Web server | `axum` | Moderne, ergonomique, tokio-native |
| Serialization | `serde` + `serde_json` | Standard Rust |
| Logging | `tracing` + `tracing-subscriber` | Structured logging |
| Configuration | `config` + `dotenvy` | Multi-source config |
| Scheduling | `tokio-cron-scheduler` | Cron jobs async |
| Validation | `validator` | Input validation |
| Error handling | `thiserror` + `anyhow` | Ergonomic errors |

## 3. Phases d'implémentation

### Phase 0 — Setup projet (1-2h)
- [ ] Cargo workspace
- [ ] Structure dossiers
- [ ] Dependencies communes
- [ ] CI/CD basique

### Phase 1 — Core infrastructure (2-3h)
- [ ] Types partagés (Guild, User, Error)
- [ ] Configuration loader
- [ ] Database connection pool
- [ ] Discord client wrapper
- [ ] Web server skeleton
- [ ] Logging setup

### Phase 2 — Features core (3-4h)
- [ ] AutoMod (filtres + sanctions)
- [ ] Welcome (messages + auto-rôles)
- [ ] XP/Levels (message + voice XP)
- [ ] Tickets (création + transcripts)

### Phase 3 — Features avancées (3-4h)
- [ ] Économie (daily, work, shop)
- [ ] Reaction Roles
- [ ] Logs & Stats
- [ ] Giveaways & Polls

### Phase 4 — Web Dashboard (2-3h)
- [ ] API REST complète
- [ ] Pages de configuration
- [ ] Feature toggles

### Phase 5 — Tests & Docs (2h)
- [ ] Tests unitaires
- [ ] Tests d'intégration
- [ ] Documentation

## 4. Stratégie multi-agents

| Agent | Rôle | Spécialité |
|---|---|---|
| Agent 1 | Core & Infra | Architecture, DB, Config |
| Agent 2 | Discord Gateway | Events, Commands, Interactions |
| Agent 3 | Web & API | REST, Dashboard, Auth |
| Agent 4 | Features | Automod, XP, Tickets, Économie |

## 5. Commandes Discord à implémenter

```
/mod warn|mute|kick|ban|history|clear
/rank|leaderboard|xp-add|xp-remove
/ticket open|close|claim|transcript
/giveaway start|end|reroll
/poll create|end
/confess (si activé)
/remind
/afk
/balance|daily|work|pay|shop
/auto-thread
/starboard
```

## 6. API Endpoints

```
GET    /api/health
GET    /api/guilds/:id/features
PATCH  /api/guilds/:id/features
GET    /api/guilds/:id/stats
GET    /api/guilds/:id/xp/leaderboard
POST   /api/guilds/:id/xp/:userId/grant
GET    /api/guilds/:id/tickets
POST   /api/guilds/:id/tickets
GET    /api/guilds/:id/economy/config
PATCH  /api/guilds/:id/economy/config
GET    /api/guilds/:id/automod/config
PATCH  /api/guilds/:id/automod/config
```

## 7. Base de données (tables principales)

```sql
guild_settings       — Configuration par guild
user_xp              — XP et niveaux
user_economy         — Solde et transactions
tickets              — Tickets et transcripts
user_warnings        — Avertissements
mod_logs             — Logs de modération
reaction_roles       — Configuration rôles-réactions
giveaways            — Concours
polls                — Sondages
```
