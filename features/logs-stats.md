# Feature : Logs détaillés & Dashboard de statistiques

> **Phase** : 4 — **Priorité** : 🟡 Moyenne — **Statut** : à implémenter

## 1. Objectif

Centraliser tous les événements importants du serveur dans des salons de logs configurables, et exposer un dashboard temps réel de statistiques.

## 2. Cas d'usage

- **Logs de modération** : warns, mutes, kicks, bans, deletes
- **Logs de messages** : suppressions, éditions, bulk delete
- **Logs de membres** : join, leave, kick, ban, role add/remove
- **Logs vocaux** : join/leave/switch de channel
- **Logs de rôles** : création, édition, suppression
- **Logs de salons** : création, édition, suppression
- **Logs d'émojis** : ajout, suppression
- **Logs serveur** : boost start/end, vanity, level up

## 3. Spécifications

### 3.1 Configuration

```yaml
features:
  logs:
    enabled: false
    allowed_roles: ["ADMIN_ROLE_ID"]
    channels:
      moderation: "MOD_LOG_CHANNEL_ID"
      messages: "MESSAGE_LOG_CHANNEL_ID"
      members: "MEMBER_LOG_CHANNEL_ID"
      voice: "VOICE_LOG_CHANNEL_ID"
      roles: "ROLE_LOG_CHANNEL_ID"
      channels: "CHANNEL_LOG_CHANNEL_ID"
      server: "SERVER_LOG_CHANNEL_ID"
    events:
      message_delete: true
      message_edit: true
      message_bulk_delete: true
      member_join: true
      member_leave: true
      member_kick: false  # déduit, pas d'event natif
      member_ban_add: true
      member_ban_remove: true
      member_update: false # peut être très bruyant (chgmt pseudo…)
      role_create: true
      role_update: true
      role_delete: true
      channel_create: true
      channel_update: true
      channel_delete: true
      voice_state_update: true
      guild_update: true
      emoji_create: true
      emoji_delete: true
    format: "embed"   # embed | simple
    color: "#2F3136"
    ignored_channels: []  # channel IDs où on ne log PAS
    ignored_users: []     # user IDs à ignorer (bots, owners)
```

### 3.2 Sanitization

Les messages loggés peuvent contenir des spoils, des images NSFW, des liens malveillants. Le service doit :

- Tronquer à 1024 caractères (limite embed)
- Remplacer les URLs par `<lien>` sauf si `whitelist_domains`
- Remplacer les IDs par des mentions `<@id>` quand possible
- Pour les images jointes, n'envoyer que le nom et l'URL masquée
- **Ne jamais re-poster le contenu intégral d'un message NSFW**

## 4. Architecture

```
src/modules/security_logs/
├── logs.module.js
├── config/schema.js
├── services/
│   ├── logs.service.js                 # dispatcher
│   ├── embed-builder.service.js        # helpers de formatage
│   ├── message-logs.service.js
│   ├── member-logs.service.js
│   ├── role-logs.service.js
│   ├── channel-logs.service.js
│   ├── voice-logs.service.js
│   └── server-logs.service.js
├── events/
│   ├── message-delete.listener.js
│   ├── message-update.listener.js
│   ├── message-bulk-delete.listener.js
│   ├── guild-member-add.listener.js
│   ├── guild-member-remove.listener.js
│   ├── guild-member-update.listener.js
│   ├── guild-ban-add.listener.js
│   ├── guild-ban-remove.listener.js
│   ├── guild-role-create.listener.js
│   ├── guild-role-update.listener.js
│   ├── guild-role-delete.listener.js
│   ├── channel-create.listener.js
│   ├── channel-update.listener.js
│   ├── channel-delete.listener.js
│   ├── voice-state-update.listener.js
│   ├── guild-update.listener.js
│   ├── emoji-create.listener.js
│   └── emoji-delete.listener.js
├── controllers/
│   └── logs.controller.js
└── tests/
    ├── embed-builder.test.js
    ├── sanitization.test.js
    └── routing.test.js
```

## 5. Événements à écouter

Mapping entre les events Discord.js et les channels de log :

| Discord event | Channel log | Embed title |
|---|---|---|
| `messageDelete` | `messages` | 🗑️ Message supprimé |
| `messageUpdate` | `messages` | ✏️ Message modifié |
| `messageDeleteBulk` | `messages` | 🗑️ Suppression en masse |
| `guildMemberAdd` | `members` | ➕ Nouveau membre |
| `guildMemberRemove` | `members` | ➖ Membre parti |
| `guildMemberUpdate` | `members` | 🔄 Membre modifié |
| `guildBanAdd` | `members` | 🔨 Membre banni |
| `guildBanRemove` | `members` | 🔓 Membre débanni |
| `roleCreate` | `roles` | ➕ Rôle créé |
| `roleUpdate` | `roles` | ✏️ Rôle modifié |
| `roleDelete` | `roles` | ➖ Rôle supprimé |
| `channelCreate` | `channels` | ➕ Salon créé |
| `channelUpdate` | `channels` | ✏️ Salon modifié |
| `channelDelete` | `channels` | ➖ Salon supprimé |
| `voiceStateUpdate` | `voice` | 🔊 Vocal |
| `guildUpdate` | `server` | ⚙️ Serveur modifié |
| `emojiCreate` | `server` | 😀 Emoji ajouté |
| `emojiDelete` | `server` | 🗑️ Emoji supprimé |

## 6. WebSocket temps réel

Pour le dashboard, on expose un endpoint WS :

```js
// src/web/ws/logs.js
const { WebSocketServer } = require('ws');

function attachLogsWs(server) {
  const wss = new WebSocketServer({ server, path: '/ws/logs' });
  wss.on('connection', (ws, req) => {
    if (!authenticate(req)) return ws.close(4401, 'Unauthorized');
    const sub = eventBus.subscribe('log.published', (entry) => {
      if (entry.guild_id === ws.guildId) {
        ws.send(JSON.stringify(entry));
      }
    });
    ws.on('close', () => eventBus.unsubscribe(sub));
  });
}
```

Côté Nuxt, `useWebSocket('/ws/logs')` écoute et alimente `useLogsStore()`.

## 7. UI (Nuxt)

- Page `/logs` : feed live, filtres par type, recherche par user
- Page `/features/logs` : config (channel par type, events activés, ignored)
- Composant `LogLiveFeed.vue` : auto-scroll, pause, export CSV
- Indicateur "● live" + compteur d'events/minute

## 8. Statistiques du dashboard

- Messages par jour (7 derniers jours)
- Top 10 posteurs
- Top 5 salons actifs
- Évolution du nombre de membres (30j)
- Modération : warns / mutes / bans par semaine
- Latence API Discord

```
GET  /api/stats/overview           # KPIs principaux
GET  /api/stats/messages?days=7
GET  /api/stats/members/growth
GET  /api/stats/moderation?weeks=4
GET  /api/stats/commands?days=30   # usages des slash commands
```

## 9. Critères d'acceptation

- [ ] Tous les events configurables et fonctionnels
- [ ] Sanitization correcte (pas de contenu NSFW re-posté)
- [ ] Ignored channels / users respectés
- [ ] WebSocket stable (reconnexion auto côté front)
- [ ] Page `/logs` opérationnelle avec filtres
- [ ] Tests sur `embed-builder` et `sanitization`
- [ ] Performances : < 50ms par event
