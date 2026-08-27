# Architecture — Vue d'ensemble

> **Statut** : validé & implémenté (cœur), en cours d'évolution pour le multi-guild et le registre de features.

## 1. Stack technique

| Couche | Technologie | Rôle |
|---|---|---|
| Runtime | Node.js 20+ | Exécution JS |
| Framework Discord | `discord.js` v14 | Client, slash commands, interactions, voice |
| Framework HTTP | `Express` 5 | API REST, webhooks, servir le dashboard |
| Frontend | `Nuxt` 3 | Dashboard utilisateur (SSR / statique) |
| ORM | `Drizzle` 0.45 | Schéma type-safe, migrations |
| Base de données | SQLite (dev) / PostgreSQL (prod) | Persistance |
| Planificateur | `node-cron` 4.6 | Tâches récurrentes (rappels, daily message…) |
| IA | `openai` SDK → OpenRouter | Génération de texte multi-modèles avec retry |
| Config | `js-yaml` | YAML statique pour la config par défaut |

## 2. Patterns architecturaux

### 2.1 Décorateurs (style NestJS / Angular)

Un système de décorateurs est implémenté dans `src/core/decorators.js` :

```js
const { Module, Injectable, Controller, Get, Post, OnEvent, Command, Cron } = require('../core');

@Module({
  imports: [OtherModule],
  providers: [MyService, MyRepository],
  controllers: [MyController],
  events: [MyEvent],
  commands: [MyCommand]
})
class MyModule {}

@Injectable()
class MyService {
  constructor(dep1, dep2) { /* DI via Container */ }
}

@Controller('/api/foo')
class MyController {
  @Get('/')
  list(req, res) { return { ok: true }; }
}

@OnEvent('messageCreate', { ignoreBots: true, priority: 10 })
class MyEventListener {
  async handle(message) { /* ... */ }
}

@Cron('0 9 * * *', { timezone: 'Europe/Paris', configKey: 'features.daily' })
class MyCron {
  async run(client) { /* ... */ }
}
```

> Les décorateurs sont **compatibles Node.js natif** (pas de TypeScript requis). Les métadonnées sont stockées sur la classe (`__moduleMetadata`, `__routes`, `__eventHandlers`, etc.).

### 2.2 Conteneur IoC (`src/core/container.js`)

- **Singleton** par défaut
- Résolution récursive des dépendances déclarées via `static inject = [Dependency1, Dependency2]`
- Permet de mocker les services en test

```js
class OrderService {
  static inject = [PaymentService, LoggerService];
  constructor(payment, logger) {
    this.payment = payment;
    this.logger = logger;
  }
}
container.register(OrderService);
const service = container.resolve(OrderService);
```

### 2.3 EventBus (`src/core/event-bus.js`)

Le bus centralise **toutes** les souscriptions aux événements Discord. Chaque module déclare ses listeners via `@OnEvent`, le `ModuleManager` les enregistre au démarrage.

**Avantages :**

- **Un seul listener Discord.js** par événement (perf)
- **Filtrage déclaratif** : `ignoreBots`, `channelId`, `configKey`, `filter`, `priority`
- **Isolation des erreurs** : une exception dans un handler n'arrête pas les autres
- **Désactivation à chaud** : si la config du module passe à `enabled: false`, le handler est skippé sans désinscription

```js
@OnEvent('messageCreate', {
  configKey: 'features.automod',  // désactivé si features.automod.enabled = false
  ignoreBots: true,
  priority: 100
})
class AutoModListener {
  async handle(message) {
    // ...
  }
}
```

### 2.4 ModuleManager (`src/core/module-manager.js`)

Orchestrateur unique :

1. **Charge les imports** (récursivement)
2. **Instancie les providers** via le Container
3. **Monte les controllers** sur l'Express Router
4. **Branche les event handlers** sur l'EventBus
5. **Planifie les cron tasks** via `node-cron`

Tous les modules sont centralisés dans `src/modules/index.js` :

```js
const { appModules } = require('./modules');
const { moduleManager } = require('./core');
moduleManager.init(client, app);
moduleManager.registerModules(appModules);
```

## 3. Arborescence du projet

```
discord-bot_Chienne/
├── src/
│   ├── commands/                # Slash commands legacy (pré-modules)
│   ├── core/                    # Cœur architectural (Container, EventBus, ModuleManager, decorators)
│   ├── modules/                 # Modules métier (feature_*, game_*, service_*, security_*, notifier_*)
│   │   ├── feature_daily-message/
│   │   ├── feature_welcome/
│   │   ├── feature_xp-level/
│   │   ├── game_road-to-infinite/
│   │   ├── game_count-down/
│   │   ├── security_captcha/
│   │   ├── security_question/
│   │   ├── service_bump-reminder/
│   │   ├── notifier_startup/
│   │   ├── feature_automod/    # 🆕 Phase 1
│   │   ├── feature_tickets/    # 🆕 Phase 3
│   │   ├── feature_logs/       # 🆕 Phase 4
│   │   ├── feature_giveaways/  # 🆕 Phase 5
│   │   └── feature_polls/      # 🆕 Phase 5
│   ├── services/                # Services transverses (DiscordCacheService, imageProxyService…)
│   ├── config/                  # Chargement YAML + .env
│   ├── events/                  # Listeners legacy (synchronisation cache BDD)
│   ├── database/                # Schémas Drizzle, migrations
│   ├── db/                      # Connexion DB (Drizzle + driver natif)
│   ├── web/                     # Express, middlewares, routes API
│   ├── utils/                   # Helpers (loggers, formatters, validation)
│   ├── index.js                 # Point d'entrée
│   └── database.js              # Bootstrap base
├── frontend/                    # Dashboard Nuxt 3
│   ├── pages/
│   ├── components/
│   ├── composables/
│   └── stores/                  # Pinia
├── data/                        # SQLite (gitignored)
├── logs/
├── tests/                       # node --test
├── config.example.yml
├── docker-compose.yml
├── Dockerfile
└── package.json
```

## 4. Cycle de vie d'un module

```
[Definition]          [Bootstrap]              [Runtime]
                                                             
  @Module({})  ──►  ModuleManager   ──►   Container.resolve()
   decorators        .registerModules       (singleton)
   (metadata)             │
                          ├──► Container.register(providers)
                          ├──► Express.use(controllers)
                          ├──► EventBus.subscribe(events)
                          └──► cron.schedule(cron tasks)
```

Voir [`diagrams/module-lifecycle.md`](../diagrams/module-lifecycle.md) pour le détail.

## 5. Conventions de code

- **Style** : `lowercase-camelcase` pour les variables / fonctions, `PascalCase` pour les classes
- **Modules** : un module = un dossier sous `src/modules/`, préfixé par son type (`feature_`, `game_`, `service_`, `security_`, `notifier_`)
- **Configuration** : un bloc YAML par module, validé au démarrage
- **Logs** : préfixés par emoji + scope (`📦 [ModuleManager]`, `🎧 [EventBus]`, `❌ [API Error]`)
- **Erreurs** : remonter via `throw new Error('contexte: détail')` ou via le bus d'erreurs (cf. `src/core/event-bus.js`)
- **Tests** : `node --test`, fichiers `*.test.js` à côté du code testé ou dans `/tests/`
- **Pas de commentaires JSDoc pour les évidences**, commentaires uniquement quand la logique n'est pas triviale (cf. consigne opencode)

## 6. Sécurité

- **API key + IP allowlist** (cf. `config.example.yml > web.auth`)
- **Protection des statiques** (optionnelle)
- **Validation des entrées** : à chaque endpoint API et chaque slash command
- **Aucune clé / token en clair dans le repo** : `.env` + `.gitignore` strict
- **Permissions Discord** : `allowed_roles` / `allowed_users` / `allowed_channels` par commande, ET par feature (cf. `plan/feature-registry.md`)

## 7. Prochaines évolutions architecturales

- **Multi-guild** : scoping systématique par `guildId` dans toutes les requêtes DB
- **FeatureRegistry** : registre déclaratif des features activables/désactivables par guild (cf. `plan/feature-registry.md`)
- **Hot-reload** : recharger un module sans redémarrer le bot (Phase future)
- **WebSocket** : push temps réel des logs et stats vers le dashboard

Voir [`../plan/roadmap.md`](../plan/roadmap.md) pour la roadmap complète.
