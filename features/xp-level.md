# Feature : XP & Niveaux enrichi

> **Phase** : 2 — **Priorité** : 🟠 Haute — **Statut** : partiellement implémenté, à enrichir

## 1. Objectif

Étendre le système XP existant (`src/modules/engagement_xp-level/`) avec :

- Leaderboards paginés (`/rank`, `/leaderboard`)
- Messages de level-up (configurables)
- Rôles automatiques par palier
- Shop XP (optionnel)
- Page leaderboard sur le dashboard

## 2. État actuel

- `feature_xp-level/` existe, config `xp:` dans `config.example.yml`
- XP message et XP vocal déjà implémentés
- Bonus quotidiens et multiplicateurs déjà câblés
- Manque : leaderboard, level-up messages, rôles auto, shop

## 3. Spécifications

### 3.1 Level-up message

Quand un utilisateur passe un niveau, envoyer un message dans le salon configuré (ou DM).

```yaml
features:
  xp:
    level_up:
      enabled: true
      channel_id: "LEVEL_UP_CHANNEL_ID"  # null = DM
      template: "🎉 Bravo {user} ! Tu passes au **niveau {level}** !"
      color: "#FFD700"
      ping_user: true
      show_rank: true  # affiche la position dans le leaderboard
```

### 3.2 Rôles automatiques

À chaque palier, attribuer / retirer le rôle correspondant.

```yaml
level_roles:
  5: "ACTIVE_MEMBER_ROLE_ID"
  10: "DEVOTED_MEMBER_ROLE_ID"
  20: "VETERAN_ROLE_ID"
  30: "LEGEND_ROLE_ID"
  50: "SERVER_GOD_ROLE_ID"
```

> **Logique** : à chaque gain d'XP, recalculer le niveau. Si le niveau croise un palier, ajouter le rôle (et retirer les anciens si non cumulables selon `cumulable: false`).

### 3.3 Slash commands

| Commande | Description | Permission |
|---|---|---|
| `/rank [user]` | Affiche le niveau et l'XP d'un utilisateur | tous |
| `/leaderboard [page]` | Top 10 par page | tous |
| `/xp-set <user> <amount>` | Override l'XP (admin) | admin |
| `/xp-add <user> <amount>` | Ajoute de l'XP (admin) | admin |
| `/xp-reset <user>` | Remet à zéro | admin |

### 3.4 Leaderboard

- Tri par XP décroissant
- Pagination 10 par page (boutons ◀ ▶)
- Affiche : rang, avatar, pseudo, niveau, XP
- Cache 60s pour éviter de marteler la BDD

### 3.5 API REST

```
GET    /api/xp/leaderboard?page=1
GET    /api/xp/user/:id
GET    /api/xp/config
PATCH  /api/xp/config
POST   /api/xp/adjust         # body: { user_id, delta, reason }
```

## 4. Base de données

Table existante `xp_users` à enrichir :

```sql
-- Champs à ajouter
ALTER TABLE xp_users ADD COLUMN total_messages INTEGER DEFAULT 0;
ALTER TABLE xp_users ADD COLUMN total_voice_minutes INTEGER DEFAULT 0;
ALTER TABLE xp_users ADD COLUMN last_level INTEGER DEFAULT 0;
ALTER TABLE xp_users ADD COLUMN rank_cached INTEGER;
ALTER TABLE xp_users ADD COLUMN rank_updated_at INTEGER;
```

## 5. Architecture

```
src/modules/engagement_xp-level/
├── xp-level.module.js
├── config/schema.js
├── db/schema.js
├── services/
│   ├── xp.service.js
│   ├── level.service.js
│   ├── leaderboard.service.js
│   ├── level-roles.service.js
│   └── level-up.service.js
├── events/
│   ├── xp-message.listener.js
│   └── xp-voice.listener.js
├── commands/
│   ├── rank.command.js
│   ├── leaderboard.command.js
│   ├── xp-set.command.js
│   ├── xp-add.command.js
│   └── xp-reset.command.js
├── controllers/
│   └── xp.controller.js
└── tests/
    ├── level.test.js
    ├── leaderboard.test.js
    └── level-roles.test.js
```

## 6. UI (Nuxt)

- Page `/leaderboard` : top 100, podium pour le top 3, recherche
- Page `/features/xp` : config (bonus, paliers, level-up message)
- Page profil utilisateur : niveau, XP, progression, badges

## 7. Critères d'acceptation

- [ ] Level-up message s'affiche correctement
- [ ] Rôles auto attribués / retirés au bon moment
- [ ] `/rank` et `/leaderboard` fonctionnels et performants (< 200ms)
- [ ] API REST testée
- [ ] UI dashboard opérationnelle
- [ ] Tests unitaires sur la formule de niveau et le service de leaderboard
- [ ] Rétrocompatibilité avec la config `xp:` YAML existante
