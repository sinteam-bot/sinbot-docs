# Phase 0 — FeatureRegistry & multi-guild ready

> **Durée estimée** : 1-2 jours
>
> **Objectif** : poser les fondations pour que toutes les features Draftbot-like soient activables/désactivables par serveur, avec permissions fines, et pilotables depuis le dashboard.

## 1. Pourquoi cette phase est critique

Aujourd'hui, la configuration se fait via `config.yml` (fichier unique, statique) et chaque feature est soit active, soit désactivée globalement. Pour des fonctionnalités type Draftbot, on a besoin de :

- **Activer / désactiver par serveur** (le bot pourrait être multi-guild demain)
- **Configurer par serveur** (chaque serveur a ses propres bad-words, ses propres seuils…)
- **Permissions fines** (qui peut utiliser / configurer la feature)
- **Pilotage web** (toggle on/off depuis le dashboard, sans redémarrer le bot)

## 2. Livrables

### 2.1 Base de données

Créer 2 tables (Drizzle) :

```sql
CREATE TABLE guild_settings (
  guild_id   TEXT PRIMARY KEY,
  name       TEXT NOT NULL,
  locale     TEXT DEFAULT 'fr',
  timezone   TEXT DEFAULT 'Europe/Paris',
  owner_id   TEXT,
  joined_at  INTEGER NOT NULL,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE TABLE feature_flags (
  guild_id       TEXT NOT NULL,
  feature_name   TEXT NOT NULL,
  enabled        INTEGER NOT NULL DEFAULT 0,
  config_json    TEXT NOT NULL DEFAULT '{}',
  allowed_roles  TEXT NOT NULL DEFAULT '[]',
  updated_by     TEXT,
  updated_at     INTEGER NOT NULL,
  PRIMARY KEY (guild_id, feature_name)
);
```

Voir [`../architecture/data-model.md`](../architecture/data-model.md#31-multi-guild--feature-flags-phase-0).

### 2.2 Module `feature-registry`

`src/core/feature-registry.js` :

```js
const { eventBus } = require('./event-bus');
const { getConfig } = require('../config');

class FeatureRegistry {
  constructor() {
    this.features = new Map(); // name -> { defaults, configSchema, onEnable, onDisable, requires }
  }

  /**
   * Déclare une feature
   * @param {string} name
   * @param {{
   *   defaults: object,
   *   configSchema?: object,        // joi schema
   *   onEnable?: (guildId) => Promise<void>,
   *   onDisable?: (guildId) => Promise<void>,
   *   requires?: string[]            // autres features requises
   * }} definition
   */
  define(name, definition) {
    this.features.set(name, definition);
  }

  /**
   * Récupère l'état d'une feature pour un guild (DB > YAML fallback)
   */
  async get(guildId, name) {
    // 1. Chercher en DB
    const row = await db.select().from(featureFlags)
      .where(and(eq(featureFlags.guildId, guildId), eq(featureFlags.featureName, name)))
      .limit(1);

    if (row[0]) {
      return {
        enabled: !!row[0].enabled,
        config: JSON.parse(row[0].config_json || '{}'),
        allowedRoles: JSON.parse(row[0].allowed_roles || '[]'),
        source: 'db'
      };
    }

    // 2. Fallback YAML
    const yamlConfig = getConfig().features?.[name];
    if (yamlConfig) {
      return {
        enabled: !!yamlConfig.enabled,
        config: yamlConfig,
        allowedRoles: yamlConfig.allowed_roles || [],
        source: 'yaml'
      };
    }

    // 3. Défaut
    const def = this.features.get(name);
    return {
      enabled: false,
      config: def?.defaults || {},
      allowedRoles: [],
      source: 'default'
    };
  }

  /**
   * Active ou désactive une feature
   */
  async set(guildId, name, { enabled, config, allowedRoles, updatedBy }) {
    const existing = await db.select()...;
    if (existing[0]) {
      await db.update(featureFlags)
        .set({ enabled: enabled ? 1 : 0, config_json: JSON.stringify(config || {}), allowed_roles: JSON.stringify(allowedRoles || []), updated_by: updatedBy, updated_at: Date.now() })
        .where(...);
    } else {
      await db.insert(featureFlags).values({ ... });
    }

    // Hook
    const def = this.features.get(name);
    if (enabled && def?.onEnable) await def.onEnable(guildId);
    if (!enabled && def?.onDisable) await def.onDisable(guildId);

    // Émettre un event pour le dashboard
    eventBus.emit('feature.updated', { guildId, name, enabled });
  }

  /**
   * Vérifie si un utilisateur a accès à la feature
   */
  async canUse(guildId, userId, name) {
    const state = await this.get(guildId, name);
    if (!state.enabled) return { allowed: false, reason: 'disabled' };

    if (state.allowedRoles.length === 0) return { allowed: true, reason: 'no_restriction' };

    const member = await guild.members.fetch(userId).catch(() => null);
    if (!member) return { allowed: false, reason: 'not_member' };

    const hasRole = state.allowedRoles.some(roleId => member.roles.cache.has(roleId));
    if (!hasRole) return { allowed: false, reason: 'missing_role' };

    return { allowed: true, reason: 'role_match' };
  }

  list() {
    return Array.from(this.features.entries()).map(([name, def]) => ({ name, defaults: def.defaults }));
  }
}

const featureRegistry = new FeatureRegistry();
module.exports = { FeatureRegistry, featureRegistry };
```

### 2.3 Intégration dans les features existantes

Chaque module doit enregistrer sa déclaration au démarrage :

```js
// src/modules/feature_xp-level/xp-level.module.js
const { featureRegistry } = require('../../core/feature-registry');
const { xpDefaults } = require('./config/defaults');

featureRegistry.define('xp', {
  defaults: xpDefaults,
  configSchema: require('./config/schema'),
  onEnable: async (guildId) => {
    console.log(`[xp] activé sur ${guildId}`);
  },
  onDisable: async (guildId) => {
    console.log(`[xp] désactivé sur ${guildId}`);
  }
});
```

### 2.4 Compatibilité ascendante

Les features actuelles (`captcha`, `welcome`, `xp`, `counter`, `countdown`, `bump_reminder`, `daily_message`) **continuent de fonctionner** avec la config YAML existante.

Une nouvelle section `features.*` dans le YAML **surcharge** la config par défaut :

```yaml
# Ancien format (toujours supporté)
captcha:
  enabled: true
  captcha_channel_name: "✅-verification-captcha"

# Nouveau format (priorité sur l'ancien si présent)
features:
  captcha:
    enabled: true
    # config additionnelle possible
```

> La migration progressive est traitée dans [`migration-yaml.md`](./migration-yaml.md).

### 2.5 Dashboard (Nuxt)

- **Page `/features`** : liste toutes les features enregistrées avec un toggle on/off
- **Composable `useFeatures`** : `list()`, `get(name)`, `update(name, payload)`
- **Store Pinia** : `stores/features.ts`
- **Middleware d'auth** : vérifie la clé API avant d'afficher la page

```vue
<!-- pages/features/index.vue -->
<script setup>
const { list, update } = useFeatures();
const features = ref([]);

onMounted(async () => {
  features.value = await list();
});

const toggle = async (name) => {
  const f = features.value.find(x => x.name === name);
  await update(name, { enabled: !f.enabled });
  f.enabled = !f.enabled;
};
</script>

<template>
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    <FeatureCard
      v-for="f in features"
      :key="f.name"
      :feature="f"
      @toggle="toggle(f.name)"
    />
  </div>
</template>
```

### 2.6 API REST

```
GET    /api/features                  # toutes les features + état
GET    /api/features/:name            # détail
PATCH  /api/features/:name            # { enabled, config, allowedRoles }
POST   /api/features/:name/test       # tester une règle (réservé automod)
GET    /api/features/:name/can-use    # { userId } → { allowed, reason }
```

```js
// src/modules/feature_registry/feature-registry.controller.js
const { Get, Patch, Post, Controller } = require('../../core');
const { featureRegistry } = require('../../core/feature-registry');

@Controller('/api/features')
class FeatureRegistryController {
  @Get('/')
  async list(req, res) {
    const guildId = req.query.guild_id || process.env.DISCORD_GUILD_ID;
    const features = featureRegistry.list();
    const states = await Promise.all(
      features.map(async f => ({
        ...f,
        state: await featureRegistry.get(guildId, f.name)
      }))
    );
    return states;
  }

  @Patch('/:name')
  async update(req, res) {
    const { name } = req.params;
    const { enabled, config, allowedRoles } = req.body;
    await featureRegistry.set(req.body.guildId, name, {
      enabled,
      config,
      allowedRoles,
      updatedBy: req.body.userId
    });
    return { ok: true };
  }

  @Post('/:name/can-use')
  async canUse(req, res) {
    const { name } = req.params;
    const { guildId, userId } = req.body;
    return featureRegistry.canUse(guildId, userId, name);
  }
}
```

## 3. Étapes d'implémentation

### Étape 1 — DB (2h)

- [ ] Ajouter `guild_settings` et `feature_flags` dans `src/database/schema/`
- [ ] Générer la migration Drizzle
- [ ] Tester `db/seed.js` avec un guild factice

### Étape 2 — `FeatureRegistry` (3h)

- [ ] Implémenter `core/feature-registry.js`
- [ ] Tests unitaires `tests/feature-registry.test.js`
- [ ] Brancher dans `core/index.js`

### Étape 3 — Déclaration des features existantes (2h)

- [ ] Refactorer chaque module pour appeler `featureRegistry.define(name, ...)`
- [ ] Conserver la rétrocompatibilité avec la config YAML
- [ ] Tests d'intégration : toggle en DB, vérification de la prise en compte

### Étape 4 — API REST (2h)

- [ ] Créer `FeatureRegistryController`
- [ ] Protéger par `requireApiKey` middleware
- [ ] Tests avec `supertest` ou `node:test`

### Étape 5 — UI Nuxt (3h)

- [ ] Page `/features` avec toggles
- [ ] Composant `FeatureCard.vue` (réutilisable)
- [ ] Store `features.ts` Pinia
- [ ] Middleware d'auth sur les pages admin

### Étape 6 — Documentation & exemples (1h)

- [ ] Documenter le pattern dans `guides/create-a-feature.md`
- [ ] Ajouter un exemple minimal d'une feature tierce

## 4. Critères d'acceptation (DoD)

- [ ] Tables créées et migrées sans casser la DB existante
- [ ] `featureRegistry.get/set/canUse` testés unitairement
- [ ] Toggle on/off d'une feature depuis l'API fonctionne et **prend effet sans redémarrage** (via invalidation du cache config)
- [ ] Permissions respectées (`canUse` retourne `false` si l'utilisateur n'a pas le rôle)
- [ ] Page `/features` opérationnelle, sécurisée par API key
- [ ] Documentation à jour
- [ ] Aucune régression sur les modules existants

## 5. Risques & mitigations

| Risque | Mitigation |
|---|---|
| La lecture DB à chaque event ralentit le bot | Cache en mémoire (TTL 30s) + invalidation sur `feature.updated` |
| Multi-guild : fuites de données entre serveurs | Toujours passer `guildId` dans toutes les requêtes, audit complet |
| Rétrocompat YAML → confusion | Documenter la précédence (DB > YAML > defaults), message d'avertissement au démarrage si les deux sont utilisés |
| `onEnable` / `onDisable` mal implémentés | Tests d'intégration obligatoires, doc claire |
