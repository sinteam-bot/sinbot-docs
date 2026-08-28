# Feature : Salons vocaux temporaires (Join-to-Create)

> **Phase** : 12b — **Priorité** : 🟡 Haute — **Statut** : à implémenter (P2 du backlog d'audit)

## 1. Objectif

Permettre à un user de **rejoindre un salon "Join-to-Create"** → le bot crée automatiquement un salon vocal privé éphémère. Le salon est **supprimé** quand il devient vide (avec un petit délai pour éviter les suppressions pendant un move rapide).

## 2. Cas d'usage

- **Gaming** : "Join-to-Create" → "John's game", "Mary's game", etc.
- **Voice channels dynamiques** : plus besoin de créer N salons manuellement
- **Vie privée** : un user entre, parle, sort → le salon disparaît
- **Rename auto** : "John's game" quand un user rejoint, "John's game 🎮" si plus d'un

## 3. Spécifications

### 3.1 Configuration

```yaml
features:
  temp_voice:
    enabled: false
    allowed_roles: []              # Discord roles autorisés à override les limites
    join_channels: []              # Channel IDs qui agissent comme Join-to-Create
    category_id: null              # Catégorie dans laquelle créer les vocaux temporaires
    format: "{user}'s game"         # Template du nom ({user} ou {username} = displayName)
    delete_delay_seconds: 5         # Délai avant suppression d'un vocal vide
    max_per_guild: 0               # Limite dure (0 = illimité)
    locked_role_id: null           # Si défini, ce role est ajouté au vocal à la création
```

### 3.2 Cycle de vie d'un salon temporaire

1. **Création** : un user rejoint un `join_channel` → le bot crée un nouveau `GuildVoiceChannel`
   dans `category_id` (ou à la racine), nommé d'après `format` (par défaut `"{user}'s game"`)
2. **Move** : le bot déplace l'user dans le nouveau salon
3. **Rename** : si 2+ users, suffix ajouté (`{user}'s game 🎮`)
4. **Empty** : quand le dernier user quitte, le bot attend `delete_delay_seconds` puis supprime
5. **Persistence** : aucun salon vide n'est supprimé si la limite `max_per_guild` est dépassée (safety)

### 3.3 Permissions

Le salon créé hérite des permissions du `join_channel` source **mais override** :
- `@everyone` : Connect (true) + View Channel (true) + Speak (true)
- Le créateur : Manage Channels (true) + Mute Members (true) + Move Members (true)
- `locked_role_id` si défini : ce role est ajouté au salon à la création

## 4. Architecture

```
feature_temp-voice/
├── config/defaults.js
├── services/
│   ├── temp-voice.repository.js
│   └── temp-voice.service.js
├── events/
│   └── temp-voice-listener.js
├── cron/
│   └── temp-voice-cleanup.js
├── commands/temp-voice-commands.js
├── controllers/temp-voice.controller.js
└── tests/temp-voice-service.test.js
```

## 5. Base de données (2 tables)

### `temp_voice_config`

| Colonne | Type | Description |
|---|---|---|
| `guild_id` | TEXT PK | Discord snowflake |
| `category_id` | TEXT | Catégorie parente |
| `format` | TEXT | Template du nom |
| `delete_delay_seconds` | INTEGER | Délai avant suppression |
| `max_per_guild` | INTEGER | Limite dure (0 = illimité) |
| `locked_role_id` | TEXT | Role optionnel à ajouter |
| `enabled` | INTEGER | 0/1 |
| `updated_at` | INTEGER | epoch ms |

### `temp_voice_state`

| Colonne | Type | Description |
|---|---|---|
| `channel_id` | TEXT PK | Le salon créé par le bot |
| `guild_id` | TEXT NOT NULL | Discord snowflake |
| `creator_id` | TEXT | Créateur initial |
| `last_empty_at` | INTEGER | Quand le salon est devenu vide (0 si pas vide) |
| `created_at` | INTEGER NOT NULL | epoch ms |

Index : `(guild_id)` pour lister tous les vocaux temporaires d'un serveur.

## 6. API publique

### `TempVoiceService`

- `getConfig(guildId)` : retourne la config ou les défauts
- `setConfig(guildId, patch)` : upsert
- `isJoinChannel(channelId, config)` : true si channelId est dans `config.join_channels`
- `createForUser(guild, user, config)` : crée le salon, déplace l'user
- `renameIfMultiUser(channel, count, config)` : ajoute un suffix si ≥ 2 users
- `scheduleDeletion(channelId, delaySec)` : marque `last_empty_at` et programme
- `cleanup()` (cron) : supprime les vocaux `last_empty_at + delay < now`
- `getActiveChannels(guildId)` : liste les vocaux temporaires actifs
- `getCount(guildId)` : nb de vocaux temporaires actifs

### `TempVoiceListener`

- `voiceStateUpdate` (Discord) :
  - si user rejoint un `join_channel` → crée un vocal
  - si user quitte un vocal temporaire → schedule deletion
- `channelDelete` (cleanup safety net)

## 7. REST API

```
GET   /api/temp-voice/config?guild_id=
PATCH /api/temp-voice/config  body { guildId, categoryId?, format?, deleteDelaySeconds?, maxPerGuild?, lockedRoleId?, joinChannels?, enabled? }
GET   /api/temp-voice/active?guild_id=
GET   /api/temp-voice/count?guild_id=
```

## 8. Slash commands (admin, ManageGuild)

- `/tempvoice-config show` : affiche la config actuelle
- `/tempvoice-config set category:#cat name:Template delay:5` : met à jour
- `/tempvoice-config add-channel #salon` : ajoute un Join-to-Create
- `/tempvoice-config remove-channel #salon`
- `/tempvoice-config toggle enabled:true|false`
- `/tempvoice-list` : liste les vocaux temporaires actifs
- `/tempvoice-lock @user` : lock un user (toggle role, optionnel)
- `/tempvoice-unlock @user`

## 9. Événements Discord écoutés

- `voiceStateUpdate` : create / rename / schedule deletion
- `channelDelete` : cleanup du state

## 10. Critères d'acceptation (DoD)

- [x] 2 tables : `temp_voice_config` + `temp_voice_state`
- [x] `voiceStateUpdate` handler : create on join, schedule deletion on leave
- [x] Rename auto si 2+ users
- [x] Cleanup cron qui supprime les vocaux vides après `delete_delay_seconds`
- [x] Tests unitaires ≥ 10
- [x] Page dashboard `/modules/temp-voice`

## 11. Tests

- getConfig / setConfig
- isJoinChannel (true / false)
- createForUser : crée un salon, déplace l'user
- scheduleDeletion : marque `last_empty_at`
- cleanup : supprime les过期
- renameIfMultiUser : ajoute un suffix si ≥ 2
- Anti-spam : max_per_guild empêche la création si dépassé
