# Feature : Signalements (Reports)

> **Phase** : 12 — **Priorité** : 🟡 Haute — **Statut** : à implémenter (P1 du backlog d'audit)

## 1. Objectif

Permettre à un membre de **signaler un message ou un comportement** via un bouton "🚩 Report" sur chaque message, et aux modérateurs de gérer ces signalements via une file d'attente.

## 2. Cas d'usage

| Cas | Comportement |
|---|---|
| **Report depuis un message** | Click "🚩 Report" → modal raison → ticket dans la file staff |
| **Report depuis un user** | Menu contextuel "Report user" → modal → ticket dans la file staff |
| **Staff : file d'attente** | `/reports` → embed avec la liste des reports ouverts |
| **Staff : résoudre** | `/reports resolve id:...` → log action + DM au reporter |
| **Staff : refuser** | `/reports dismiss id:...` → status dismissed, log sans DM |
| **Stats** | `/reports stats` → compteurs par type, période, par staff |
| **Anti-spam** | Cooldown par user (5 min) pour éviter les reports abusifs |

## 3. Spécifications

### 3.1 Configuration

```yaml
features:
  reports:
    enabled: false
    allowed_roles: []              # Discord roles autorisés à gérer (par défaut ManageMessages)
    report_channel_id: null        # Salon où poster la file d'attente
    log_channel_id: null           # Salon où poster chaque report (optionnel)
    cooldown_seconds: 300         # Cooldown entre deux reports du même user
    max_open_per_user: 5           # Max de reports ouverts par user cible
    dm_on_resolve: true            # DM le reporter quand son report est résolu
    categories:                    # Catégories de raisons
      - spam
      - harassment
      - inappropriate
      - misinformation
      - other
```

### 3.2 Permissions

- **Reporter** : tout membre peut signaler (avec cooldown)
- **Staff** : `ManageMessages` (par défaut) ou rôles listés dans `allowed_roles`
- **Anti-spam** : un user ne peut pas signaler plus de `max_open_per_user` fois le même user cible (pour éviter le harcèlement par signalements répétés)

### 3.3 Cycle de vie d'un report

```
open (créé) ──► dismissed (staff rejette)
            └────► resolved (staff prend une action, ex: warn)
```

`resolved` est associé à une `report_action` (warn / kick / ban / dismiss / custom) qui loggue l'action et le staff_id.

## 4. Architecture

```
feature_reports/
├── config/defaults.js
├── db/schema.js (couvert par src/db/schema/pg.js)
├── services/
│   ├── reports.repository.js
│   └── reports.service.js
├── events/
│   └── reports-listener.js
├── commands/reports-commands.js
├── controllers/reports.controller.js
└── tests/reports-service.test.js
```

## 5. Base de données (2 tables)

### `reports`

| Colonne | Type | Description |
|---|---|---|
| `id` | TEXT PK | UUID v4 |
| `guild_id` | TEXT NOT NULL | Discord snowflake |
| `reporter_id` | TEXT NOT NULL | ID de celui qui signale |
| `reported_id` | TEXT NOT NULL | ID du user signalé |
| `channel_id` | TEXT | Salon d'origine |
| `message_id` | TEXT | Message d'origine |
| `reason` | TEXT | Raison (texte libre ou catégorie) |
| `category` | TEXT | spam / harassment / ... |
| `status` | TEXT | `open` / `resolved` / `dismissed` |
| `resolved_by` | TEXT | Staff qui a traité |
| `resolved_at` | INTEGER | epoch ms |
| `created_at` | INTEGER NOT NULL | epoch ms |

Index : `(guild_id, status, created_at)` pour la file d'attente.

### `report_actions`

| Colonne | Type | Description |
|---|---|---|
| `id` | TEXT PK | UUID v4 |
| `report_id` | TEXT NOT NULL | FK vers reports.id |
| `staff_id` | TEXT NOT NULL | Staff qui a agi |
| `action` | TEXT NOT NULL | `warn` / `kick` / `ban` / `dismiss` / `custom` |
| `notes` | TEXT | Notes du staff |
| `created_at` | INTEGER NOT NULL | epoch ms |

Index : `(report_id)` pour récupérer les actions d'un report.

## 6. API publique

### `ReportsService`

- `create({guildId, reporterId, reportedId, channelId?, messageId?, reason, category?})` : valide + insère
- `list({guildId, status?, reporterId?, reportedId?, limit, offset})` : file d'attente
- `get(id)` : détails d'un report
- `resolve(id, staffId, action, notes?)` : marquer résolu + log action
- `dismiss(id, staffId, notes?)` : refuser
- `addAction(reportId, staffId, action, notes?)` : log d'une action
- `listActions(reportId)` : log des actions sur un report
- `stats(guildId, sinceMs?)` : compteurs (open, resolved, dismissed, top category)
- `canReport(reporterId, reportedId)` : anti-spam (cooldown + max_open_per_user)

### `ReportsListener`

- `messageCreate` : détecte le bouton "🚩 Report" → ouvre un modal de raison
- `interactionCreate` (modal submit) : crée le report
- `messageContextMenu` (Report user) : ouvre le modal

## 7. REST API

```
GET    /api/reports?guild_id=&status=&reporter_id=&reported_id=&page=1
GET    /api/reports/:id
POST   /api/reports                  : body { guildId, reporterId, reportedId, channelId, messageId, reason, category }
POST   /api/reports/:id/resolve      : body { staffId, action, notes }
POST   /api/reports/:id/dismiss      : body { staffId, notes }
GET    /api/reports/:id/actions
GET    /api/reports/stats?guild_id=
```

## 8. Slash commands

**User side** :
- `/report` (context menu : Report message) → modal raison
- `/report-user` (context menu : Report user) → modal raison

**Staff side** (ManageMessages) :
- `/reports list [status:open] [limit:10]` : file d'attente
- `/reports resolve id:... action:warn|kick|ban [notes:...]` : résoudre + action
- `/reports dismiss id:... [notes:...]` : refuser
- `/reports stats` : statistiques

## 9. Événements Discord écoutés

- `messageCreate` : si le message contient un bouton "🚩 Report" et qu'un user le clique, ouvrir modal de raison
- `interactionCreate` (modal submit) : créer le report
- `messageContextMenu` (Report message / Report user) : alternative au bouton

## 10. Critères d'acceptation (DoD)

- [x] 2 tables : `reports` + `report_actions`
- [x] Anti-spam : cooldown 5 min + max 5 reports ouverts par user cible
- [x] Listener bouton Report + modal raison
- [x] File d'attente staff via `/reports list`
- [x] Résolution avec action (warn / kick / ban / dismiss)
- [x] DM au reporter sur résolution (configurable)
- [x] Tests unitaires ≥ 10
- [x] Page dashboard `/modules/reports`

## 11. Tests

- create : insert valide, refus si reporter === reported, refus si cooldown pas écoulé
- list : filtre par status / reporter / reported
- resolve : passe en `resolved`, log une action
- dismiss : passe en `dismissed`
- stats : compteurs corrects
- canReport : retourne false si cooldown, true si ok
