# Référence API REST

> Base URL : `https://bot.example.com`
>
> Authentification : header `x-api-key: <key>` ou `?api_key=<key>` (cf. `config.yml > web.auth`)

## 1. Santé

### `GET /healthz`

```json
{ "status": "ok", "db": true, "uptime": 12345 }
```

## 2. Features

### `GET /api/features`

Liste toutes les features enregistrées avec leur état pour la guild courante.

**Query** : `guild_id` (optionnel, défaut : `process.env.DISCORD_GUILD_ID`)

**Réponse 200** :

```json
[
  {
    "name": "automod",
    "defaults": { "enabled": false, "spam": { "max_messages": 5 } },
    "state": {
      "enabled": true,
      "config": { "spam": { "max_messages": 8 } },
      "allowedRoles": ["123456789"],
      "source": "db"
    }
  },
  {
    "name": "xp",
    "defaults": { "enabled": false },
    "state": { "enabled": false, "config": {}, "allowedRoles": [], "source": "default" }
  }
]
```

### `GET /api/features/:name`

Détail d'une feature.

### `PATCH /api/features/:name`

Modifie l'état d'une feature.

**Body** :

```json
{
  "guildId": "123456789",
  "enabled": true,
  "config": { "spam": { "max_messages": 10 } },
  "allowedRoles": ["MOD_ROLE_ID"],
  "userId": "ADMIN_USER_ID"
}
```

**Réponse 200** : `{ "ok": true }`

### `POST /api/features/:name/can-use`

Vérifie si un utilisateur peut utiliser la feature.

**Body** :

```json
{ "guildId": "123456789", "userId": "USER_ID" }
```

**Réponse 200** :

```json
{ "allowed": true, "reason": "role_match" }
```

Raisons possibles : `disabled`, `no_restriction`, `role_match`, `missing_role`, `not_member`.

## 3. Modération (Phase 1)

### `GET /api/mod/logs`

Liste paginée des logs de modération.

**Query** :
- `guild_id` (requis)
- `user_id` (optionnel)
- `action` (optionnel : `warn`, `mute`, `kick`, `ban`, `clear`…)
- `page` (défaut 1)
- `limit` (défaut 25, max 100)

**Réponse 200** :

```json
{
  "logs": [
    {
      "id": "uuid",
      "user_id": "...",
      "mod_id": "...",
      "action": "warn",
      "reason": "spam",
      "source": "automod",
      "created_at": 1693142400000
    }
  ],
  "page": 1,
  "total": 142,
  "pages": 6
}
```

### `GET /api/mod/users/:id`

Profil modération d'un utilisateur (warns actifs, historique, sanctions).

### `POST /api/mod/sanctions`

Applique une sanction manuellement (côté dashboard).

**Body** :

```json
{
  "guild_id": "123456789",
  "user_id": "...",
  "type": "warn",  // 'warn' | 'mute' | 'kick' | 'ban'
  "reason": "Spam dans #général",
  "duration_ms": 3600000,  // optionnel, pour mute
  "mod_id": "ADMIN_USER_ID"
}
```

### `POST /api/mod/sanctions/preview`

Simule l'escalade automatique : retourne la sanction qui serait appliquée avec le nombre de warns actuel.

### `POST /api/features/automod/test`

Teste une règle sur un message factice.

**Body** :

```json
{
  "guild_id": "123456789",
  "content": "VISITEZ discord.gg/spam-maintenant",
  "rules": ["badwords", "anti_invite"]
}
```

**Réponse 200** :

```json
{
  "rules_triggered": ["anti_invite"],
  "action": "delete_warn"
}
```

## 4. XP & Niveaux (Phase 2)

### `GET /api/xp/leaderboard?page=1`

```json
{
  "users": [
    { "user_id": "...", "xp_total": 5400, "level": 12, "rank": 1 }
  ],
  "page": 1,
  "pages": 10
}
```

### `GET /api/xp/user/:id`

### `GET /api/xp/config`

### `PATCH /api/xp/config`

### `POST /api/xp/adjust`

```json
{ "guild_id": "...", "user_id": "...", "delta": 100, "reason": "event" }
```

## 5. Tickets (Phase 3)

### `GET /api/tickets?status=open&page=1`

### `GET /api/tickets/:id`

### `GET /api/tickets/:id/transcript`

### `POST /api/tickets/:id/close`

## 6. Logs & Stats (Phase 4)

### `GET /api/logs?type=message_delete&page=1&limit=50`

### `GET /api/stats/overview`

```json
{
  "members": 1234,
  "messages_24h": 5678,
  "warnings_24h": 12,
  "active_users_7d": 234,
  "boost_count": 5,
  "boost_tier": 2
}
```

### `GET /api/stats/messages?days=7`

### `GET /api/stats/members/growth?days=30`

### `GET /api/stats/moderation?weeks=4`

### `WS /ws/logs`

WebSocket pour le live feed. Messages :

```json
{ "ts": 1693142400000, "type": "message_delete", "channel": "...", "user": "...", "data": {...} }
```

## 7. Giveaways & Polls (Phase 5)

### `GET /api/giveaways?status=active`

### `GET /api/polls?status=active`

### `POST /api/polls/:id/vote`

## 8. Erreurs

Format uniforme :

```json
{ "success": false, "error": "Message d'erreur lisible" }
```

| Code | Signification |
|---|---|
| 400 | Paramètres invalides |
| 401 | API key manquante ou invalide |
| 403 | IP non autorisée |
| 404 | Ressource inexistante |
| 429 | Rate limit |
| 500 | Erreur serveur |

## 9. Rate limiting

- Par IP : 60 req/min par défaut
- Par API key : 600 req/min
- Headers : `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
