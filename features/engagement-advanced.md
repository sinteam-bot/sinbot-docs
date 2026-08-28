# Feature : Engagement avancé (Phase 11)

> **Phase** : 11 — **Priorité** : 🟡 Haute — **Statut** : à implémenter
>
> Issue du backlog d'audit `docs/audit/backlog.md` (P1).

## 1. Objectif

Trois sous-features orientées "engagement utilisateur" qui s'appuient sur la même architecture de listeners :

1. **Rappels (`/remind`)** : un user peut programmer un rappel qui lui sera DM à une heure donnée
2. **Réactions de mots (auto-réponses)** : si un message contient un mot-clé, le bot répond automatiquement
3. **Commandes personnalisées (`/customcmd`)** : un admin peut définir des réponses automatiques à des commandes custom

## 2. Cas d'usage

| Cas | Comportement |
|---|---|
| **Rappels** | `/remind duration:2h message:faire la vaisselle` → DM dans 2h |
| **Liste rappels** | `/reminders` → liste des rappels actifs |
| **Annuler rappel** | `/reminder-cancel id:abc` |
| **Trigger mot-clé** | "Pizza" → bot répond "🍕" dans le salon |
| **Trigger exact** | "!ping" → bot répond "Pong!" |
| **Commande custom** | `!bienvenue` → bot répond "Bienvenue sur le serveur !" |
| **Liste custom cmds** | `/customcmd-list` |
| **Supprimer custom** | `/customcmd-remove name:bienvenue` |

## 3. Spécifications

### 3.1 Rappels

- **Durée** : supporte les formats `1s`, `5m`, `2h`, `1d` (format déjà implémenté dans `parseDuration` de `announcer.service.js`)
- **Destinataire** : DM à l'auteur de la commande (par défaut). Optionnel : mentionner un autre user via `user:@user`
- **Channel** : si mentionné dans un salon, optionnellement poster dans ce salon à la place d'un DM
- **Récurrence** : non supportée en V1 (single-shot). Le `repeat` sera en V2
- **Cooldown** : 5 secondes minimum entre deux rappels (anti-spam)

### 3.2 Réactions de mots

- **Match type** : `exact` (le message === le trigger) ou `contains` (le message contient le trigger)
- **Réponse** : texte brut OU embed JSON (réutilise le format du `templateEngine`)
- **Cooldown par trigger** : 10 secondes (anti-spam)
- **Channels exclus** : liste optionnelle de channel IDs où le trigger est actif
- **Roles exclus** : liste optionnelle de role IDs qui **ne déclenchent pas** le trigger (pour éviter le ping-pong avec d'autres bots)

### 3.3 Commandes personnalisées

- **Préfixe** : `!` (configurable, défaut = `!`)
- **Réponse** : texte brut OU embed JSON OU action spéciale (future: `addrole`, `givemoney`)
- **Cooldown par cmd** : 5 secondes
- **Channel restrict** : liste optionnelle de channel IDs où la cmd est active
- **Role restrict** : liste optionnelle de role IDs requis pour utiliser la cmd

## 4. Architecture

```
feature_engagement-advanced/
├── config/defaults.js
├── db/schema.js (Drizzle — couvert par src/db/schema/pg.js)
├── services/
│   ├── engagement.repository.js
│   ├── reminder.service.js
│   ├── word-trigger.service.js
│   └── custom-command.service.js
├── events/
│   └── message-create.listener.js
├── cron/
│   └── reminder-cron.js
├── commands/engagement-commands.js
├── controllers/engagement.controller.js
└── tests/engagement-services.test.js
```

## 5. Base de données

### `reminders` (Phase 11.1)

| Colonne | Type | Description |
|---|---|---|
| `id` | TEXT PK | UUID v4 |
| `guild_id` | TEXT | nullable (null pour DM) |
| `channel_id` | TEXT | nullable (null pour DM) |
| `user_id` | TEXT NOT NULL | destinataire |
| `reminder_text` | TEXT NOT NULL | message du rappel |
| `fire_at` | BIGINT NOT NULL | epoch ms |
| `created_at` | BIGINT NOT NULL | epoch ms |
| `status` | TEXT | `pending` (default) / `done` / `cancelled` |
| `source_message_id` | TEXT | lien vers le message source (si salon) |

Index : `(status, fire_at)` pour le cron de scan.

### `word_triggers` (Phase 11.2)

| Colonne | Type | Description |
|---|---|---|
| `id` | TEXT PK | UUID v4 |
| `guild_id` | TEXT NOT NULL | |
| `trigger_text` | TEXT NOT NULL | mot-clé |
| `match_type` | TEXT | `exact` (default) / `contains` / `regex` |
| `response_text` | TEXT nullable | |
| `response_embed_json` | TEXT nullable | |
| `exclude_channel_ids_json` | TEXT | array JSON de channel IDs |
| `exclude_role_ids_json` | TEXT | array JSON de role IDs |
| `cooldown_seconds` | INTEGER | default 10 |
| `created_by` | TEXT | admin user_id |
| `created_at` | BIGINT NOT NULL | |
| `updated_at` | BIGINT NOT NULL | |

Index : `(guild_id)` pour la liste, `(guild_id, trigger_text)` pour le lookup.

### `custom_commands` (Phase 11.3)

| Colonne | Type | Description |
|---|---|---|
| `id` | TEXT PK | UUID v4 |
| `guild_id` | TEXT NOT NULL | |
| `name` | TEXT NOT NULL | nom de la commande (sans préfixe) |
| `response_text` | TEXT nullable | |
| `response_embed_json` | TEXT nullable | |
| `restrict_channel_ids_json` | TEXT | array JSON |
| `restrict_role_ids_json` | TEXT | array JSON |
| `cooldown_seconds` | INTEGER | default 5 |
| `created_by` | TEXT | |
| `created_at` | BIGINT NOT NULL | |
| `updated_at` | BIGINT NOT NULL | |

UNIQUE(guild_id, name) — pas deux commandes du même nom.
Index : `(guild_id)`.

## 6. API publique des services

### ReminderService

- `create({guildId, channelId, userId, text, fireAt})` -> `{ok, id, fireAt}`
- `list(userId, limit)` : rappels actifs d'un user
- `cancel(id, userId)` : annule un rappel (owner check)
- `get(id)`
- `tick(now)` : appelé par le cron, retourne les rappels到期 et les marque done
- `dispatch(reminder, client)` : envoie le DM ou le message

### WordTriggerService

- `create({guildId, triggerText, matchType, responseText, responseEmbed, excludeChannels, excludeRoles, cooldown})` -> `{ok, id}`
- `list(guildId, limit)`
- `findMatching(guildId, content)` : retourne le premier trigger qui matche
- `shouldFire(trigger, message, member)` : check cooldown, channels exclus, roles exclus
- `incrementCooldown(triggerId)`
- `delete(id)`

### CustomCommandService

- `create({guildId, name, responseText, responseEmbed, restrictChannels, restrictRoles, cooldown})`
- `list(guildId, limit)`
- `find(guildId, name)` : lookup
- `canRun(command, member)` : check cooldowns + restrict
- `incrementCooldown(commandId)`
- `delete(id)`

## 7. REST API

```
GET    /api/engagement-advanced/reminders?user_id=&guild_id=
POST   /api/engagement-advanced/reminders
DELETE /api/engagement-advanced/reminders/:id

GET    /api/engagement-advanced/triggers?guild_id=
POST   /api/engagement-advanced/triggers
DELETE /api/engagement-advanced/triggers/:id

GET    /api/engagement-advanced/commands?guild_id=
POST   /api/engagement-advanced/commands
DELETE /api/engagement-advanced/commands/:id
```

## 8. Slash commands

**Rappels** :
- `/remind duration:2h message:...` — programme un rappel DM
- `/reminders` — liste des rappels actifs
- `/reminder-cancel id:...` — annule un rappel

**Triggers (admin)** :
- `/trigger-add trigger:... response:... [match:contains]` — ajoute un trigger
- `/trigger-list` — liste
- `/trigger-remove id:...` — supprime

**Commandes custom (admin)** :
- `/customcmd-add name:... response:...`
- `/customcmd-list`
- `/customcmd-remove name:...`

## 9. Cron

`@Cron('* * * * *', Europe/Paris)` : `ReminderService.tick()` pour scanner et dispatcher les rappels到期.

## 10. Critères d'acceptation (DoD)

- [ ] Tables créées idempotemment (3 tables)
- [ ] Rappels : create / list / cancel / dispatch via DM
- [ ] Triggers : match exact + contains, cooldown, channels/roles exclus
- [ ] Custom commands : préfixe !, lookup, cooldowns
- [ ] Frontend `/modules/engagement-advanced` avec les 3 onglets
- [ ] Tests unitaires ≥ 20
- [ ] Aucune régression

## 11. Tests

- ReminderService : create/list/cancel/dispatch
- WordTriggerService : match exact/contains, shouldFire (cooldown, excludes)
- CustomCommandService : create/list/find/canRun
- Integration : create trigger + send message → bot replies
