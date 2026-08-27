# Diagramme — Cycle de vie d'un module

## 1. Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────────┐
│                        src/modules/index.js                        │
│                                                                     │
│  const { appModules } = [                                          │
│    AutoModModule,                                                   │
│    XPLevelModule,                                                   │
│    DailyMessageModule,                                              │
│    ...                                                              │
│  ];                                                                 │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       src/index.js (bootstrap)                      │
│                                                                     │
│  const client = new Client(...);                                    │
│  const app = express();                                             │
│  moduleManager.init(client, app);                                   │
│  moduleManager.registerModules(appModules);                         │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
                ┌───────────────────────────────┐
                │       ModuleManager           │
                │       .registerModules()      │
                └───────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌──────────────┐        ┌──────────────┐        ┌──────────────┐
│ 1. Imports   │        │ 2. Providers │        │ 3. Events    │
│ (récursif)   │        │  (Container) │        │  (EventBus)  │
└──────────────┘        └──────────────┘        └──────────────┘
                                                       │
                                                       ▼
                                              ┌──────────────┐
                                              │ 4. Commands  │
                                              │  (deploy)    │
                                              └──────────────┘
                                                       │
                                                       ▼
                                              ┌──────────────┐
                                              │ 5. Cron      │
                                              │  (node-cron) │
                                              └──────────────┘
```

## 2. Détail des phases

### 2.1 Initialisation

```
Démarrage Node.js
   │
   ▼
src/index.js
   │
   ├── Crée le client Discord (login)
   ├── Crée l'app Express
   ├── Charge le config YAML + .env
   ├── Initialise la DB (Drizzle)
   │
   ├── moduleManager.init(client, app)
   │      │
   │      ├── eventBus.init(client)
   │      │      │
   │      │      └── Attache les listeners "cache BDD" au client
   │      │
   │      └── app.use(apiRouter)   ← Express
   │
   └── moduleManager.registerModules(appModules)
```

### 2.2 Chargement d'un module

Pour chaque `ModuleClass` dans `appModules` :

```
ModuleManager._loadModule(MyModule)
   │
   ├── Lecture de MyModule.__moduleMetadata
   │
   ├── 1. registerModules(metadata.imports)   ← récursif
   │
   ├── 2. Pour chaque Provider :
   │      container.register(Provider)
   │      instance = container.resolve(Provider)  ← DI auto via static inject
   │      _bindCronTasks(Provider, instance)
   │
   ├── 3. Pour chaque Controller :
   │      container.register(Controller)
   │      instance = container.resolve(Controller)
   │      _mountController(Controller, instance)
   │         │
   │         └── Pour chaque route : apiRouter[method](path, handler)
   │
   ├── 4. Pour chaque Event :
   │      container.register(Event)
   │      instance = container.resolve(Event)
   │      _bindEventHandlers(Event, instance)
   │         │
   │         └── Pour chaque @OnEvent : eventBus.subscribe(eventName, handler, options)
   │
   ├── 5. container.register(MyModule)
   │      instance = container.resolve(MyModule)
   │      _bindCronTasks(MyModule, instance)
   │
   └── this.modules.push({ name, instance, metadata })
```

### 2.3 Runtime — événement Discord

```
Client Discord → 'messageCreate'
   │
   ▼
EventBus._ensureDiscordListener('messageCreate')
   │
   ▼
EventBus.dispatch('messageCreate', message)
   │
   ├── Pour chaque subscriber (trié par priority DESC) :
   │
   │   ┌─► Vérifie options.configKey → skip si enabled=false
   │   │
   │   ├─► Vérifie options.ignoreBots (true par défaut) → skip si bot
   │   │
   │   ├─► Vérifie options.channelId → skip si autre salon
   │   │
   │   ├─► Vérifie options.filter(...args) → skip si false
   │   │
   │   └─► handler.apply(context, message)
   │            │
   │            └── try/catch → isole les erreurs
   │
   └── Continue avec le subscriber suivant
```

### 2.4 Runtime — slash command

```
Discord → INTERACTION_CREATE (type 2 = APPLICATION_COMMAND)
   │
   ▼
src/events/interactionCreate.js (legacy)
   │
   ▼
ModuleManager._dispatchCommand(interaction)
   │
   ├── Cherche la @Command correspondante (par name)
   │
   ├── Si @Command a un metadata.handlerName :
   │      instance = container.resolve(Module)
   │      instance[handlerName](interaction)
   │
   └── Si permission insuffisante → reply ephemeral "Vous n'avez pas la permission"
```

## 3. Codes de retour & erreurs

- **Container** : `throw new Error('[Container] Impossible de résoudre la dépendance : "<name>"')`
- **EventBus** : `console.error("❌ [EventBus] Erreur dans le gestionnaire pour "<eventName>":", err)`
- **API** : `res.status(500).json({ success: false, error: err.message })`
- **Command** : `await interaction.reply({ content: '❌ Erreur', ephemeral: true })`

## 4. Arrêt gracieux

```js
// src/index.js (handler SIGINT)
process.on('SIGINT', async () => {
  console.log('🛑 Arrêt en cours…');
  moduleManager.stopCronJobs();   // stoppe tous les cron
  await client.destroy();
  process.exit(0);
});
```

## 5. Hot-reload (futur)

Pour recharger un module sans redémarrer le bot :

```
moduleManager.reloadModule(MyModule)
   │
   ├── moduleManager.stopCronJobs()  ← pour les cron du module
   │
   ├── eventBus.unsubscribe(MyModule)  ← détache les listeners
   │
   ├── container.clear() pour les providers du module
   │
   ├── Détruire les routes Express du module
   │
   ├── require.cache : delete require.resolve(MyModule)
   │
   └── moduleManager.registerModules([newMyModule])
```

> **Statut** : non implémenté, prévu Phase post-MVP.
