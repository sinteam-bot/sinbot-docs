# Feature — Invites (InviteLogger-like)

> **Statut** : implémenté (v1)
> **Inspiration** : [InviteLogger](https://invitelogger.me) — réimplémentation 100% interne via l'API Discord native.

## Cas d'usage

- Tracker qui a invité qui sur un serveur Discord (join + leave)
- Afficher un embed de notification dans un salon dédié à chaque join/leave
- Leaderboard des inviteurs (`/invite leaderboard`)
- Stats par utilisateur (`/invite stats`, `/invite info`)
- Bonus / malus d'invites par les admins (`/invite add`, `/invite remove`)
- Reset + restore avec snapshots
- Détection de "fake invites" (compte trop récent, pas d'avatar)
- Blacklist de membres/rôles (exclus du leaderboard)
- Création d'invitations permanentes via le bot
- Configuration des salons de log, des paramètres de fake, etc.

## Configuration

```yaml
features:
  invites:
    enabled: true
    join_log_channel_id: "1234567890"
    leave_log_channel_id: "0987654321"
    join_message: ":incoming_envelope: {member} a rejoint via **{inviter}**"
    leave_message: ":outbox_tray: {member} a quitté"
    embed_color: "#2F3136"
    show_account_age: true
    track_bots: false
    fake_account_threshold_days: 7
    fake_no_avatar: true
    leaderboard:
      enabled: true
      page_size: 25
```

## Commandes

Toutes les commandes sont des **subcommands de `/invite`** (pour éviter les conflits avec d'autres bots) :

| Subcommand | Description | Permissions |
|---|---|---|
| `/invite stats [@user]` | Stats d'invites (real / bonus / leaves / fake) | tous |
| `/invite leaderboard [page]` | Top des inviteurs | tous |
| `/invite info @user` | Détails + liste des invités récents | tous |
| `/invite add @user <amount> [reason]` | Ajoute des bonus invites | admin |
| `/invite remove @user <amount> [reason]` | Retire des bonus invites | admin |
| `/invite reset <@user\|all>` | Snapshot + reset | admin |
| `/invite restore <@user\|all>` | Restore depuis snapshot | admin |
| `/invite create [#channel] [@user]` | Crée une invite permanente | admin |
| `/invite logs <join\|leave> #channel` | Définit le salon de log | admin |
| `/invite fake <setting> <value>` | Configure la détection fake | admin |
| `/invite blacklist <add\|remove\|list> [id]` | Blacklist un membre/rôle | admin |
| `/invite config [key] [value]` | Lit / écrit la config | admin |

## API REST

| Endpoint | Description |
|---|---|
| `GET /api/invites/:guildId/:userId` | Stats détaillées d'un user |
| `GET /api/invites/:guildId/leaderboard?limit=25` | Top inviters |
| `GET /api/invites/:guildId/blacklist` | Liste blacklist |

## Architecture

```
feature_invites/
├── db/
│   └── schema.js                 # 5 tables Drizzle
├── invites.module.js             # @Module, providers
├── invites.repository.js         # CRUD sur les 5 tables
├── config/
│   ├── defaults.js               # Config par défaut
│   └── schema.js                 # Validation joi
├── services/
│   └── invites.service.js        # Logique métier (détection, stats, restore)
├── events/
│   └── invites-listener.js       # guildMemberAdd, guildMemberRemove, inviteCreate/Delete, ready
├── commands/
│   └── invite-commands.js         # /invite <subcommand>
├── controllers/
│   └── invites.controller.js      # /api/invites/*
└── tests/
```

### Tables

| Table | Rôle |
|---|---|
| `invite_codes` | Cache des invites par guilde (alimenté par les events Discord) |
| `invite_uses` | Table de faits : 1 ligne par (inviteur, invité, join) |
| `invite_bonuses` | Bonus/malus manuels accordés par les admins |
| `invite_blacklist` | Membres/rôles exclus du leaderboard |
| `invite_restore` | Snapshots pour /invite restore |

### Algorithme de détection de l'inviteur

1. **Sur `ready`** : cache des invites de chaque guilde (`guild.invites.fetch()`)
2. **Sur `guildMemberAdd`** :
   - Snapshot du cache avant
   - `guild.invites.fetch()` après
   - Compare les `uses` : l'invite dont le compteur a augmenté est l'inviteur
   - Fallback "vanity" si aucune invite n'a bougé (URL personnalisée du serveur)
   - Fallback "unknown" si rien ne matche
3. **Sur `guildMemberRemove`** : marque `left_at` sur la dernière ligne d'`invite_uses` du membre

### Détection "fake"

- **Compte trop récent** : `Date.now() - createdTimestamp < fake_account_threshold_days * 24h`
- **Pas d'avatar** : `!member.user.avatar`
- **IP dupliquée** (à venir) : nécessite un service tiers

## Critères d'acceptation

- [x] Tracking correct de l'inviteur via diff de `uses`
- [x] Embed de join envoyé dans `join_log_channel_id`
- [x] Embed de leave envoyé dans `leave_log_channel_id`
- [x] `/invite stats` retourne real/bonus/leaves/fake
- [x] `/invite leaderboard` trié par total décroissant
- [x] `/invite add` / `remove` modifient `invite_bonuses`
- [x] `/invite reset` crée un snapshot et vide les compteurs
- [x] `/invite restore` restaure depuis le dernier snapshot
- [x] `/invite blacklist add/remove/list`
- [x] `/invite fake` configure la détection
- [x] `/invite config key value` (lecture/écriture)
- [x] `/invite create` crée une invite permanente
- [x] REST `GET /api/invites/:guildId/:userId`
- [x] REST `GET /api/invites/:guildId/leaderboard`
- [x] Blacklist appliquée au leaderboard
- [x] Tests unitaires (9 tests)
- [x] Migration Drizzle versionnée (`0001_flashy_forgotten_one.sql`)
