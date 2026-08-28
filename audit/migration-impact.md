# Impact des features à ajouter

> Ce document détaille, pour chaque feature Draftbot manquante, l'impact sur :
> - Le schéma DB (tables à ajouter)
> - L'API REST (endpoints à exposer)
> - L'architecture (services / events / commands / tests)

## Légende

- 🟢 **Faible** : ≤ 1 table ou ≤ 2 endpoints
- 🟡 **Moyen** : 2-3 tables ou 3-5 endpoints
- 🔴 **Élevé** : > 3 tables ou > 5 endpoints, ou feature transverse

---

## Phase 9 — Économie & Inventaire 🔴

> **Prérequis** : aucun (première brique du système économique)

### Économie

**DB** (1 table) :
```sql
CREATE TABLE user_economy (
    user_id TEXT NOT NULL,
    guild_id TEXT NOT NULL,
    balance INTEGER NOT NULL DEFAULT 0,
    bank_balance INTEGER NOT NULL DEFAULT 0,    -- optionnel
    last_daily_claim_at INTEGER,
    total_earned INTEGER NOT NULL DEFAULT 0,
    total_spent INTEGER NOT NULL DEFAULT 0,
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL,
    PRIMARY KEY (user_id, guild_id)
);
CREATE TABLE economy_transactions (
    id TEXT PRIMARY KEY,
    user_id TEXT NOT NULL,
    guild_id TEXT NOT NULL,
    amount INTEGER NOT NULL,
    type TEXT NOT NULL,                          -- 'daily', 'pay', 'shop', 'gift', 'admin'
    counterparty_id TEXT,                        -- pour /pay
    reason TEXT,
    metadata TEXT,
    created_at INTEGER NOT NULL
);
```

**REST** (6 endpoints) :
- `GET /api/economy/balance/:userId`
- `POST /api/economy/daily` (claim journalier)
- `POST /api/economy/pay` (transfert entre users)
- `GET /api/economy/transactions/:userId`
- `POST /api/economy/admin/grant` (admin)
- `GET /api/economy/leaderboard`

**Slash commands** (6) :
- `/balance [user]`
- `/daily`
- `/pay user:@user amount:100`
- `/leaderboard`
- `/eco-admin grant user amount`
- `/eco-admin reset user`

### Inventaire (dépend de l'économie)

**DB** (4 tables) :
```sql
CREATE TABLE shop_items (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    name TEXT NOT NULL,
    description TEXT,
    emoji TEXT,
    price INTEGER NOT NULL,
    role_reward_id TEXT,           -- gift rôle (anniv)
    xp_reward INTEGER,              -- gift XP (anniv)
    is_tradeable INTEGER NOT NULL DEFAULT 1,
    is_droppable INTEGER NOT NULL DEFAULT 1,
    max_per_user INTEGER,           -- stock par user
    created_at INTEGER NOT NULL
);

CREATE TABLE user_inventory (
    user_id TEXT NOT NULL,
    guild_id TEXT NOT NULL,
    item_id TEXT NOT NULL,
    quantity INTEGER NOT NULL DEFAULT 1,
    acquired_at INTEGER NOT NULL,
    PRIMARY KEY (user_id, guild_id, item_id)
);

CREATE TABLE inventory_drops (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    channel_id TEXT NOT NULL,
    message_id TEXT,
    item_id TEXT NOT NULL,
    quantity INTEGER NOT NULL DEFAULT 1,
    started_at INTEGER NOT NULL,
    expires_at INTEGER NOT NULL,
    claimed_by TEXT,
    status TEXT NOT NULL DEFAULT 'active'   -- 'active' | 'claimed' | 'expired'
);

CREATE TABLE inventory_transfers (
    id TEXT PRIMARY KEY,
    from_user_id TEXT NOT NULL,
    to_user_id TEXT NOT NULL,
    item_id TEXT NOT NULL,
    quantity INTEGER NOT NULL DEFAULT 1,
    type TEXT NOT NULL,                          -- 'give' | 'sell' | 'trade'
    price INTEGER,                              -- si 'sell'
    created_at INTEGER NOT NULL
);
```

**REST** (10 endpoints) :
- `GET /api/inventory/:userId`
- `GET /api/items`
- `POST /api/items` (admin create)
- `PATCH /api/items/:id` (admin)
- `DELETE /api/items/:id` (admin)
- `POST /api/inventory/give` (admin)
- `POST /api/inventory/remove` (admin)
- `POST /api/inventory/reset` (admin)
- `POST /api/inventory/transfer` (give/sell/trade)
- `POST /api/inventory/drop` (start drop)
- `POST /api/inventory/drop/:id/claim`
- `GET /api/inventory/topitems`
- `POST /api/inventory/merge` (admin)
- `POST /api/inventory/rename` (admin)

**Slash commands** (15+) :
- `/inventaire [user]`
- `/shop`
- `/shop buy item:Name`
- `/objet donner user:@user item:Name [qty:1]`
- `/objet vendre user:@user item:Name [price:100]`
- `/objet échanger user:@user item:Name`
- `/objet drop item:Name [duration:1m] [channel:#salon]`
- `/dropobjet nom item:Name [duration] [channel]`
- `/topitems`
- `/admininventaire ajouter/retirer/reset/renommer/fusionner/transferer`

**Listener** : `messageReactionAdd` pour le claim des drops.

---

## Phase 10 — Rôles-Réactions 🟡

> **Prérequis** : aucun

**DB** (1 table) :
```sql
CREATE TABLE reaction_roles (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    channel_id TEXT NOT NULL,
    message_id TEXT NOT NULL,
    emoji TEXT NOT NULL,                    -- unicode ou custom ID
    role_id TEXT NOT NULL,
    description TEXT,
    created_at INTEGER NOT NULL,
    UNIQUE(message_id, emoji)
);
```

**REST** (5) :
- `GET /api/reaction-roles?channel_id=&message_id=`
- `POST /api/reaction-roles`
- `PATCH /api/reaction-roles/:id`
- `DELETE /api/reaction-roles/:id`
- `POST /api/reaction-roles/sync/:id` (re-post le message)

**Slash commands** (3) :
- `/reactionrole create channel message emoji role`
- `/reactionrole list [message]`
- `/reactionrole delete id`

**Listeners** : `messageReactionAdd` + `messageReactionRemove` → `member.roles.add/remove`.

---

## Phase 8 — Quick wins UX 🟢

### Commandes d'informations (3 endpoints + 5 commands)

- `GET /api/info/server?guild_id=` → `/serverinfo`
- `GET /api/info/user/:userId?guild_id=` → `/userinfo`
- `GET /api/info/avatar/:userId` → `/avatar`

### Sticky roles (1 table + 1 listener)

```sql
CREATE TABLE sticky_roles (
    user_id TEXT NOT NULL,
    guild_id TEXT NOT NULL,
    role_id TEXT NOT NULL,
    PRIMARY KEY (user_id, guild_id, role_id)
);
```

Listeners : `guildMemberAdd` (re-attribue), `guildMemberRemove` (sauvegarde).

REST : 3 endpoints (get / set / remove).
Slash : `/stickyrole add/remove/list`.

### Stats de jeux (extension des endpoints existants)

Le contrôleur `feature_engagement` a déjà `findGiveaway` etc. Il faut ajouter :
- `GET /api/games/counter/leaderboard?guild_id=`
- `GET /api/games/countdown/leaderboard?guild_id=`
- `GET /api/games/stats?guild_id=` (agrégat : parties jouées, top scores, etc.)

Pas de nouvelle table — les données existent déjà dans `counterState` + `countdownScores`.

---

## Phase 11 — Engagement avancé 🟡

### Réactions de mots (1 table + listener)

```sql
CREATE TABLE word_triggers (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    trigger_text TEXT NOT NULL,
    match_type TEXT NOT NULL DEFAULT 'contains',  -- 'exact' | 'contains' | 'regex'
    response_text TEXT,
    response_embed_json TEXT,
    is_staff_only INTEGER NOT NULL DEFAULT 0,
    created_at INTEGER NOT NULL
);
```

Listener : `messageCreate` qui match les triggers et répond.

REST : 5 endpoints (CRUD).
Slash : `/trigger add/list/remove`.

### Rappels (1 table + 1 cron)

```sql
CREATE TABLE reminders (
    id TEXT PRIMARY KEY,
    user_id TEXT NOT NULL,
    guild_id TEXT,
    channel_id TEXT,
    message TEXT NOT NULL,
    fire_at INTEGER NOT NULL,
    status TEXT NOT NULL DEFAULT 'pending',   -- 'pending' | 'done' | 'cancelled'
    created_at INTEGER NOT NULL
);
CREATE INDEX idx_reminders_fire ON reminders(fire_at, status);
```

Cron : `* * * * *` qui scanne les reminders到期 et envoie en DM.

REST : 5 endpoints.
Slash : `/remind duration message`, `/reminders list`, `/reminder cancel id`.

### Commandes personnalisées (1 table + listener)

```sql
CREATE TABLE custom_commands (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    name TEXT NOT NULL,
    response_text TEXT,
    response_embed_json TEXT,
    is_embed INTEGER NOT NULL DEFAULT 0,
    created_by TEXT,
    created_at INTEGER NOT NULL,
    UNIQUE(guild_id, name)
);
```

Listener : `messageCreate` qui match `/<name>` et répond avec la config stockée.

REST : 5 endpoints (CRUD).
Slash : `/customcmd add/list/remove/show`.

---

## Phase 12 — Modération communautaire 🟡

### Starboards (1 table + listener)

```sql
CREATE TABLE starboard_entries (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    source_channel_id TEXT NOT NULL,
    source_message_id TEXT NOT NULL,
    starboard_message_id TEXT,
    author_id TEXT NOT NULL,
    reaction_count INTEGER NOT NULL DEFAULT 0,
    created_at INTEGER NOT NULL,
    UNIQUE(guild_id, source_message_id)
);
```

Listener : `messageReactionAdd` (⭐) → vérifie seuil → post dans le salon starboard.

REST : 4 endpoints.
Slash : `/starboard setup/threshold/blacklist/list`.

### Signalements (2 tables + button + cron)

```sql
CREATE TABLE reports (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    reporter_id TEXT NOT NULL,
    reported_user_id TEXT NOT NULL,
    channel_id TEXT,
    message_id TEXT,
    reason TEXT,
    status TEXT NOT NULL DEFAULT 'open',    -- 'open' | 'resolved' | 'dismissed'
    resolved_by TEXT,
    resolved_at INTEGER,
    created_at INTEGER NOT NULL
);

CREATE TABLE report_actions (
    id TEXT PRIMARY KEY,
    report_id TEXT NOT NULL,
    staff_id TEXT NOT NULL,
    action TEXT NOT NULL,                     -- 'warn' | 'mute' | 'kick' | 'ban' | 'dismiss'
    notes TEXT,
    created_at INTEGER NOT NULL
);
```

Bouton "Signaler" sur les messages, modal pour la raison, queue côté staff.

REST : 5 endpoints.
Slash : `/report user reason`, `/reports list/resolve/dismiss`.

### Salons vocaux temporaires (0 table, beaucoup de logique)

Utilise la table `voiceStateUpdate` listener :
- Si user rejoint "Join to Create" → `guild.channels.create({type: Voice})` + permissions
- Si salon devient vide → `channel.delete()`

**Config** : table `temp_voice_config` (1 table simple).

REST : 3 endpoints (config + on/off).
Slash : `/tempvoice setup/on/off`.

### Sauvegardes serveur (0 table, snapshot JSON)

Pas de table — snapshot JSON du serveur (rôles, salons, perms) stocké sur disque ou dans `bot_state` (clé/valeur).

REST : 3 endpoints (create/restore/list).
Slash : `/backup create/restore`.

---

## Phase 13 — Jeux additionnels 🟡

### Calendrier de l'Avent (1 table + cron)

```sql
CREATE TABLE advent_calendar (
    guild_id TEXT NOT NULL,
    day INTEGER NOT NULL,                    -- 1..24
    reward_text TEXT,
    reward_role_id TEXT,
    reward_xp INTEGER,
    claimed_by TEXT,                          -- user_id
    claimed_at INTEGER,
    PRIMARY KEY (guild_id, day)
);
```

Cron : `0 0 * 12 * *` (1er décembre) + check quotidien.

### Bingo (2 tables + commands)

```sql
CREATE TABLE bingo_games (
    id TEXT PRIMARY KEY,
    guild_id TEXT NOT NULL,
    channel_id TEXT NOT NULL,
    host_id TEXT NOT NULL,
    pattern TEXT NOT NULL,        -- JSON 5x5 grid
    called_numbers TEXT NOT NULL,  -- JSON array
    started_at INTEGER,
    ended_at INTEGER,
    winner_id TEXT,
    status TEXT NOT NULL DEFAULT 'setup'
);

CREATE TABLE bingo_players (
    game_id TEXT NOT NULL,
    user_id TEXT NOT NULL,
    card_json TEXT NOT NULL,
    marked_json TEXT NOT NULL DEFAULT '[]',
    joined_at INTEGER NOT NULL,
    PRIMARY KEY (game_id, user_id)
);
```

Slash : `/bingo create/start/join/mark/win/cancel`.

### Commandes fun (1 table optionnelle pour stats)

```sql
CREATE TABLE fun_stats (
    user_id TEXT NOT NULL,
    guild_id TEXT NOT NULL,
    command TEXT NOT NULL,         -- 'roll', 'coinflip', etc.
    count INTEGER NOT NULL DEFAULT 0,
    last_used_at INTEGER,
    PRIMARY KEY (user_id, guild_id, command)
);
```

Slash : `/roll`, `/coinflip`, `/8ball`, `/rps`, `/trivia`. Effort faible par commande (1 service par commande).

---

## Synthèse : impact global

| Phase | Nouvelles tables | Nouveaux endpoints | Slash commands | Effort |
|---|---:|---:|---:|---|
| 8 (quick wins) | 1 | 5 | 5+ | 🟢 ≈ 1 sem |
| 9 (économie) | 5 | 25+ | 20+ | 🔴 ≈ 3-4 sem |
| 10 (réaction roles) | 1 | 5 | 3 | 🟡 ≈ 1 sem |
| 11 (engagement avancé) | 3 | 15 | 10+ | 🟡 ≈ 2 sem |
| 12 (modération communauté) | 5 | 15 | 10+ | 🟡 ≈ 2 sem |
| 13 (jeux) | 3 | 5 | 5+ | 🟡 ≈ 1 sem |
| **Total** | **18** | **70+** | **53+** | **≈ 10-11 sem** |

**Note** : la phase 9 (Économie + Inventaire) représente **40% de l'effort total**. À elle seule elle débloque :
- Le cadeau "argent" dans `feature_birthdays` (placeholder actuel)
- Le cadeau "objet inventaire" dans `feature_birthdays`
- Les drops d'objets dans `feature_engagement` (giveaways)

C'est pour ça qu'elle est la **priorité #1** de la roadmap.

## Voir aussi

- [`audit-draftbot.md`](./audit-draftbot.md) — analyse stratégique + roadmap ordonnée
- [`draftbot-feature-list.md`](./draftbot-feature-list.md) — table exhaustive avec liens
