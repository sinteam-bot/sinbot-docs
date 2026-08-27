# Feature : Modération automatique (AutoMod)

> **Phase** : 1 — **Priorité** : 🔴 Critique — **Statut** : à implémenter

## 1. Objectif

Fournir un système de modération automatique comparable à Draftbot, configurable par serveur, accessible depuis le dashboard web et pilotable par slash commands.

## 2. Cas d'usage

| Règle | Comportement |
|---|---|
| **Anti-spam** | Bloque les messages trop rapprochés ou trop répétés |
| **Bad-words** | Supprime les messages contenant des mots interdits (regex) |
| **Anti-raid** | Détecte une vague de joins et lock les salons |
| **Anti-invite** | Supprime les liens d'invitation Discord (sauf whitelist) |
| **Anti-link** | Supprime les liens (sauf whitelist de domaines) |
| **Mass mention** | Bloque les messages mentionnant trop de personnes |
| **Anti-caps** | Supprime les messages en majuscules excessives |
| **Sanctions progressives** | warn → mute → kick → ban selon le nombre d'infractions |

## 3. Architecture

### 3.1 Arborescence

```
src/modules/feature_automod/
├── index.js                          # AutoModModule (export principal)
├── automod.module.js
├── config/
│   ├── schema.js                     # validation YAML
│   ├── defaults.js
│   └── messages.js                   # templates (i18n-ready)
├── db/
│   └── schema.js                     # user_warnings, user_sanctions, mod_logs (Drizzle)
├── events/
│   ├── message-create.listener.js    # règle principale
│   ├── message-update.listener.js    # bad-words sur edit
│   └── guild-member-add.listener.js  # anti-raid
├── services/
│   ├── spam-detector.service.js
│   ├── raid-detector.service.js
│   ├── bad-words.service.js
│   ├── sanctions.service.js          # logique d'escalade
│   ├── automod-engine.service.js     # orchestrateur
│   └── mod-log.service.js
├── commands/
│   ├── mod-warn.command.js
│   ├── mod-mute.command.js
│   ├── mod-kick.command.js
│   ├── mod-ban.command.js
│   ├── mod-history.command.js
│   ├── mod-clear.command.js
│   ├── mod-unban.command.js
│   └── automod-config.command.js
├── controllers/
│   └── automod.controller.js         # REST pour le dashboard
├── ui/                               # composants Vue (référencés)
│   └── README.md
└── tests/
    ├── spam-detector.test.js
    ├── raid-detector.test.js
    ├── sanctions.test.js
    └── bad-words.test.js
```

### 3.2 Enregistrement

```js
// src/modules/feature_automod/automod.module.js
const { Module } = require('../../core');
const { SpamDetector } = require('./services/spam-detector.service');
const { RaidDetector } = require('./services/raid-detector.service');
const { Sanctions } = require('./services/sanctions.service');
const { ModLog } = require('./services/mod-log.service');
const { AutomodEngine } = require('./services/automod-engine.service');

const { MessageCreateListener } = require('./events/message-create.listener');
const { MessageUpdateListener } = require('./events/message-update.listener');
const { GuildMemberAddListener } = require('./events/guild-member-add.listener');

const { ModWarnCommand } = require('./commands/mod-warn.command');
const { ModMuteCommand } = require('./commands/mod-mute.command');
const { ModKickCommand } = require('./commands/mod-kick.command');
const { ModBanCommand } = require('./commands/mod-ban.command');
const { ModHistoryCommand } = require('./commands/mod-history.command');
const { ModClearCommand } = require('./commands/mod-clear.command');
const { ModUnbanCommand } = require('./commands/mod-unban.command');
const { AutomodConfigCommand } = require('./commands/automod-config.command');

const { AutomodController } = require('./controllers/automod.controller');

@Module({
  providers: [SpamDetector, RaidDetector, Sanctions, ModLog, AutomodEngine],
  controllers: [AutomodController],
  events: [MessageCreateListener, MessageUpdateListener, GuildMemberAddListener],
  commands: [
    ModWarnCommand, ModMuteCommand, ModKickCommand, ModBanCommand,
    ModHistoryCommand, ModClearCommand, ModUnbanCommand, AutomodConfigCommand
  ]
})
class AutoModModule {}

module.exports = { AutoModModule };
```

```js
// src/modules/index.js (ajout)
const { AutoModModule } = require('./feature_automod/automod.module');
appModules.push(AutoModModule);
```

## 4. Configuration

### 4.1 Schéma YAML (extrait de `config.example.yml`)

```yaml
features:
  automod:
    enabled: false
    allowed_roles: ["MODERATOR_ROLE_ID"]
    spam:
      enabled: true
      max_messages: 5
      window_seconds: 5
      max_mentions: 3
      mentions_window_seconds: 10
      action: "warn"        # warn | mute | kick
    badwords:
      enabled: true
      list: []
      case_sensitive: false
      whole_word: true
      action: "delete_warn"
    anti_raid:
      enabled: true
      join_threshold: 8
      window_seconds: 30
      action: "lock"        # lock | alert | both
      lock_duration_seconds: 300
    anti_invite:
      enabled: true
      whitelist_guilds: []
      action: "delete_warn"
    anti_link:
      enabled: false
      whitelist_domains: ["github.com", "discord.com"]
      action: "delete"
    mass_mention:
      enabled: true
      threshold: 5
      action: "delete_warn"
    anti_caps:
      enabled: false
      min_length: 8
      caps_ratio: 0.7
      action: "delete_warn"
    sanctions:
      warn_expire_days: 30
      progression:
        - { warnings: 3,  action: "mute",   duration: "1h" }
        - { warnings: 5,  action: "mute",   duration: "24h" }
        - { warnings: 7,  action: "kick" }
        - { warnings: 10, action: "ban" }
    log_channel_id: "MOD_LOG_CHANNEL_ID"
    dm_on_action: true
```

### 4.2 Validation

```js
// config/schema.js
const joi = require('joi');

const automodConfigSchema = joi.object({
  enabled: joi.boolean().default(false),
  allowed_roles: joi.array().items(joi.string()).default([]),
  spam: joi.object({
    enabled: joi.boolean().default(true),
    max_messages: joi.number().integer().min(2).default(5),
    window_seconds: joi.number().integer().min(1).default(5),
    max_mentions: joi.number().integer().min(1).default(3),
    mentions_window_seconds: joi.number().integer().min(1).default(10),
    action: joi.string().valid('warn', 'mute', 'kick').default('warn'),
  }),
  // ... etc
});
```

## 5. Base de données

Voir [`../architecture/data-model.md`](../architecture/data-model.md#32-modération-phase-1) pour le détail des tables `user_warnings`, `user_sanctions`, `mod_logs`.

## 6. Services

### 6.1 `SpamDetector`

- Sliding window en mémoire (Map par `guildId` → `userId` → array de timestamps)
- Purge périodique des entrées expirées
- Émet un événement interne `spam.detected` avec métadonnées

```js
class SpamDetector {
  static inject = [];

  constructor() {
    this.userHistory = new Map(); // guildId+userId -> [timestamps]
    this.mentionHistory = new Map();
  }

  /**
   * @returns {{ isSpam: boolean, reason?: string, count?: number }}
   */
  check(guildId, userId, message) {
    // ...
  }
}
```

### 6.2 `RaidDetector`

- Sliding window sur les `guildMemberAdd` (en mémoire + persistance pour analyse)
- Si seuil dépassé, émet `raid.detected`

### 6.3 `Sanctions`

- Lit la config de progression
- Calcule la prochaine sanction à partir du nombre de warns actifs
- Timeout Discord (jusqu'à 28j) via `member.timeout(ms, reason)`
- Kick / ban via `guild.members.kick()` / `guild.members.ban()`
- Écrit dans `user_sanctions` + `mod_logs`

### 6.4 `AutomodEngine` (orchestrateur)

```js
class AutomodEngine {
  static inject = [SpamDetector, RaidDetector, Sanctions, ModLog];

  constructor(spam, raid, sanctions, log) { /* ... */ }

  async onMessageCreate(message) {
    if (!this.isEnabledFor(message.guild.id)) return;

    const result = await this.runAllRules(message);
    if (result.action) {
      await this.applyAction(message, result);
      await this.log.publish(message, result);
    }
  }
}
```

## 7. Événements

### 7.1 `MessageCreateListener`

```js
const { OnEvent } = require('../../core');

@OnEvent('messageCreate', {
  configKey: 'features.automod',
  ignoreBots: true,
  priority: 100,
  filter: (message) => !message.system && !message.webhookId
})
class MessageCreateListener {
  static inject = [AutomodEngine];
  constructor(engine) { this.engine = engine; }

  async handle(message) {
    await this.engine.onMessageCreate(message);
  }
}
```

### 7.2 `GuildMemberAddListener`

```js
@OnEvent('guildMemberAdd', {
  configKey: 'features.automod',
  priority: 50
})
class GuildMemberAddListener {
  static inject = [RaidDetector, Sanctions];
  constructor(raid, sanctions) { /* ... */ }

  async handle(member) {
    if (this.raidDetector.detect(member.guild.id)) {
      await this.sanctions.handleRaid(member.guild);
    }
  }
}
```

## 8. Commandes Discord

### 8.1 `/mod warn <user> <reason>`

Permission : rôle `automod.allowed_roles` ou admin.

```js
@Command({ name: 'mod-warn', description: 'Avertir un utilisateur' })
class ModWarnCommand {
  static inject = [Sanctions, ModLog];
  constructor(sanctions, log) { /* ... */ }

  async execute(interaction) {
    const user = interaction.options.getUser('user');
    const reason = interaction.options.getString('reason');
    const sanction = await this.sanctions.warn(interaction.guild, user, interaction.user, reason, 'manual');
    await this.log.publish(interaction.guild, user, interaction.user, 'warn', reason);
    await interaction.reply({ content: `✅ ${user.tag} averti.`, ephemeral: true });
  }
}
```

### 8.2 Liste des commandes

| Commande | Description | Permission |
|---|---|---|
| `/mod warn` | Avertir un utilisateur | mod |
| `/mod mute` | Timeout un utilisateur | mod |
| `/mod kick` | Kicker un utilisateur | mod |
| `/mod ban` | Bannir un utilisateur | mod |
| `/mod unban` | Débannir un utilisateur | mod |
| `/mod history` | Voir l'historique d'un utilisateur | mod |
| `/mod clear` | Supprimer X messages | mod |
| `/mod automod-config` | Afficher la config actuelle | admin |

## 9. API REST (dashboard)

```
GET    /api/features/automod           # config complète
PATCH  /api/features/automod           # maj config
GET    /api/mod/logs?user=&action=&page=  # liste paginée
GET    /api/mod/users/:id              # profil (warns, sanctions)
POST   /api/mod/sanctions/preview      # simuler une escalade
POST   /api/features/automod/test      # tester une règle sur un message
```

## 10. UI (Nuxt)

- `/features/automod` : page de config complète (sliders, textarea bad-words, import/export)
- `/moderation` : dashboard avec KPIs et table de logs
- `FeatureCard` + `FeatureToggle` : composants génériques

## 11. Tests

```js
// tests/spam-detector.test.js
const { test } = require('node:test');
const assert = require('node:assert');
const { SpamDetector } = require('../src/modules/feature_automod/services/spam-detector.service');

test('SpamDetector: détecte 5 messages en 5s', () => {
  const detector = new SpamDetector();
  const guildId = 'g1';
  const userId = 'u1';

  for (let i = 0; i < 4; i++) {
    const result = detector.check(guildId, userId, { content: 'spam' });
    assert.strictEqual(result.isSpam, false);
  }
  const result = detector.check(guildId, userId, { content: 'spam' });
  assert.strictEqual(result.isSpam, true);
});
```

## 12. Critères d'acceptation (Definition of Done)

- [ ] Toutes les règles de la section 2 sont implémentées et testées
- [ ] Configuration valide au démarrage (joi)
- [ ] Permissions respectées (slash + API)
- [ ] Logs persistés en BDD (`mod_logs`)
- [ ] Page `/features/automod` fonctionnelle
- [ ] Tests unitaires : `SpamDetector`, `RaidDetector`, `Sanctions`, `BadWords`
- [ ] Documentation inline (JSDoc) sur les services
- [ ] Aucune régression sur les features existantes (counter, countdown, captcha…)
