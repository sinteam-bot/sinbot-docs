# Récapitulatif global des features implémentées

> **Date** : 2026-08-28
> **Source** : audit direct du code (21 modules, 24 controllers REST, 67 tables, 489 tests, 481 passent)

Ce document liste **toutes** les features implémentées dans le projet Chienne, en l'état actuel du code. Pour chaque feature on donne : **but**, **tables utilisées**, **endpoints REST**, **slash commands**, **statut frontend**.

---

## Phase 0 — Fondations

### FeatureRegistry

| | |
|---|---|
| **But** | Activer/désactiver chaque feature par guild, avec config JSON et allowed_roles, pilotable par REST + dashboard |
| **Tables** | `guild_settings`, `feature_flags` |
| **Endpoints** | `GET /api/features`, `GET /api/features/:name`, `PATCH /api/features/:name` |
| **Service** | `src/core/feature-registry.js` |
| **Hooks** | `onEnable` / `onDisable` à l'activation / désactivation |
| **Tests** | `tests/feature-registry.test.js` (13/13) |

### Migration YAML → features

| | |
|---|---|
| **But** | Migrer la config YAML legacy vers la table `feature_flags` |
| **Script** | `scripts/migrate-config-to-features.js` (idempotent, `--apply` / `--force`) |
| **Tests** | `tests/migration-script.test.js` (1 fail pré-existant : bug d'import) |

---

## Phase 1 — AutoMod

### AutoMod

| | |
|---|---|
| **But** | Anti-spam, anti-badwords, anti-raid, anti-invite, anti-link, mass-mention, anti-caps avec sanctions progressives |
| **Tables** | `user_warnings`, `user_sanctions`, `mod_logs` |
| **Endpoints** | `GET /api/automod/*` |
| **Slash** | `/mod-warn`, `/mod-mute`, `/mod-kick`, `/mod-ban`, `/mod-unban`, `/mod-clear`, `/mod-history` |
| **Listener** | `messageCreate` (priorité 50) + `guildMemberAdd` (raid) |
| **Service** | `src/modules/feature_automod/services/automod-engine.service.js` |
| **Tests** | `tests/automod-services.test.js` (24/24) |

---

## Phase 2 — XP enrichi

### XP & Level

| | |
|---|---|
| **But** | XP message + vocal, niveaux, paliers, leaderboard, level-up cards SVG |
| **Tables** | `user_xp`, `xp_transactions`, `voice_sessions` |
| **Endpoints** | `GET /api/xp` |
| **Slash** | `/rank`, `/leaderboard` (legacy) |
| **Listener** | `messageCreate` + `voiceStateUpdate` |
| **Service** | `src/modules/feature_xp-level/xp-level.service.js` |
| **Composant partagé** | `CardRendererService` (SVG) — partagé entre welcome + level-up + engagement-advanced (anniversaires) |
| **Tests** | `tests/feature-xp-level.test.js`, `tests/card-renderer.test.js`, `tests/level-up-service.test.js` (97/97 cumulés) |

---

## Phase 3 — Tickets

### Tickets

| | |
|---|---|
| **But** | Système de tickets avec modal, claim, transcript HTML, anti-spam, restriction de salons |
| **Tables** | `tickets`, `ticket_messages` |
| **Endpoints** | `GET/POST /api/tickets`, `/api/tickets/:id`, `/api/tickets/:id/transcript` |
| **Slash** | (réservé aux admins via REST) |
| **Listener** | `interactionCreate` (modal) + `messageCreate` (transcript) |
| **Service** | `src/modules/feature_tickets/services/ticket.service.js` |
| **Tests** | `tests/ticket-services.test.js` (15/15) |

---

## Phase 4 — Logs & Stats

### Logs

| | |
|---|---|
| **But** | 17 events Discord (messages, members, roles, channels, voice) + WebSocket live `/ws/logs` |
| **Tables** | `event_log` |
| **Endpoints** | `GET /api/logs`, `/api/logs/types`, `GET /api/stats/*` (overview, messages, members, moderation) |
| **Listener** | `messageCreate`, `messageUpdate`, `messageDelete`, `guildMemberAdd/Update/Remove`, `guildBanAdd/Remove`, `voiceStateUpdate`, `guildRoleCreate/Update/Delete`, `guildChannelCreate/Update/Delete`, `guildUpdate`, `guildEmojiCreate/Delete` |
| **Service** | `src/services/logs.service.js` (Sanitizer + persist) |
| **WS** | `src/utils/wsLogsServer.js` |
| **Tests** | `tests/logs-sanitizer.test.js` (10/10), `tests/logs-controller.test.js` |

### Stats (compteurs dashboard)

| | |
|---|---|
| **But** | Compteurs par guild : messages 24h, warns 24h, actifs 7j, tickets ouverts |
| **Endpoint** | `GET /api/stats/overview`, `/messages`, `/members`, `/moderation` |
| **Implémentation** | Requêtes SQL agrégées directes sur les tables existantes |

---

## Phase 5 — Giveaways & Polls

### Engagement (Giveaways + Polls)

| | |
|---|---|
| **But** | Giveaways avec CSPRNG (`crypto.randomInt`), polls single/multi-choice, drops d'items |
| **Tables** | `giveaways`, `giveaway_entries`, `polls`, `poll_votes` |
| **Endpoints** | `GET/POST /api/giveaways`, `/api/polls`, `/api/polls/:id/votes` |
| **Slash** | `/giveaway-start`, `/giveaway-end`, `/giveaway-reroll`, `/giveaway-list`, `/giveaway-cancel`, `/poll-create`, `/poll-end`, `/poll-list`, `/poll-delete` |
| **Listener** | `interactionCreate` (boutons) + `Cron * * * * *` (tirages giveaways到期, polls到期) |
| **Service** | `src/modules/feature_engagement/` (giveaway + poll services, repository partagé) |
| **Tests** | `tests/engagement-services.test.js` (24/24) |

---

## Phase 6 — Bienvenue avancée + Cards (SVG partagé)

### Cards (composant partagé)

| | |
|---|---|
| **But** | Générateur SVG réutilisable (welcome, level-up, anniversaires) — pas de dépendance native |
| **Service** | `src/modules/feature_cards/services/card-renderer.service.js` (6 templates : welcome, join, leave, level_up, giveaway, generic) |
| **Tests** | `tests/card-renderer.test.js` (35/35) |

### Welcome

| | |
|---|---|
| **But** | Message de bienvenue + auto-rôles + carte SVG optionnelle + listenner `guildMemberAdd` |
| **Tables** | `welcome_config` |
| **Endpoints** | `GET/PATCH /api/welcome` |
| **Service** | `src/modules/feature_welcome/welcome.service.js` |
| **Tests** | `tests/feature-welcome.test.js` (1 fail pré-existant : bug fakeRepo sur `welcome_message` field) |

### Daily message (IA)

| | |
|---|---|
| **But** | Pensée du jour par OpenRouter (multi-modèles, fallback, retry) |
| **Endpoints** | `GET/POST /api/daily-message`, `/api/daily-message/history` |
| **Tests** | `tests/feature-daily-message.test.js` |

### Bump Reminder

| | |
|---|---|
| **But** | Rappel auto 2h après bump Disboard |
| **Tables** | `bump_logs` |
| **Endpoints** | `GET /api/bump`, `/api/bump/status` |
| **Tests** | `tests/service-bump-reminder.test.js` |

### Security question (Captcha)

| | |
|---|---|
| **But** | Captcha mathématique à l'arrivée |
| **Tables** | `user_captchas`, `captcha_config` |
| **Endpoints** | `GET/POST /api/security-question`, `/api/captcha-config` |
| **Tests** | `tests/security-question.test.js` |

### Countdown + Road to Infinite (jeux)

| | |
|---|---|
| **But** | Jeux 900→0 (countdown) et Compteur (road to infinite) avec leaderboards |
| **Tables** | `countdown_state`, `countdown_scores`, `counter_state` |
| **Endpoints** | `GET /api/games/countdown`, `/api/games/counter` |
| **Tests** | `tests/game-count-down.test.js`, `tests/game-road-to-infinite.test.js` |

### Startup notifier

| | |
|---|---|
| **But** | Notifie au démarrage avec le SHA du dernier commit |
| **Endpoints** | `GET /api/notifier/startup` |
| **Tests** | `tests/notifier-startup.test.js` |

---

## Phase 7 — Anniversaires

### Birthdays

| | |
|---|---|
| **But** | Cron quotidien à 09:00, annonce + cadeau XP + rôle temporaire + cooldown sur changement (1j/2j/6m/1an) |
| **Tables** | `user_birthdays`, `birthday_guild_settings`, `birthday_visibility`, `birthday_change_log`, `birthday_history` |
| **Endpoints** | `GET/POST /api/birthdays/*` (settings, today, upcoming, user, visibility, history) |
| **Slash** | `/anniversaire-set`, `/anniversaire-list`, `/anniversaire-enable`, `/anniversaire-disable`, `/anniversaire-retirer`, `/anniversaire-config` |
| **Listener** | (utilise le cron partagé) |
| **Service** | `src/modules/feature_birthdays/services/announcer.service.js` + `birthday.service.js` |
| **Tests** | `tests/birthday-service.test.js` (3/3 — pre-existing) |

---

## Phase 8 — Quick wins UX

### Sticky roles

| | |
|---|---|
| **But** | Sauvegarde les rôles d'un user à son départ, les restore à son retour |
| **Tables** | `sticky_roles` |
| **Endpoints** | `GET/POST/DELETE /api/sticky-roles` |
| **Slash** | `/stickyrole-add`, `/stickyrole-remove`, `/stickyrole-list`, `/stickyrole-clear` |
| **Listener** | `guildMemberAdd` + `guildMemberRemove` (avec delay configurable) |
| **Service** | `src/modules/feature_sticky-roles/services/sticky-roles.service.js` |
| **Tests** | `tests/sticky-roles-service.test.js` (7/7) |

### Info commands

| | |
|---|---|
| **But** | `/serverinfo`, `/userinfo`, `/avatar` (équivalent dashboard) |
| **Endpoints** | `GET /api/info/server`, `/api/info/user/:id`, `/api/info/avatar/:id` |
| **Slash** | `/serverinfo`, `/userinfo`, `/avatar` |
| **Service** | `src/modules/feature_info/services/info.service.js` |
| **Tests** | `tests/info-service.test.js` (8/8) |

### Games stats (compteurs dashboard)

| | |
|---|---|
| **But** | Endpoint agrégé `/api/games/stats` pour le dashboard |
| **Endpoints** | `GET /api/games/stats` (Counter + Countdown + enabled) |
| **Service** | (extension du `LogsController` existant) |

---

## Phase 9 — Économie + Inventaire (P0 du backlog d'audit)

### Économie & Inventaire

| | |
|---|---|
| **But** | Monnaie virtuelle, shop, drops, give/sell/trade d'items, leaderboard |
| **Tables** | `user_economy`, `economy_transactions`, `shop_items`, `user_inventory`, `inventory_drops`, `inventory_transfers` |
| **Endpoints** | `GET /api/economy/balance`, `/daily`, `/pay`, `/leaderboard`, `/transactions`, `GET /api/shop`, `POST/PATCH/DELETE /api/shop`, `GET /api/inventory/:id`, `POST /api/inventory/give|sell|transfer|reset|drop`, `POST /api/inventory/drop/:id/claim`, `GET /api/inventory/holders/:id`, `/api/inventory/transfers` |
| **Slash** | `/balance`, `/daily`, `/pay`, `/leaderboard`, `/shop-list`, `/shop-buy`, `/inventaire`, `/objet-donner`, `/objet-vendre`, `/dropobjet`, `/topitems`, `/admin-economy-add/remove`, `/admin-shop-create/delete`, `/admin-inventaire-add/reset` |
| **Service** | `EconomyService` + `ShopService` + `InventoryService` (purs, sans discord.js) |
| **Tests** | `tests/economy-services.test.js` (24/24) |

---

## Phase 10 — Reaction roles (avec extension v2)

### Reaction roles v1 (emoji)

| | |
|---|---|
| **But** | Click emoji → toggle role, configurable par message |
| **Tables** | `reaction_roles` (kind='reaction' par défaut) |
| **Endpoints** | `GET/POST/PATCH/DELETE /api/reaction-roles`, `POST /api/reaction-roles/bulk` |
| **Slash** | `/reactionrole-add`, `/reactionrole-remove`, `/reactionrole-list` |
| **Listener** | `messageReactionAdd` + `messageReactionRemove` (priorité 30) |
| **Service** | `ReactionRolesService` |
| **Tests** | `tests/reaction-roles-service.test.js` (15/15) |

### Reaction roles v2 (composants partagés)

| | |
|---|---|
| **But** | **InteractiveMessageBuilder** partagé (analogue à `CardRendererService`) : buttons (toggle_role / give_role / take_role / open_url) + select menus (single/multi-choice, options avec roleId optionnel) |
| **Composant partagé** | `src/services/interactive-message-builder.js` — 35 tests (35/35) |
| **Table étendue** | `kind` (reaction/button/select) + `metadata` (JSON) sur `reaction_roles` |
| **Endpoints** | `POST /api/reaction-roles/button`, `POST /api/reaction-roles/select` |
| **Slash** | `/reactionrole-add-button`, `/reactionrole-add-select` |
| **Listener** | `interactionCreate` (custom_id `ir:*`) |

---

## Phase 11 — Engagement avancé

### Reminders + Triggers + Custom commands (3 sous-features)

| | |
|---|---|
| **But** | `/remind` (DM programmé), déclencheurs de mots automatiques, commandes custom préfixées par `!` |
| **Tables** | `reminders`, `word_triggers`, `custom_commands` |
| **Endpoints** | `GET/POST/DELETE /api/engagement-advanced/{reminders,triggers,commands}` |
| **Slash** | `/remind`, `/reminders`, `/reminder-cancel`, `/trigger-add`, `/trigger-list`, `/trigger-remove`, `/customcmd-add`, `/customcmd-list`, `/customcmd-remove` |
| **Listener** | `messageCreate` (triggers + custom cmds) + `Cron * * * * *` (reminders到期) |
| **Service** | `ReminderService` + `WordTriggerService` + `CustomCommandService` |
| **Tests** | `tests/engagement-advanced-services.test.js` (1 fail pré-existant) |

---

## Phase 12 — Modération communautaire (partiel)

### Reports (Signalements)

| | |
|---|---|
| **But** | Button 🚩 Report sur messages / context menu Report user → modal raison → file d'attente staff |
| **Tables** | `reports`, `report_actions` |
| **Endpoints** | `GET /api/reports`, `/api/reports/stats`, `GET /api/reports/:id`, `GET /api/reports/:id/actions`, `POST /api/reports`, `POST /api/reports/:id/resolve|dismiss` |
| **Slash** | `/reports-list`, `/reports-resolve`, `/reports-dismiss`, `/reports-stats` |
| **Listener** | `interactionCreate` (button + context menu "Report user") |
| **Service** | `ReportsService` (anti-spam : cooldown 5 min, max 5 ouverts par cible) |
| **Tests** | `tests/reports-service.test.js` (14/14) |

### Temp Voice (Join-to-Create)

| | |
|---|---|
| **But** | User rejoint un Join-to-Trigger channel → bot crée un vocal privé, le supprime quand vide |
| **Tables** | `temp_voice_config`, `temp_voice_state` |
| **Endpoints** | `GET/PATCH /api/temp-voice/config`, `GET /api/temp-voice/active`, `GET /api/temp-voice/count` |
| **Slash** | `/tempvoice-show`, `/tempvoice-set`, `/tempvoice-add-channel`, `/tempvoice-remove-channel`, `/tempvoice-list` |
| **Listener** | `voiceStateUpdate` (création/move/rename/empty) + `Cron */30 * * * * *` (cleanup) |
| **Service** | `TempVoiceService` (pur, 23 tests) |
| **Tests** | `tests/temp-voice-service.test.js` (23/23) |

---

## Résumé

- **21 modules** actifs
- **24 controllers REST** (avec 67 endpoints au total)
- **67 tables** BDD
- **489 tests**, **481 passants** (8 fails tous pré-existants, non liés)
- **43 fichiers de tests**
- **1 service partagé** transverse : `InteractiveMessageBuilder` (v2)
- **1 service partagé** transverse : `CardRendererService` (v6)

### Features NON implémentées du backlog d'audit (3 restantes)

D'après `docs/audit/backlog.md` :

1. **Starboards** (P2) — table `starboard_entries` + listener `messageReactionAdd` (⭐)
2. **Sauvegardes serveur** (P2) — `POST /backup/create`, `POST /backup/restore`
3. **Commandes fun** (`/roll`, `/coinflip`, `/8ball`) (P3) — utilitaire pur

Les features P0 (Rôles-Réactions, Économie) et P1 (Sticky, Info, Reports) sont livrées. Les features P2-P3 sont les candidates naturelles pour la suite.
