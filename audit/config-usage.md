# Audit — Système de config actuel

> **Date** : 2026-08-29
> **Contexte** : préparation de la migration vers c12 (cf. [`../plan/migrate-to-c12.md`](../plan/migrate-to-c12.md))
> **Périmètre** : `src/core/feature-registry.js`, `src/db/schemas/shared/feature-flags.js`, `src/modules/*/config/defaults.js`, `src/config/index.js`

---

## 1. Le `FeatureRegistry` — composant central

**Fichier** : `src/core/feature-registry.js` (296 lignes)

### 1.1 Interface publique

| Méthode | Signature | Description |
|---|---|---|
| `define(name, definition)` | `(name: string, definition: {defaults, configSchema, onEnable, onDisable, requires})` | Déclare une feature dans le registre en mémoire |
| `get(guildId, name)` | `(guildId, name) => Promise<{enabled, config, allowedRoles, source}>` | Lit l'état d'une feature (DB > YAML > defaults) |
| `set(guildId, name, patch)` | `(guildId, name, {enabled, config, allowedRoles, updatedBy}) => Promise<...>` | Upsert en DB, émet `feature.updated` |
| `list()` | `() => Array<{name, defaults, ...}>` | Liste les features déclarées |
| `listForGuild(guildId)` | `(guildId) => Promise<Array<{name, defaults, state}>>` | Liste les features avec leur état pour une guilde |
| `canUse(guildId, userId, name)` | `(guildId, userId, name) => Promise<{allowed, reason}>` | Vérifie les permissions RBAC |
| `_reset()` | `() => void` | Vide le cache (tests) |

### 1.2 Sources de données actuelles (par ordre de priorité dans `get`)

1. **DB** : table `feature_flags` (colonne `enabled`, `config_json`, `allowed_roles`, `updated_by`, `updated_at`)
2. **YAML** : `config.features.<name>` (legacy : `config.<name>`)
3. **Defaults** : passés à `define()` (mémoire)

### 1.3 Problèmes identifiés

- **Double source de vérité** : DB + YAML peuvent être désynchronisés (bug vécu avec le captcha où l'API lit YAML mais écrit en DB)
- **Hot reload impossible** : la DB ne se met pas à jour toute seule
- **Tests complexes** : nécessitent un mock de DB ou une vraie DB
- **Pas de versioning** : pas de moyen de revenir à une version antérieure d'une config
- **Schema Drizzle couplé** : impossible de changer le format sans migration

---

## 2. `featureFlags` — table DB à supprimer

**Fichier** : `src/db/schemas/shared/feature-flags.js` (39 lignes)

### 2.1 Schema

```js
featureFlags = pgTable('feature_flags', {
    guildId: text('guild_id').notNull(),
    featureName: text('feature_name').notNull(),
    enabled: integer('enabled').notNull().default(0),
    configJson: text('config_json').notNull().default('{}'),
    allowedRoles: text('allowed_roles').notNull().default('[]'),
    updatedBy: text('updated_by'),
    updatedAt: bigint('updated_at', { mode: 'number' }).notNull()
}, (table) => [
    primaryKey({ columns: [table.guildId, table.featureName] }),
    index('idx_pg_feature_flags_enabled').on(table.enabled)
]);
```

### 2.2 Référencée dans

| Fichier | Raison |
|---|---|
| `src/core/feature-registry.js` | `get()` lit la table, `set()` upsert |
| `src/db/schemas/legacy.js` (ligne 27) | Importé dans le barrel |
| `src/db/migrations/0000_chubby_romulus.sql` | Créée par la migration initiale |
| `src/db/migrations/0001_flashy_forgotten_one.sql` | Snapshot |

---

## 3. `defaults.js` — 13 fichiers à migrer

Liste des features qui ont un `defaults.js` :

| Fichier (src) | Feature name (c12) | Lignes |
|---|---|---|
| `src/modules/security_automod/config/defaults.js` | `automod` | ~20 |
| `src/modules/engagement_birthdays/config/defaults.js` | `birthdays` | ~10 |
| `src/modules/engagement_economy/config/defaults.js` | `economy` | ~50 |
| `src/modules/game_engagement/config/defaults.js` | `engagement` | ~10 |
| `src/modules/util_reminders/config/defaults.js` | `engagement_advanced` | ~15 |
| `src/modules/util_info/config/defaults.js` | `info` | ~10 |
| `src/modules/util_invites/config/defaults.js` | `invites` | ~50 |
| `src/modules/security_logs/config/defaults.js` | `logs` | ~5 |
| `src/modules/community_reaction-roles/config/defaults.js` | `reaction-roles` | ~20 |
| `src/modules/community_reports/config/defaults.js` | `reports` | ~20 |
| `src/modules/community_sticky-roles/config/defaults.js` | `sticky-roles` | ~5 |
| `src/modules/util_temp-voice/config/defaults.js` | `temp-voice` | ~20 |
| `src/modules/community_tickets/config/defaults.js` | `tickets` | ~30 |

**Total** : ~13 fichiers, ~250 lignes cumulées (à convertir en YAML)

### 3.1 Schémas de validation joi (info supplémentaire)

Plusieurs `config/schema.js` (utilisent `joi` pour valider la config) :

- `src/modules/util_invites/config/schema.js` (29 lignes)

⚠️ **Note** : `joi` n'est **pas installé** comme dépendance. Le code a été supprimé dans une PR précédente. Si on veut réintroduire la validation, on ajoutera `joi` ou on passera à `zod` (compatible avec c12 ?).

---

## 4. `featureRegistry.define` — 13 déclarations

Liste des features déclarées via `define()` dans le registre centralisé (`src/modules/feature-declarations.js`) :

| Feature | defaults ref | onEnable log | onDisable log |
|---|---|---|---|
| `xp` | `defaultFor('xp')` | `✨ [xp] enabled` | `💤 [xp] disabled` |
| `welcome` | `defaultFor('welcome')` | `👋 [welcome] enabled` | `💤 [welcome] disabled` |
| `daily_message` | `defaultFor('daily_message')` | `📅 [daily_message] enabled` | `💤 [daily_message] disabled` |
| `counter` | `defaultFor('counter')` | `🔢 [counter] enabled` | `💤 [counter] disabled` |
| `countdown` | `defaultFor('countdown')` | `⏳ [countdown] enabled` | `💤 [countdown] disabled` |
| `bump_reminder` | `defaultFor('bump_reminder')` | `⏰ [bump_reminder] enabled` | `💤 [bump_reminder] disabled` |
| `captcha` | `defaultFor('captcha')` (+ alias `security_question`) | `🔒 [captcha] enabled` | `💤 [captcha] disabled` |
| `welcome` (doublon) | idem | idem | idem |

⚠️ **Note** : `welcome` est déclaré **deux fois**. La 2ème déclaration est silencieusement ignorée (le `define` ne fait rien si la feature existe déjà). À nettoyer en Phase 6.

### 4.1 Déclarations faites dans les modules eux-mêmes (pas centralisées)

Modules qui font leur propre `featureRegistry.define()` dans leur `*.module.js` :

| Module | Feature name |
|---|---|
| `feature_automod/automod.module.js` | `automod` (doublon avec centralisé) |
| `feature_birthdays/birthdays.module.js` | `birthdays` (doublon) |
| `feature_economy/economy.module.js` | `economy` |
| `feature_engagement/engagement.module.js` | `engagement` (doublon) |
| `feature_engagement-advanced/engagement-advanced.module.js` | `engagement_advanced` |
| `feature_info/info.module.js` | `info` |
| `feature_invites/invites.module.js` | `invites` |
| `feature_logs/logs.module.js` | `logs` (doublon) |
| `feature_reaction-roles/reaction-roles.module.js` | `reaction-roles` |
| `feature_reports/reports.module.js` | `reports` |
| `feature_sticky-roles/sticky-roles.module.js` | `sticky-roles` |
| `feature_temp-voice/temp-voice.module.js` | `temp-voice` |
| `feature_tickets/tickets.module.js` | `tickets` |

⚠️ **Problème** : il y a des doublons de déclaration entre `feature-declarations.js` (centralisé) et chaque `*.module.js` (décentralisé). Les 2èmes sont silencieusement ignorées. **À unifier en Phase 6** : tout centraliser dans `feature-declarations.js` ou tout décentraliser dans chaque module, mais pas les deux.

---

## 5. Usages de `featureRegistry.get()` — 23 fichiers

| Fichier | Usage |
|---|---|
| `src/core/feature-registry.js` | `set()` (interne) |
| `src/modules/feature-declarations.js` | (aucun direct) |
| `src/modules/security_automod/events/message-create.listener.js` | check `isEnabled('automod')` |
| `src/modules/security_automod/controllers/automod.controller.js` | get state for response |
| `src/modules/security_automod/automod.module.js` | (via `define`) |
| `src/modules/engagement_birthdays/birthdays.module.js` | (via `define`) |
| `src/modules/welcome_cards/events/card-listeners.js` | get `cards` config |
| `src/modules/engagement_economy/events/drop-reaction-listener.js` | check `isEnabled('economy')` |
| `src/modules/engagement_economy/economy.module.js` | (via `define`) |
| `src/modules/game_engagement/events/interaction-create.listener.js` | get `engagement` config |
| `src/modules/util_reminders/events/message-create.listener.js` | get state |
| `src/modules/util_info/info.module.js` | (via `define`) |
| `src/modules/util_invites/commands/invite-commands.js` | get config for default values |
| `src/modules/util_invites/services/invites.service.js` | get enabled state for guild |
| `src/modules/util_invites/invites.module.js` | (via `define`) |
| `src/modules/security_logs/events/logs-listeners.js` | check `isEnabled('logs')` |
| `src/modules/security_logs/logs.module.js` | (via `define`) |
| `src/modules/community_reaction-roles/events/reaction-listener.js` | get state |
| `src/modules/community_reaction-roles/reaction-roles.module.js` | (via `define`) |
| `src/modules/community_reports/events/reports-listener.js` | get state |
| `src/modules/community_reports/reports.module.js` | (via `define`) |
| `src/modules/community_sticky-roles/events/sticky-roles-listener.js` | get state |
| `src/modules/community_sticky-roles/sticky-roles.module.js` | (via `define`) |
| `src/modules/util_temp-voice/events/temp-voice-listener.js` | get state |
| `src/modules/util_temp-voice/temp-voice.module.js` | (via `define`) |
| `src/modules/community_tickets/events/interaction-create.listener.js` | get state |
| `src/modules/community_tickets/events/message-create.listener.js` | get state |
| `src/modules/community_tickets/tickets.module.js` | (via `define`) |
| `src/web/featuresRouter.js` | API endpoint |

---

## 6. Usages de `featureRegistry.set()` — 2 fichiers

| Fichier | Usage |
|---|---|
| `src/core/feature-registry.js` | (interne, appelé par `set()`) |
| `src/web/featuresRouter.js` | `PATCH /api/features/:name` (admin only) |

---

## 7. `config.yml` (racine) — 230 lignes

**Fichier** : `config.yml` (à la racine, voir aussi `config.example.yml` et `config.chienne.yml`)

### 7.1 Sections principales

- `web` : port, CORS, domain
- `discord` : client_id, guild_id, token (via env)
- `database` : url, ssl, pool
- `logger` : level
- `scheduler` : timezone, tasks
- `webhook` : config
- **features** : sous-sections par feature (`xp`, `welcome`, `daily_message`, `counter`, `countdown`, `bump_reminder`, `captcha`)

### 7.2 Problèmes

- Le `config.yml` est mélangé avec des secrets en dev (`.env.example` montre ce qui doit être en env)
- Le `config.yml` est versionné (sauf `config.chienne.yml` qui semble être la version dev)
- L'API `GET /api/config` renvoie un sous-ensemble du `config.yml` (filtré)
- `POST /api/config` écrit dans `config.yml` (legacy, à migrer)

---

## 8. `src/config/index.js` — point d'entrée

**Fichier** : `src/config/index.js`

### 8.1 API actuelle

| Fonction | Description |
|---|---|
| `getConfig()` | Lit `config.yml` (avec cache mémoire, mutable) |
| `getConfigPath()` | Retourne le chemin du fichier YAML |
| `saveModuleConfig(module, config)` | Écrit dans `config.yml` via `js-yaml` |

### 8.2 Problèmes

- Pas de hot reload (cache mémoire invalidé seulement sur `saveModuleConfig`)
- `js-yaml` utilisé en interne, à remplacer par c12
- Pas de validation de schéma

---

## 9. Routes API concernées

### 9.1 `src/web/featuresRouter.js`

| Route | Méthode | Code actuel |
|---|---|---|
| `/api/features` | GET | `featureRegistry.listForGuild(guildId)` |
| `/api/features/:name` | GET | `featureRegistry.get(guildId, name)` |
| `/api/features/:name` | PATCH | `featureRegistry.set(guildId, name, patch)` (admin only) |
| `/api/features/:name/can-use` | POST | `featureRegistry.canUse(...)` |

### 9.2 `src/web/webRouter.js`

| Route | Méthode | Code actuel |
|---|---|---|
| `/api/config` | GET | `getConfig()` (lecture `config.yml`) |
| `/api/config` | POST | `saveModuleConfig(module, config)` (écriture `config.yml`) |

---

## 10. Total des modifications

| Catégorie | Nombre |
|---|---|
| Fichiers utilisant `featureRegistry` | **29** |
| Fichiers `defaults.js` à migrer en YAML | **13** |
| Fichiers `featureFlags` (DB) référencés | **3** (`feature-flags.js`, `legacy.js`, `feature-registry.js`) |
| Routes API à adapter | **6** |
| Fichiers de config YAML à migrer | **1** (`config.yml` → `data/common/*.yml`) |

---

## 11. Recommandations pour la Phase 1+

1. **Centraliser les `define()`** : tout dans `feature-declarations.js`, plus dans les `*.module.js`
2. **Supprimer les doublons** : `welcome` est déclaré 2 fois
3. **Unifier les call-sites** : remplacer `featureRegistry.get/set` par `c12Loader.getFeatureConfig/setFeatureConfig` directement (le `FeatureRegistry` devient optionnel ou est réécrit comme wrapper)
4. **Ajouter un cache LRU** : pour ne pas relire les YAML à chaque appel
5. **Tester le hot reload** : un test qui modifie un YAML et vérifie que l'event est émis
