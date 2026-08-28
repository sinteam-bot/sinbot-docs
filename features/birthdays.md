# Feature : Anniversaires (Birthdays)

> **Phase** : 7 — **Priorité** : 🟡 Moyenne — **Statut** : ✅ livrée

## 1. Objectif

Implémenter le système d'anniversaires de Draftbot dans Chienne :
- Permettre aux membres de définir leur date d'anniversaire
- Célébrer automatiquement les anniversaires du jour avec une annonce
- Donner un rôle temporaire + des cadeaux le jour J
- Mode public (BDD globale) ou privé (BDD par-guild)
- Cooldown sur les changements de date (1j / 2j / 6 mois / 1 an)

## 2. Cas d'usage

| Cas | Comportement |
|---|---|
| **Définir** | `/anniversaire set date:JJ/MM` |
| **Lister** | `/anniversaire list` — 10 prochains du serveur |
| **Activer visibilité** | `/anniversaire enable` (par serveur) |
| **Désactiver visibilité** | `/anniversaire disable` |
| **Retirer** | `/anniversaire retirer` (BDD globale) |
| **Annonce auto** | Cron quotidien 09:00, embed avec `{user} {age} {gifts} {role}` |
| **Rôle temporaire** | Donné le jour J, retiré à 00:00 le lendemain |
| **Cadeaux** | Jusqu'à 2 par user (rôle, XP, argent, objet, custom) |
| **Cooldown** | 1j / 2j / 6 mois / 1 an selon le nb de changements |

## 3. Spécifications

### 3.1 Configuration

```yaml
features:
  birthdays:
    enabled: false
    allowed_roles: []

    mode: public                  # public | private

    announce:
      channel_id: null            # null = salon courant
      hour: 9                     # 0-23
      timezone: Europe/Paris
      ping_role_id: null
      message_template: "🎂 Joyeux anniversaire {user} ! Tu fêtes tes **{age} ans** aujourd'hui !"

    temp_role:
      enabled: true
      role_id: null

    gifts:
      max_per_user: 2
      xp_per_birthday: 500        # cadeau XP par défaut (0 = désactivé)
      items: []                   # [{type: 'role', id: '...', name: '...', amount: 1}, ...]

    cooldown:
      first_change_days: 1
      second_change_days: 2
      third_change_days: 180     # 6 mois
      default_change_days: 365    # 1 an
```

### 3.2 Variables du template

| Variable | Description |
|---|---|
| `{user}` | Mention Discord `<@id>` |
| `{username}` | Pseudo de l'utilisateur |
| `{age}` | Âge calculé (années) |
| `{role}` | Mention du rôle à mentionner |
| `{gifts}` | Liste descadeaux formatée |

### 3.3 Cooldown (spec Draftbot)

| Nombre de changements | Cooldown |
|---|---|
| 1er | 1 jour |
| 2ème | 2 jours |
| 3ème | 6 mois |
| 4ème+ | 1 an |

## 4. Architecture

```
feature_birthdays/
├── config/defaults.js
├── services/
│   ├── birthday.repository.js   # CRUD BDD
│   ├── birthday.service.js      # Logique métier + cooldown
│   ├── announcer.service.js     # Génère l'embed + envoie
│   ├── gift.service.js          # Distribution des cadeaux
│   └── birthday-cron.js         # @Cron quotidien
├── commands/birthday-commands.js # /anniversaire *
├── controllers/birthday.controller.js # REST
└── tests/birthday-service.test.js
```

## 5. Base de données

### `birthday_guild_settings` (PK = `guild_id`)

| Colonne | Type | Description |
|---|---|---|
| `guild_id` | TEXT PK | Discord snowflake |
| `mode` | TEXT | `public` ou `private` |
| `announce_channel_id` | TEXT nullable | Salon d'annonce |
| `announce_hour` | INTEGER | Heure d'envoi (0-23) |
| `announce_timezone` | TEXT | Fuseau horaire |
| `ping_role_id` | TEXT nullable | Rôle à mentionner |
| `message_template` | TEXT | Template avec variables |
| `temp_role_id` | TEXT nullable | Rôle temporaire |
| `enabled` | INTEGER | 0/1 |
| `created_at`, `updated_at` | INTEGER | epoch ms |

### `birthday_visibility` (PK = `(user_id, guild_id)`)

| Colonne | Type | Description |
|---|---|---|
| `user_id` | TEXT | Discord snowflake |
| `guild_id` | TEXT | Discord snowflake |
| `enabled` | INTEGER | 0/1, visibilité sur CE serveur |

### `birthday_change_log` (PK = `id`)

| Colonne | Type | Description |
|---|---|---|
| `user_id` | TEXT | Discord snowflake |
| `guild_id` | TEXT nullable | null en mode public |
| `change_number` | INTEGER | 1, 2, 3, 4... |
| `previous_birthdate`, `new_birthdate` | TEXT | JJ-MM ou JJ-MM-YYYY |
| `cooldown_until` | INTEGER | epoch ms |
| `changed_at` | INTEGER | epoch ms |

### `birthday_history` (PK = `id`)

| Colonne | Type | Description |
|---|---|---|
| `guild_id`, `user_id` | TEXT | Discord snowflakes |
| `username` | TEXT | pseudo au moment de l'annonce |
| `age` | INTEGER nullable | âge calculé |
| `message_id` | TEXT nullable | ID du message d'annonce |
| `gifts_given` | TEXT | JSON array descadeaux |
| `announced_at` | INTEGER | epoch ms |

## 6. API publique du service

```js
class BirthdayService {
    // Settings (admin)
    getSettings(guildId): Promise<Settings>
    updateSettings(guildId, patch): Promise<Settings>

    // User-side
    setBirthday({ userId, username, guildId, birthdate }): Promise<{ok, error?, nextChangeAt?}>
    getBirthday(userId, guildId): Promise<Birthday | null>
    removeBirthday(userId, guildId): Promise<{ok}>
    setVisibility(userId, guildId, enabled): Promise<{ok}>
    listToday(guildId): Promise<Birthday[]>
    listUpcoming(guildId, days = 7): Promise<Birthday[]>
    canChangeBirthday(userId, guildId): Promise<{allowed, nextChangeAt}>

    // Cron
    announceTodaysBirthdays(guildId): Promise<{announced, errors}>
    cleanupTempRoles(guildId): Promise<{removed}>
}
```

## 7. Algorithmes clés

### Cooldown

```js
function computeCooldown(changeNumber) {
    if (changeNumber === 1) return 1 * 86400_000;
    if (changeNumber === 2) return 2 * 86400_000;
    if (changeNumber === 3) return 180 * 86400_000;
    return 365 * 86400_000;
}
```

### Prochain anniversaire

```js
function nextBirthday(birthdate, fromDate = new Date()) {
    let next = new Date(fromDate.getFullYear(), birthdate.getMonth(), birthdate.getDate());
    if (next < fromDate) next = new Date(fromDate.getFullYear() + 1, birthdate.getMonth(), birthdate.getDate());
    return next;
}
```

### Rendu du template

```js
function renderTemplate(template, vars) {
    return template
        .replace(/\{user\}/g, `<@${vars.userId}>`)
        .replace(/\{username\}/g, vars.username || 'Utilisateur')
        .replace(/\{age\}/g, String(vars.age ?? '?'))
        .replace(/\{role\}/g, vars.roleId ? `<@&${vars.roleId}>` : '')
        .replace(/\{gifts\}/g, vars.gifts || 'Aucun cadeau');
}
```

## 8. REST API

```
GET    /api/birthdays/settings?guild_id=
PATCH  /api/birthdays/settings
GET    /api/birthdays/today?guild_id=
GET    /api/birthdays/upcoming?guild_id=&days=7
GET    /api/birthdays/user/:userId?guild_id=
PUT    /api/birthdays/user/:userId        # body: { birthdate, username }
DELETE /api/birthdays/user/:userId
POST   /api/birthdays/user/:userId/visibility # body: { enabled }
```

## 9. Slash commands

- `/anniversaire set date:JJ-MM` — définir
- `/anniversaire list` — 10 prochains
- `/anniversaire enable` — visibilité ON
- `/anniversaire disable` — visibilité OFF
- `/anniversaire retirer` — suppression totale
- `/anniversaire config` (admin) — modifier la config

## 10. Cron

- `@Cron('0 9 * * *', Europe/Paris)` — annonce les anniversaires du jour
- `@Cron('0 0 * * *', Europe/Paris)` — retire les rôles temporaires

## 11. Critères d'acceptation (DoD)

- [x] Tables créées idempotemment (4 tables)
- [x] Cooldown conforme à la spec Draftbot
- [x] Mode public + mode privé supportés
- [x] Visibilité par serveur
- [x] Annonce quotidienne fonctionnelle
- [x] Rôle temporaire donné + retiré automatiquement
- [x] Jusqu'à 2cadeaux par user
- [x] Variables du template rendues correctement
- [x] Page `/birthdays` opérationnelle
- [x] Tests unitaires ≥ 10
- [x] Aucune régression

## 12. Tests

- `setBirthday` avec date valide → insert
- `setBirthday` avec date invalide → erreur
- `setBirthday` dans le cooldown → erreur + `nextChangeAt`
- `setVisibility` toggle
- `listToday` filtre les anniversaires du jour
- `listUpcoming` tri par jours restants
- `renderTemplate` substitue les variables
- `nextBirthday` calcule la bonne date
- `computeCooldown` suit la spec Draftbot
- `announceTodaysBirthdays` envoie l'embed
