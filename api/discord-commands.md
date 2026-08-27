# Référence — Slash Commands

> Convention : `kebab-case` pour les noms, groupes par préfixe (`/mod ...`, `/ticket ...`).

## 1. Légende permissions

- **Tous** : aucun rôle requis
- **Mod** : rôle autorisé pour la feature ou rôle `ADMIN`
- **Admin** : `ADMINISTRATOR` permission
- **Bot owner** : `process.env.BOT_OWNER_ID`

## 2. Commandes existantes

### `/choose_member`

Choisit aléatoirement un membre parmi les participants.

| Option | Type | Requis | Description |
|---|---|---|---|
| `channel` | Channel | non | Salon où chercher (défaut : courant) |

Permission : **Tous**

### `/config`

Configure le bot (rôles, salons, seuils).

Permission : **Admin** (rôles configurés dans `discord.commands.permissions.config.allowed_roles`)

### `/confirm_member`

Confirme un membre (workflow de validation manuelle).

Permission : **Mod**

## 3. Commandes à venir (Phase 1 — Automod)

### `/mod warn <user> <reason>`

Permission : **Mod**

| Option | Type | Requis | Description |
|---|---|---|---|
| `user` | User | oui | Utilisateur à avertir |
| `reason` | String | oui | Raison (max 512) |

### `/mod mute <user> <duration> [reason]`

Permission : **Mod**

| Option | Type | Requis | Description |
|---|---|---|---|
| `user` | User | oui | Utilisateur à mute |
| `duration` | String | oui | Durée (ex: `1h`, `30m`, `1d`) |
| `reason` | String | non | Raison |

### `/mod kick <user> <reason>`

Permission : **Mod**

### `/mod ban <user> [duration] [reason] [delete_days]`

Permission : **Mod**

### `/mod unban <user_id> <reason>`

Permission : **Mod**

### `/mod history <user>`

Permission : **Mod**

Affiche les 20 dernières actions contre cet utilisateur.

### `/mod clear <amount> [user]`

Permission : **Mod**

| Option | Type | Requis | Description |
|---|---|---|---|
| `amount` | Integer | oui | Nombre de messages (1-100) |
| `user` | User | non | Filtrer par utilisateur |

### `/mod automod-config [view|set|reset]`

Permission : **Admin**

## 4. Commandes à venir (Phase 2 — XP)

### `/rank [user]`

Permission : **Tous**

### `/leaderboard [page]`

Permission : **Tous**

### `/xp-set <user> <amount>`

Permission : **Admin**

### `/xp-add <user> <amount>`

Permission : **Admin**

### `/xp-reset <user>`

Permission : **Admin**

## 5. Commandes à venir (Phase 3 — Tickets)

### `/ticket close [reason]`

Permission : claimer / staff

### `/ticket claim`

Permission : **staff**

### `/ticket unclaim`

Permission : claimer / admin

### `/ticket add <user>`

Permission : **staff**

### `/ticket remove <user>`

Permission : **staff**

### `/ticket rename <name>`

Permission : **staff**

### `/ticket transcript`

Permission : **staff**

### `/ticket reopen`

Permission : **Admin**

## 6. Commandes à venir (Phase 5 — Giveaways & Polls)

### `/giveaway start`

Modal : `prize`, `winners_count`, `duration`, `channel`

Permission : **Mod**

### `/giveaway end <id>`

Permission : **Mod**

### `/giveaway reroll <id>`

Permission : **Mod**

### `/giveaway list`

Permission : **Mod**

### `/giveaway cancel <id>`

Permission : **Mod**

### `/poll create`

Modal : `question`, `options` (jusqu'à 10), `duration`, `multi_choice`

Permission : **Mod** (par défaut, modifiable via config)

### `/poll end <id>`

Permission : auteur / **Mod**

### `/poll delete <id>`

Permission : auteur / **Mod**

### `/poll list`

Permission : **Tous**

## 7. Déploiement des commandes

```bash
# Déployer sur le(s) serveur(s) de dev
npm run deploy

# Nettoyer toutes les commandes
npm run delete:all-commands
```

Le script `src/deploy-commands.js` lit les définitions de chaque `@Command` (via `__commandBuilder`) et les enregistre via l'API Discord.

## 8. Bonnes pratiques

- **Description** : claire, 1-2 phrases
- **Options** : ordonnées par importance, requis en premier
- **Choices** : utiliser des `addChoices` pour les valeurs énumérées
- **Choices localisées** : `addChoices({ name: '...', nameLocalizations: { fr: '...' }, value: '...' })`
- **DM** : `setContexts(0)` (Guild), `1` (Bot DM), `2` (Private Channel)
- **Permissions** : `setDefaultMemberPermissions(PermissionFlagsBits.ModerateMembers)` au niveau du builder
