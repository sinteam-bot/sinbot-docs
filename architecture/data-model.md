# Modèle de données

> **ORM** : Drizzle 0.45 — **Driver** : `better-sqlite3` (dev) ou `pg` (prod)
>
> Le code est écrit en SQL-style Drizzle et la migration est gérée par `drizzle-kit`.

## 1. Principes

1. **Une table par agrégat métier** (guild, user, sanction, mod_log, feature_flag…)
2. **Multi-guild ready** : toutes les tables portent un `guild_id` (sauf les tables vraiment globales)
3. **Soft-delete** par défaut (colonne `deleted_at`) pour les entités cache (membres, salons, rôles…)
4. **UUID v4** en PK pour les entités transactionnelles (sanctions, logs, tickets) ; **Discord IDs** (string snowflakes) en PK pour les entités de cache
5. **Audit** : `created_at`, `updated_at`, `created_by` sur les tables critiques

## 2. Schéma global (existant)

> Schéma déjà partiellement en place — voir `src/database/` et `src/database.js`.

### 2.1 Cache Discord (synchronisé via `DiscordCacheService`)

```
guilds           (id PK, name, icon, member_count, created_at, deleted_at)
channels         (id PK, guild_id FK, name, type, parent_id, position, deleted_at)
roles            (id PK, guild_id FK, name, color, permissions, position, deleted_at)
emojis           (id PK, guild_id FK, name, animated, url, deleted_at)
members          (id PK, guild_id FK, username, discriminator, avatar, joined_at, deleted_at)
messages         (id PK, channel_id FK, author_id, content, timestamp, deleted_at)
```

### 2.2 Configuration & runtime

```
scheduled_jobs   (id PK, name, cron, enabled, last_run_at, next_run_at)
ai_generations   (id PK, model, prompt, response, tokens, latency_ms, created_at)
```

### 2.3 Système existant

```
captcha_sessions (id PK, user_id, guild_id, answer, attempts, expires_at, status)
welcome_logs     (id PK, user_id, guild_id, channel_id, message_id, created_at)
bump_reminders   (id PK, guild_id, channel_id, user_id, last_bump_at)
xp_users         (id PK, user_id, guild_id, xp_total, level, last_xp_at, streak)
countdown_runs   (id PK, channel_id, current_number, status, started_at, ended_at)
counter_runs     (id PK, channel_id, current_number, status, started_at, ended_at)
```

## 3. Schéma cible — ajouts pour Draftbot-like

### 3.1 Multi-guild & feature flags (Phase 0)

```sql
-- Une ligne par serveur Discord que le bot connaît
CREATE TABLE guild_settings (
  guild_id        TEXT PRIMARY KEY,           -- Discord snowflake
  name            TEXT NOT NULL,
  locale          TEXT DEFAULT 'fr',
  timezone        TEXT DEFAULT 'Europe/Paris',
  owner_id        TEXT,
  premium_tier    INTEGER DEFAULT 0,          -- 0, 1, 2, 3
  joined_at       INTEGER NOT NULL,           -- epoch ms
  created_at      INTEGER NOT NULL,
  updated_at      INTEGER NOT NULL
);

-- Activation / configuration par feature par guild
CREATE TABLE feature_flags (
  guild_id        TEXT NOT NULL,
  feature_name    TEXT NOT NULL,              -- ex: 'automod', 'tickets', 'logs'
  enabled         INTEGER NOT NULL DEFAULT 0,
  config_json     TEXT NOT NULL DEFAULT '{}', -- configuration spécifique
  allowed_roles   TEXT NOT NULL DEFAULT '[]', -- JSON array de role IDs
  updated_by      TEXT,
  updated_at      INTEGER NOT NULL,
  PRIMARY KEY (guild_id, feature_name)
);
CREATE INDEX idx_feature_flags_enabled ON feature_flags(enabled);
```

### 3.2 Modération (Phase 1)

```sql
-- Avertissements adressés aux utilisateurs
CREATE TABLE user_warnings (
  id              TEXT PRIMARY KEY,            -- UUID v4
  guild_id        TEXT NOT NULL,
  user_id         TEXT NOT NULL,
  mod_id          TEXT NOT NULL,                -- staff qui a warn
  reason          TEXT NOT NULL,
  source          TEXT NOT NULL DEFAULT 'manual', -- 'manual' | 'automod'
  rule            TEXT,                          -- ex: 'spam', 'badwords', 'raid'
  created_at      INTEGER NOT NULL,
  expires_at      INTEGER,                       -- null = permanent
  active          INTEGER NOT NULL DEFAULT 1
);
CREATE INDEX idx_warnings_guild_user ON user_warnings(guild_id, user_id);
CREATE INDEX idx_warnings_active ON user_warnings(active);

-- Sanctions actives (timeout, kick, ban)
CREATE TABLE user_sanctions (
  id              TEXT PRIMARY KEY,
  guild_id        TEXT NOT NULL,
  user_id         TEXT NOT NULL,
  type            TEXT NOT NULL,                -- 'timeout' | 'kick' | 'ban' | 'softban'
  reason          TEXT NOT NULL,
  mod_id          TEXT NOT NULL,
  duration_ms     INTEGER,                      -- null = permanent
  starts_at       INTEGER NOT NULL,
  expires_at      INTEGER,
  revoked_by      TEXT,
  revoked_at      INTEGER,
  revoked_reason  TEXT,
  active          INTEGER NOT NULL DEFAULT 1,
  created_at      INTEGER NOT NULL
);
CREATE INDEX idx_sanctions_guild_user ON user_sanctions(guild_id, user_id);
CREATE INDEX idx_sanctions_active ON user_sanctions(active);

-- Journal d'audit de toutes les actions de modération
CREATE TABLE mod_logs (
  id              TEXT PRIMARY KEY,
  guild_id        TEXT NOT NULL,
  user_id         TEXT NOT NULL,                -- cible
  mod_id          TEXT,                          -- auteur (null si auto)
  action          TEXT NOT NULL,                -- 'warn' | 'mute' | 'kick' | 'ban' | 'clear' | 'lock' | ...
  channel_id      TEXT,
  message_id      TEXT,
  reason          TEXT,
  metadata        TEXT,                          -- JSON libre (compteurs, contexte…)
  source          TEXT NOT NULL DEFAULT 'manual', -- 'manual' | 'automod' | 'system'
  created_at      INTEGER NOT NULL
);
CREATE INDEX idx_modlogs_guild_created ON mod_logs(guild_id, created_at);
CREATE INDEX idx_modlogs_guild_user ON mod_logs(guild_id, user_id);
```

### 3.3 Tickets (Phase 3)

```sql
CREATE TABLE tickets (
  id              TEXT PRIMARY KEY,            -- UUID v4
  guild_id        TEXT NOT NULL,
  channel_id      TEXT NOT NULL,                -- channel / thread créé
  user_id         TEXT NOT NULL,                -- demandeur
  category        TEXT NOT NULL DEFAULT 'support', -- 'support' | 'report' | 'partner'
  subject         TEXT,
  status          TEXT NOT NULL DEFAULT 'open',  -- 'open' | 'claimed' | 'closed'
  claimed_by      TEXT,                          -- staff en charge
  closed_by       TEXT,
  closed_at       INTEGER,
  created_at      INTEGER NOT NULL,
  updated_at      INTEGER NOT NULL
);
CREATE INDEX idx_tickets_guild_status ON tickets(guild_id, status);

CREATE TABLE ticket_messages (
  id              TEXT PRIMARY KEY,
  ticket_id       TEXT NOT NULL,
  author_id       TEXT NOT NULL,
  content         TEXT,
  attachments     TEXT,                          -- JSON
  created_at      INTEGER NOT NULL,
  FOREIGN KEY (ticket_id) REFERENCES tickets(id) ON DELETE CASCADE
);
CREATE INDEX idx_ticket_messages_ticket ON ticket_messages(ticket_id);
```

### 3.4 Giveaways (Phase 5)

```sql
CREATE TABLE giveaways (
  id              TEXT PRIMARY KEY,
  guild_id        TEXT NOT NULL,
  channel_id      TEXT NOT NULL,
  message_id      TEXT NOT NULL,
  host_id         TEXT NOT NULL,
  prize           TEXT NOT NULL,
  winners_count   INTEGER NOT NULL DEFAULT 1,
  starts_at       INTEGER NOT NULL,
  ends_at         INTEGER NOT NULL,
  status          TEXT NOT NULL DEFAULT 'active', -- 'active' | 'ended' | 'cancelled'
  winners         TEXT,                           -- JSON array
  required_role   TEXT,
  created_at      INTEGER NOT NULL
);
CREATE INDEX idx_giveaways_guild_status ON giveaways(guild_id, status);
CREATE INDEX idx_giveaways_ends_at ON giveaways(ends_at);

CREATE TABLE giveaway_entries (
  giveaway_id     TEXT NOT NULL,
  user_id         TEXT NOT NULL,
  entered_at      INTEGER NOT NULL,
  PRIMARY KEY (giveaway_id, user_id)
);
```

### 3.5 Polls (Phase 5)

```sql
CREATE TABLE polls (
  id              TEXT PRIMARY KEY,
  guild_id        TEXT NOT NULL,
  channel_id      TEXT NOT NULL,
  message_id      TEXT NOT NULL,
  question        TEXT NOT NULL,
  options         TEXT NOT NULL,                -- JSON array
  multi_choice    INTEGER NOT NULL DEFAULT 0,
  ends_at         INTEGER,
  status          TEXT NOT NULL DEFAULT 'active',
  created_by      TEXT NOT NULL,
  created_at      INTEGER NOT NULL
);

CREATE TABLE poll_votes (
  poll_id         TEXT NOT NULL,
  user_id         TEXT NOT NULL,
  option_index    INTEGER NOT NULL,
  voted_at        INTEGER NOT NULL,
  PRIMARY KEY (poll_id, user_id, option_index)
);
```

## 4. Relations principales

```
guild_settings 1 ─┬─ N feature_flags
                  ├─ N user_warnings
                  ├─ N user_sanctions
                  ├─ N mod_logs
                  ├─ N tickets
                  ├─ N giveaways
                  └─ N polls

tickets 1 ─── N ticket_messages

giveaways 1 ── N giveaway_entries

polls 1 ── N poll_votes
```

## 5. Stratégie de migration

1. **Phase 0** : création des tables `guild_settings`, `feature_flags` (idempotent avec `CREATE TABLE IF NOT EXISTS`)
2. **Phase 1** : `user_warnings`, `user_sanctions`, `mod_logs`
3. **Phases suivantes** : tables par feature, isolées dans leur module
4. **Migration SQLite → PostgreSQL** : déjà supportée via `scripts/migrate-sqlite-to-postgres.js`
5. **Drizzle Kit** : générer les migrations avec `npx drizzle-kit generate`, valider en CI

## 6. Conventions Drizzle

```js
// src/database/schema/guild_settings.js
const { sqliteTable, text, integer } = require('drizzle-orm/sqlite-core');

const guildSettings = sqliteTable('guild_settings', {
  guildId:    text('guild_id').primaryKey(),
  name:       text('name').notNull(),
  locale:     text('locale').default('fr'),
  timezone:   text('timezone').default('Europe/Paris'),
  ownerId:    text('owner_id'),
  joinedAt:   integer('joined_at').notNull(),
  createdAt:  integer('created_at').notNull(),
  updatedAt:  integer('updated_at').notNull(),
});

module.exports = { guildSettings };
```

```js
// src/database/schema/feature_flags.js
const { sqliteTable, text, integer, primaryKey } = require('drizzle-orm/sqlite-core');

const featureFlags = sqliteTable('feature_flags', {
  guildId:      text('guild_id').notNull(),
  featureName:  text('feature_name').notNull(),
  enabled:      integer('enabled').notNull().default(0),
  configJson:   text('config_json').notNull().default('{}'),
  allowedRoles: text('allowed_roles').notNull().default('[]'),
  updatedBy:    text('updated_by'),
  updatedAt:    integer('updated_at').notNull(),
}, (t) => ({
  pk: primaryKey({ columns: [t.guildId, t.featureName] }),
}));

module.exports = { featureFlags };
```

## 7. Soft-delete

Toutes les tables cache (membres, salons, rôles…) ont une colonne `deleted_at` (INTEGER nullable, epoch ms). Les requêtes de lecture filtrent par `deleted_at IS NULL`.

`DiscordCacheService.softDelete*(id)` met à jour la colonne sans supprimer la ligne, ce qui permet de garder un historique pour les logs et l'audit.
