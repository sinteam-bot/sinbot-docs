# Modèle de données

> **ORM** : Drizzle 0.45
> **Driver** : `pg` (production) ou `@electric-sql/pglite` (dev / tests, PostgreSQL 16 WASM in-memory)
> **Migrations** : `drizzle-kit`, versionnées dans `src/db/migrations/`
> **Date** : 2026-08-28 — refactor post-audit (cf. `docs/plan/db-repository-split.md`)

## 1. Principes

1. **Une table par agrégat métier** : chaque table appartient à un module.
2. **Multi-guild ready** : toutes les tables portent un `guild_id` (sauf les tables vraiment globales).
3. **Soft-delete** par défaut (colonne `deleted_at`) pour les entités cache (membres, salons, rôles…).
4. **IDs** :
   - **UUID v4** (TEXT) en PK pour les entités transactionnelles (sanctions, logs, tickets, reports).
   - **Discord IDs** (TEXT, snowflakes) en PK pour les entités de cache.
   - **Composite keys** (guild_id + entity_id) pour les agrégats multi-guild (economy, inventory, sticky_roles).
5. **Audit** : `created_at`, `updated_at` sur les tables critiques.
6. **Schémas par module** : chaque module expose son schema Drizzle dans son propre `db/schema.js`. Pas de god-file.

## 2. Architecture du schema

```
src/
├── db/
│   ├── client.js                # factory de connexion (PGlite / pg)
│   ├── index.js                 # barrel public (db, schema, ready)
│   ├── schemas/
│   │   ├── _drizzle.js          # helpers Drizzle centralisés
│   │   ├── legacy.js            # barrel d'agrégation (conservé pour rétrocompat)
│   │   ├── index.js             # réexporte legacy.js (utilisé par client.js)
│   │   ├── shared/              # tables transverses (multiples modules)
│   │   │   ├── audit.js         # user_events, form_responses, discord_events_archive
│   │   │   ├── cache.js         # server_members, discord_channels/roles/…, guild_stats
│   │   │   ├── feature-flags.js # guild_settings, feature_flags
│   │   │   ├── bot-info.js      # bot_version_state
│   │   │   ├── openai.js        # openaimessages
│   │   │   ├── audit.repository.js          # ← wrapper Drizzle
│   │   │   ├── members.repository.js        # ← wrapper Drizzle
│   │   │   ├── commands.repository.js       # ← wrapper Drizzle
│   │   │   ├── discord-cache.repository.js  # ← wrapper Drizzle
│   │   │   ├── dump-discord.repository.js   # ← wrapper Drizzle
│   │   │   └── openai.repository.js         # ← wrapper Drizzle
│   │   └── migrations/          # généré par drizzle-kit (versionné git)
│   │       ├── 0000_chubby_romulus.sql      # baseline (~50 tables)
│   │       └── meta/_journal.json
│   ├── legacy-bridge.js         # pont namespace par module
│   └── legacy-bridge-impl.js    # implémentation legacy des 70 fonctions
└── modules/
    ├── feature_xp-level/db/schema.js       # 5 tables XP
    ├── feature_birthdays/db/schema.js      # 5 tables birthdays
    ├── feature_automod/db/schema.js        # 4 tables (warnings, sanctions, mod_logs, event_log)
    ├── feature_tickets/db/schema.js        # 2 tables
    ├── feature_welcome/db/schema.js        # 2 tables
    ├── feature_economy/db/schema.js        # 6 tables
    ├── feature_reports/db/schema.js        # 2 tables
    ├── feature_reaction-roles/db/schema.js # 1 table
    ├── feature_temp-voice/db/schema.js     # 2 tables
    ├── feature_sticky-roles/db/schema.js   # 1 table
    ├── feature_engagement/db/schema.js     # 7 tables (giveaways, polls, reminders, …)
    ├── feature_info/db/schema.js           # 3 tables (auth)
    ├── security_question/db/schema.js      # 2 tables (captcha)
    ├── service_bump-reminder/db/schema.js  # 1 table
    ├── game_count-down/db/schema.js        # 2 tables
    ├── game_road-to-infinite/db/schema.js  # 1 table
    └── …
```

## 3. Schéma par module (50 tables au total)

### 3.1 Module XP (`feature_xp-level`)

```
user_xp            (id PK, user_id UQ, username, xp, level, total_xp_earned,
                    messages_count, voice_minutes, events_participated, last_message_xp,
                    created_at, updated_at)
xp_transactions    (id PK, user_id, username, xp_amount, xp_type, description, metadata, created_at)
voice_sessions     (id PK, user_id, username, channel_id, channel_name, join_time,
                    leave_time, duration_minutes, xp_earned)
events             (id PK, event_name, event_description, event_date, xp_reward, created_by,
                    is_active, created_at)
event_participants (id PK, event_id, user_id, username, xp_earned, joined_at,
                    UQ(event_id, user_id))
```

### 3.2 Module Birthdays (`feature_birthdays`)

```
user_birthdays            (id PK, user_id UQ, username, birthdate, created_at, updated_at)
birthday_guild_settings   (guild_id PK, mode, announce_channel_id, announce_hour,
                            announce_timezone, ping_role_id, message_template,
                            temp_role_id, enabled, created_at, updated_at)
birthday_visibility       (user_id, guild_id, enabled, updated_at, PK(user_id, guild_id))
birthday_change_log       (id PK, user_id, guild_id, change_number, previous_birthdate,
                            new_birthdate, cooldown_until, changed_at)
birthday_history          (id PK, guild_id, user_id, username, age, message_id,
                            gifts_given, announced_at)
```

### 3.3 Module Automod (`feature_automod`)

```
user_warnings  (id PK, guild_id, user_id, mod_id, reason, source, rule,
                created_at, expires_at, active)
user_sanctions (id PK, guild_id, user_id, type, reason, mod_id, duration_ms,
                starts_at, expires_at, revoked_by, revoked_at, revoked_reason,
                active, created_at)
mod_logs       (id PK, guild_id, user_id, mod_id, action, channel_id, message_id,
                reason, metadata, source, created_at)
event_log      (id PK, guild_id, event_type, actor_id, target_id, channel_id,
                metadata, summary, created_at)
```

### 3.4 Module Tickets (`feature_tickets`)

```
tickets         (id PK, guild_id, channel_id, user_id, category, subject, status,
                 claimed_by, closed_by, closed_at, created_at, updated_at)
ticket_messages (id PK, ticket_id, author_id, content, attachments, is_staff, created_at)
```

### 3.5 Module Welcome (`feature_welcome`)

```
welcome_config (id PK, guild_id UQ, welcome_channel_id, welcome_message, auto_roles,
                is_enabled, created_at, updated_at)
welcome_cards  (id PK, guild_id, user_id, template, payload, svg,
                created_at, expires_at)
```

### 3.6 Module Economy (`feature_economy`)

```
user_economy         (user_id, guild_id, balance, bank_balance, last_daily_claim_at,
                      total_earned, total_spent, created_at, updated_at,
                      PK(user_id, guild_id))
economy_transactions (id PK, guild_id, user_id, amount, type, counterparty_id,
                      reason, metadata, created_at)
shop_items           (id PK, guild_id, name, description, emoji, price, role_reward_id,
                      xp_reward, is_tradeable, is_droppable, max_per_user,
                      created_at, updated_at)
user_inventory       (user_id, guild_id, item_id, quantity, acquired_at,
                      PK(user_id, guild_id, item_id))
inventory_drops      (id PK, guild_id, channel_id, message_id, item_id, quantity,
                      started_at, expires_at, claimed_by, claimed_at, status)
inventory_transfers  (id PK, guild_id, from_user_id, to_user_id, item_id, quantity,
                      type, price, created_at)
```

### 3.7 Module Reports (`feature_reports`)

```
reports        (id PK, guild_id, reporter_id, reported_id, channel_id, message_id,
                reason, category, status, resolved_by, resolved_at, created_at)
report_actions (id PK, report_id, staff_id, action, notes, created_at)
```

### 3.8 Module Reaction Roles (`feature_reaction-roles`)

```
reaction_roles (id PK, guild_id, channel_id, message_id, emoji, role_id,
                description, mode, kind, metadata, created_at, updated_at)
```

### 3.9 Module Temp Voice (`feature_temp-voice`)

```
temp_voice_config (guild_id PK, category_id, format, delete_delay_seconds,
                   max_per_guild, locked_role_id, join_channels_json,
                   enabled, updated_at)
temp_voice_state  (channel_id PK, guild_id, creator_id, last_empty_at, created_at)
```

### 3.10 Module Sticky Roles (`feature_sticky-roles`)

```
sticky_roles (user_id, guild_id, role_id, saved_at, PK(user_id, guild_id, role_id))
```

### 3.11 Module Engagement (`feature_engagement`)

```
giveaways         (id PK, guild_id, channel_id, message_id, host_id, prize,
                   description, winners_count, required_role_id, starts_at, ends_at,
                   status, winners_json, color, created_at, updated_at)
giveaway_entries  (giveaway_id, user_id, entered_at, PK(giveaway_id, user_id))
polls             (id PK, guild_id, channel_id, message_id, question, options_json,
                   multi_choice, anonymous, ends_at, status, created_by, created_at)
poll_votes        (poll_id, user_id, option_index, voted_at,
                   PK(poll_id, user_id, option_index))
reminders         (id PK, guild_id, channel_id, user_id, reminder_text,
                   fire_at, created_at, status, source_message_id)
word_triggers     (id PK, guild_id, trigger_text, match_type, response_text,
                   response_embed_json, exclude_channel_ids_json,
                   exclude_role_ids_json, cooldown_seconds, created_by,
                   created_at, updated_at)
custom_commands   (id PK, guild_id, name, response_text, response_embed_json,
                   restrict_channel_ids_json, restrict_role_ids_json,
                   cooldown_seconds, created_by, created_at, updated_at,
                   UQ(guild_id, name))
```

### 3.12 Module Captcha (`security_question`)

```
user_captchas  (id PK, user_id, username, guild_id, question, answer, channel_id,
                attempts, is_verified, created_at, expires_at, verified_at,
                expired_at, updated_at, UQ(user_id, guild_id))
captcha_config (id PK, guild_id UQ, channel_id, verified_role_id, timeout_minutes,
                max_attempts, is_enabled, created_at, updated_at)
```

### 3.13 Module Auth / Info (`feature_info`)

```
auth_sessions         (id PK, user_id, username, avatar_url, role,
                       refresh_token_hash, ip_address, user_agent,
                       expires_at, created_at, updated_at, revoked_at)
auth_audit_logs       (id PK, event_type, user_id, username, ip_address, user_agent,
                       reason, metadata, created_at)
auth_failed_attempts  (identifier PK, attempt_count, first_attempt_at,
                       last_attempt_at, blocked_until)
```

### 3.14 Service Bump Reminder (`service_bump-reminder`)

```
bump_logs (id PK, guild_id, channel_id, user_id, username, bumped_at,
           reminder_sent, reminder_sent_at)
```

### 3.15 Jeux (`game_count-down` / `game_road-to-infinite`)

```
countdown_state   (channel_id PK, current_number, error_count, is_trap_active,
                   trap_number, last_user_id, updated_at)
countdown_scores  (channel_id, user_id, username, score, PK(channel_id, user_id))
counter_state     (channel_id PK, current_number, error_count, last_user_id, updated_at)
```

## 4. Tables transverses (`db/schemas/shared/`)

### 4.1 Audit & télémétrie (`shared/audit.js`)

```
user_events             (id PK, user_id, username, event_type, event_data, created_at)
form_responses          (id PK, user_id, username, form_name, responses, created_at)
discord_events_archive  (id PK, event_name, guild_id, target_id, user_id, username,
                         summary, data_json, created_at)
```

### 4.2 Cache Discord (`shared/cache.js`)

Cache synchronisé via `DiscordCacheService` :

```
server_members   (id PK, user_id UQ, username, discriminator, tag, display_name,
                  avatar_url, display_color, highest_role_id, highest_role_name,
                  highest_role_color, joined_at, account_created_at, is_bot,
                  rejoin_count, left_at, roles, presence, deleted_at,
                  created_at, updated_at)
member_history   (id PK, user_id, username, action, guild_id, metadata, created_at)
discord_channels (channel_id PK, guild_id, name, type, parent_id, position, topic,
                  is_nsfw, created_at, deleted_at, updated_at)
discord_threads  (thread_id PK, guild_id, parent_id, name, owner_id, archived,
                  locked, message_count, member_count, created_at, deleted_at, updated_at)
discord_users    (user_id PK, username, global_name, discriminator, bot, avatar_url,
                  banner_url, created_at, updated_at)
discord_messages (message_id PK, channel_id, thread_id, guild_id, author_id,
                  author_username, content, pinned, embeds_json, attachments_json,
                  reactions_json, created_at, deleted_at, updated_at)
discord_roles    (role_id PK, guild_id, name, color, color_hex, icon_url, unicode_emoji,
                  member_count, hoist, position, permissions, managed, mentionable,
                  created_at, deleted_at, updated_at)
discord_emojis   (emoji_id PK, guild_id, name, animated, url, roles_json,
                  created_at, deleted_at, updated_at)
guild_members    (id PK, user_id UQ, username, created_at)
grognement       (id PK, user_id UQ, username, created_at)
guild_stats      (guild_id, stat_key, stat_value, updated_at, PK(guild_id, stat_key))
```

### 4.3 Feature flags & config guild (`shared/feature-flags.js`)

```
guild_settings (guild_id PK, name, locale, timezone, owner_id, premium_tier,
                joined_at, created_at, updated_at)
feature_flags  (guild_id, feature_name, enabled, config_json, allowed_roles,
                updated_by, updated_at, PK(guild_id, feature_name))
```

### 4.4 Bot runtime (`shared/bot-info.js`)

```
bot_version_state (key PK, value, updated_at)
```

### 4.5 OpenAI / Daily Message (`shared/openai.js`)

```
openaimessages (id PK, msgid UQ, prompt, instruction, model, tokeninput,
                tokenoutput, content, previousmsgid, rawdata,
                created_at, updated_at)
```

## 5. Relations principales

```
guild_settings 1 ─┬─ N feature_flags
                  ├─ N user_warnings
                  ├─ N user_sanctions
                  ├─ N mod_logs
                  ├─ N tickets
                  ├─ N giveaways
                  ├─ N polls
                  ├─ N reaction_roles
                  ├─ N birthday_guild_settings
                  ├─ N user_economy
                  ├─ N shop_items
                  ├─ N temp_voice_config
                  └─ N guild_stats

tickets        1 ─── N ticket_messages
giveaways      1 ─── N giveaway_entries
polls          1 ─── N poll_votes
reports        1 ─── N report_actions
discord_threads N ── 1 discord_channels (parent_id)
```

## 6. Workflow de migration

```bash
# 1. Modifier un src/modules/<x>/db/schema.js
# 2. Générer la migration correspondante
npm run db:generate
# → produit src/db/migrations/0001_xxx.sql + maj meta/_journal.json

# 3. TOUJOURS relire le SQL généré

# 4. Appliquer en dev
npm run db:migrate

# 5. Commit schema + migration ensemble
git add src/modules/<x>/db/schema.js src/db/migrations/
git commit -m "feat(<x>): add <table> for <feature>"
```

Voir `docs/guides/migrations.md` pour la politique complète (convention `-- @down`, renommages, FK inter-modules).

## 7. Conventions Drizzle

```js
// src/modules/engagement_xp-level/db/schema.js
const { pgTable, text, integer, serial, bigint } = require('../../../db/schemas/_drizzle.js');

const userXp = pgTable('user_xp', {
    id: serial('id').primaryKey(),
    userId: text('user_id').notNull().unique(),
    username: text('username').notNull(),
    xp: integer('xp').default(0),
    level: integer('level').default(1),
    createdAt: bigint('created_at', { mode: 'number' }).notNull(),
});

module.exports = { userXp };
```

Règles :
- `pgTable` (jamais `sqliteTable`) — on est sur PostgreSQL.
- `_drizzle.js` centralise les imports `pg-core` pour éviter les oublis.
- `bigint('col', { mode: 'number' })` pour les colonnes epoch-ms (timestamps stockés en nombre).
- `text` pour les timestamps lisibles (`CURRENT_TIMESTAMP`).
- `text` pour les JSON sérialisés (`config_json`, `winners_json`, etc.).

## 8. Soft-delete

Toutes les tables cache (membres, salons, rôles, messages, threads, emojis) ont une colonne `deleted_at` (TEXT nullable, ISO 8601). Les requêtes de lecture filtrent par `deleted_at IS NULL`.

`DiscordCacheRepository.softDelete*(id)` met à jour la colonne sans supprimer la ligne, ce qui permet de garder un historique pour les logs et l'audit.

## 9. État actuel & dette technique

- ✅ Schéma Drizzle par module (19 modules, 50 tables)
- ✅ Schéma transverse dans `db/schemas/shared/` (5 fichiers de schéma, 9 repositories)
- ✅ Migrations versionnées via `drizzle-kit` (`src/db/migrations/0000_*.sql`)
- ✅ Tests passent (604/604) avec PGlite + migrator Drizzle
- ✅ **70 fonctions legacy portées nativement** dans les repositories cibles (modules + `shared/`)
- ✅ **`legacy-bridge.js` et `legacy-bridge-impl.js` supprimés** (2108 lignes effacées)
- ⚠️ Dette : la validation de l'équivalence avec la base de prod n'a pas été testée (à faire au prochain déploiement).
