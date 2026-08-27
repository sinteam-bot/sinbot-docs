# Documentation Chienne Bot

Bienvenue dans la documentation du bot Discord **Chienne**. Ce dossier regroupe l'ensemble des documents techniques : architecture, guides de développement, références API et plans d'évolution.

## Sommaire

### Architecture

- [`architecture/overview.md`](./architecture/overview.md) — Vue d'ensemble du projet, patterns utilisés (IoC, EventBus, ModuleManager), et conventions de code
- [`architecture/data-model.md`](./architecture/data-model.md) — Modèle de données Drizzle (SQLite / PostgreSQL), schémas, relations et migrations
- [`architecture/web-dashboard.md`](./architecture/web-dashboard.md) — Architecture du dashboard Nuxt, communication avec l'API Express, auth et time-to-live

### Features

- [`features/automod.md`](./features/automod.md) — Spécification de la feature **Modération automatique** (P1)
- [`features/xp-level.md`](./features/xp-level.md) — Système XP & niveaux enrichi (P2)
- [`features/tickets.md`](./features/tickets.md) — Système de tickets / support (P2)
- [`features/logs-stats.md`](./features/logs-stats.md) — Logs détaillés & dashboard de statistiques (P2)
- [`features/giveaways-polls.md`](./features/giveaways-polls.md) — Giveaways & sondages (P3)
- [`features/welcome-advanced.md`](./features/welcome-advanced.md) — Bienvenue avancée (cartes, autoroles) (P3)

### Plan d'intégration

- [`plan/feature-registry.md`](./plan/feature-registry.md) — **Phase 0** : registre de features, permissions, multi-guild ready
- [`plan/migration-yaml.md`](./plan/migration-yaml.md) — Migration progressive du YAML actuel vers le nouveau format `features.*`
- [`plan/roadmap.md`](./plan/roadmap.md) — Roadmap globale, estimation des charges, jalons et critères d'acceptation

### Diagrammes

- [`diagrams/module-lifecycle.md`](./diagrams/module-lifecycle.md) — Cycle de vie d'un module (register → bind events → ready)
- [`diagrams/event-bus.md`](./diagrams/event-bus.md) — Flux de l'EventBus, priorités, filtrage et isolation des erreurs

### Guides

- [`guides/create-a-feature.md`](./guides/create-a-feature.md) — Tutoriel pas-à-pas pour créer une feature (de zéro à déploiement)
- [`guides/testing.md`](./guides/testing.md) — Stratégie de tests (`node --test`), mocks, intégration
- [`guides/deployment.md`](./guides/deployment.md) — Déploiement (Docker, env, healthcheck, sauvegardes)

### Référence API

- [`api/rest.md`](./api/rest.md) — Endpoints HTTP exposés (`/api/features`, `/api/mod/*`, etc.)
- [`api/discord-commands.md`](./api/discord-commands.md) — Liste exhaustive des slash commands

---

## À propos du projet

**Chienne** est un bot Discord modulaire construit autour :

- **discord.js v14** pour l'API Discord
- **Express 5** pour le dashboard web et les webhooks
- **Drizzle ORM** (SQLite local, PostgreSQL en prod via `docker-compose`)
- **node-cron** pour le planificateur de tâches
- **OpenRouter** (multi-modèles avec fallback) pour la génération de contenu
- **Nuxt 3** pour le frontend (SSR / statique)

L'architecture suit un **style NestJS / Angular** via un système de décorateurs maison (`@Module`, `@Injectable`, `@OnEvent`, `@Command`, `@Cron`, etc.), un **conteneur IoC**, un **EventBus** centralisé et un **ModuleManager** qui orchestre l'enregistrement des providers, controllers, events et tâches planifiées.

## Démarrage rapide

```bash
# Installation
npm install

# Copier la configuration
cp config.example.yml config.yml
cp .env.example .env

# Lancer en mode développement
npm start

# Tests
npm test

# Build & déploiement UI
npm run build:ui
```

## Contribuer

1. Créer une branche feature depuis `main`
2. Suivre les conventions de l'architecture modulaire (cf. `guides/create-a-feature.md`)
3. Ajouter des tests pour toute nouvelle logique métier
4. Documenter la feature dans `docs/features/`
5. Ouvrir une PR avec description détaillée

---

_Dernière mise à jour : 2026-08-27_
