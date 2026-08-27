# Guide — Créer une feature (de zéro à déploiement)

> Ce guide explique comment ajouter une nouvelle feature en suivant les conventions du projet.

## 1. Pré-requis

- Avoir lu [`../architecture/overview.md`](../architecture/overview.md)
- Avoir fait un tour dans `src/modules/` pour comprendre la structure existante
- Avoir fait un tour dans `src/core/` (decorators, event-bus, container)

## 2. Étape 1 — Définir le besoin

Répondre à ces questions avant de coder :

- Quel est le **nom** de la feature ? (kebab-case)
- À quelle **catégorie** appartient-elle ? (`feature_`, `game_`, `service_`, `security_`, `notifier_`)
- Quels **événements Discord** sont nécessaires ?
- Quelles **slash commands** ?
- Quels **endpoints REST** pour le dashboard ?
- Quelle **config YAML** est requise ?

Exemple : je veux créer une feature de **rappels personnalisés** (`feature_reminders`).

## 3. Étape 2 — Créer l'arborescence

```bash
mkdir -p src/modules/feature_reminders/{config,db,services,events,commands,controllers,tests}
```

Structure type :

```
src/modules/feature_reminders/
├── reminders.module.js
├── config/
│   ├── schema.js
│   └── defaults.js
├── db/
│   └── schema.js
├── services/
│   └── reminders.service.js
├── events/
│   └── interaction-create.listener.js
├── commands/
│   ├── reminder-add.command.js
│   ├── reminder-list.command.js
│   └── reminder-remove.command.js
├── controllers/
│   └── reminders.controller.js
└── tests/
    └── reminders.test.js
```

## 4. Étape 3 — Écrire les defaults et le schéma de validation

`config/defaults.js` :

```js
module.exports = {
  enabled: false,
  allowed_roles: [],
  max_per_user: 10,
  default_color: '#5865F2'
};
```

`config/schema.js` (validation joi ou zod) :

```js
const joi = require('joi');

module.exports = joi.object({
  enabled: joi.boolean().default(false),
  allowed_roles: joi.array().items(joi.string()).default([]),
  max_per_user: joi.number().integer().min(1).max(50).default(10),
  default_color: joi.string().pattern(/^#[0-9A-F]{6}$/i).default('#5865F2')
});
```

## 5. Étape 4 — Le service métier

`services/reminders.service.js` :

```js
const { featureRegistry } = require('../../core/feature-registry');
const { Injectable } = require('../../core');

@Injectable()
class RemindersService {
  static inject = [];

  async create(guildId, userId, message, remindAt) {
    // Validation
    const state = await featureRegistry.get(guildId, 'reminders');
    if (!state.enabled) throw new Error('Feature désactivée');

    // Logique métier
    // ...

    return { id: '...', message, remindAt };
  }

  async list(guildId, userId) {
    // ...
  }

  async remove(guildId, userId, id) {
    // ...
  }
}

module.exports = { RemindersService };
```

## 6. Étape 5 — La slash command

`commands/reminder-add.command.js` :

```js
const { Command, getConfig } = require('../../core');
const { SlashCommandBuilder } = require('discord.js');

class ReminderAddCommand {
  static __commandBuilder = new SlashCommandBuilder()
    .setName('reminder-add')
    .setDescription('Créer un rappel')
    .addStringOption(opt => opt.setName('message').setDescription('Message').setRequired(true))
    .addStringOption(opt => opt.setName('when').setDescription('Quand (ex: 2h, 1j)').setRequired(true));

  static inject = [RemindersService];

  constructor(service) {
    this.service = service;
  }

  async execute(interaction) {
    // Vérifier la permission
    const canUse = await featureRegistry.canUse(
      interaction.guildId,
      interaction.user.id,
      'reminders'
    );
    if (!canUse.allowed) {
      return interaction.reply({ content: '❌ Feature désactivée ou permissions insuffisantes', ephemeral: true });
    }

    // Logique
    const message = interaction.options.getString('message');
    const when = interaction.options.getString('when');
    const remindAt = parseDuration(when);
    if (!remindAt) return interaction.reply({ content: '❌ Durée invalide', ephemeral: true });

    const reminder = await this.service.create(interaction.guildId, interaction.user.id, message, remindAt);
    await interaction.reply({ content: `✅ Rappel créé pour <t:${Math.floor(remindAt/1000)}:R>`, ephemeral: true });
  }
}

module.exports = { ReminderAddCommand };
```

## 7. Étape 6 — Le module

`reminders.module.js` :

```js
const { Module } = require('../../core');
const { RemindersService } = require('./services/reminders.service');
const { ReminderAddCommand } = require('./commands/reminder-add.command');
const { ReminderListCommand } = require('./commands/reminder-list.command');
const { ReminderRemoveCommand } = require('./commands/reminder-remove.command');
const { RemindersController } = require('./controllers/reminders.controller');
const { defaults } = require('./config/defaults');
const { schema } = require('./config/schema');
const { featureRegistry } = require('../../core/feature-registry');

featureRegistry.define('reminders', {
  defaults,
  configSchema: schema,
  onEnable: async (guildId) => console.log(`[reminders] enabled on ${guildId}`),
  onDisable: async (guildId) => console.log(`[reminders] disabled on ${guildId}`)
});

@Module({
  providers: [RemindersService],
  controllers: [RemindersController],
  commands: [ReminderAddCommand, ReminderListCommand, ReminderRemoveCommand]
})
class RemindersModule {}

module.exports = { RemindersModule };
```

## 8. Étape 7 — Enregistrer dans `src/modules/index.js`

```js
const { RemindersModule } = require('./feature_reminders/reminders.module');

const appModules = [
  // ... existants
  RemindersModule
];
```

## 9. Étape 8 — Ajouter au YAML

```yaml
# config.example.yml
features:
  reminders:
    enabled: false
    allowed_roles: []
    max_per_user: 10
    default_color: "#5865F2"
```

## 10. Étape 9 — Tests

`tests/reminders.test.js` :

```js
const { test } = require('node:test');
const assert = require('node:assert');
const { RemindersService } = require('../src/modules/feature_reminders/services/reminders.service');

test('RemindersService: create retourne un id', async () => {
  const service = new RemindersService();
  const r = await service.create('guild1', 'user1', 'test', Date.now() + 60_000);
  assert.ok(r.id);
  assert.strictEqual(r.message, 'test');
});
```

## 11. Étape 10 — Déployer

```bash
# 1. Régénérer la config
cp config.example.yml config.yml
# → éditer pour activer la feature

# 2. Re-déployer les slash commands
npm run deploy

# 3. Restart
npm start
```

## 12. Étape 11 — Documenter

Créer `docs/features/reminders.md` avec :

- Cas d'usage
- Configuration
- Commandes
- API REST
- Architecture
- Critères d'acceptation

## 13. Checklist finale

- [ ] Module enregistré dans `appModules`
- [ ] `featureRegistry.define()` appelé au chargement
- [ ] Defaults + schéma de validation
- [ ] Slash commands testées manuellement
- [ ] Permissions vérifiées à chaque entrée
- [ ] Tests unitaires (`node --test`)
- [ ] Config YAML documentée
- [ ] Page UI Nuxt (si applicable)
- [ ] Documentation `docs/features/`

## 14. Bonnes pratiques

- **Nommage** : `feature_<name>/` pour les features, `game_<name>/` pour les jeux, etc.
- **Décorateurs** : utiliser `@Injectable`, `@Module`, `@OnEvent`, `@Command` systématiquement
- **DI** : déclarer `static inject = [Dependency1, Dependency2]`
- **Erreurs** : `throw new Error('contexte: détail')`, isoler via try/catch dans les services critiques
- **Logs** : préfixer par emoji + scope (ex: `⏰ [reminders] Rappel créé`)
- **Async** : tous les handlers Discord sont `async`, tolérer les rejets non gérés
- **Pas de secrets** dans le code : utiliser `.env` ou `config.yml`
- **Pas de commentaires évidents** (cf. consigne opencode) — commenter uniquement la logique métier non triviale
