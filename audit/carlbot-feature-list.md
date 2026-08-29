# Carl-bot — Liste exhaustive des features

> **Date** : 2026-08-29
> **Source** : https://docs.carl.gg/ + https://carl.gg/
>
> Ce document liste **toutes** les features publiques Carl-bot, avec :
> - Le lien direct vers la doc
> - Le status dans le Bot (✅ / 🟡 / ❌ / 🔶)
> - Une référence au commit / module qui l'implémente (ou `docs/plan/` si manquant)

---

## 1. Accueil des membres

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Messages de bienvenue (embed) | [link](https://docs.carl.gg/#/welcome) | ✅ | `welcome_welcome` — message embed personnalisable |
| Messages de bienvenue (DM) | [link](https://docs.carl.gg/#/welcome) | ✅ | `welcome_welcome` — DM de bienvenue |
| Messages de départ (leave) | [link](https://docs.carl.gg/#/welcome) | ✅ | `welcome_welcome` — événement `guildMemberRemove` |
| Messages d'annonce (ban/unban) | [link](https://docs.carl.gg/#/welcome) | 🟡 | `welcome_welcome` — pas de message d'annonce de ban |
| Rôles automatiques (autorole) | [link](https://docs.carl.gg/#/roles) | ✅ | `welcome_welcome` — `config.auto_roles` |
| Rôles temporisés (timed roles) | [link](https://docs.carl.gg/#/roles) | ❌ | **À planifier** — pas de rôles temporisés |
| Blacklist de rôles | [link](https://docs.carl.gg/#/roles) | ❌ | **À planifier** — pas de blacklist de rôles |

## 2. Engagement — Niveaux & XP

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| XP par message | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` — [`docs/features/xp-level.md`](../features/xp-level.md) |
| XP vocal | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` — `voice_sessions` |
| Leaderboard | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` — endpoint `/api/xp/leaderboard` |
| Leaderboard public (web) | [link](https://docs.carl.gg/#/levels) | ❌ | **À planifier** — pas de page web publique |
| Récompenses de rôles | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` — attribution par palier |
| Messages de level-up | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` + `welcome_cards` |
| Carte de rang (/rank) | [link](https://docs.carl.gg/#/levels) | ✅ | `welcome_cards` — SVG rank card |
| Taux d'XP configurable | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` — `xp.config.js` |
| Rôles sans XP | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` — exclusion par rôle |
| Salons sans XP | [link](https://docs.carl.gg/#/levels) | ✅ | `engagement_xp-level` — exclusion par salon |

## 3. Engagement — Giveaways & Sondages

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Créer un giveaway | [link](https://docs.carl.gg/#/giveaways) | ✅ | `game_engagement` (Phase 5) — [`docs/features/giveaways-polls.md`](../features/giveaways-polls.md) |
| Terminer un giveaway | [link](https://docs.carl.gg/#/giveaways) | ✅ | `game_engagement` — `/giveaway-end` |
| Reroll giveaway | [link](https://docs.carl.gg/#/giveaways) | ✅ | `game_engagement` — `/giveaway-reroll` |
| Lister les giveaways | [link](https://docs.carl.gg/#/giveaways) | ✅ | `game_engagement` — `/giveaway-list` |
| Annuler un giveaway | [link](https://docs.carl.gg/#/giveaways) | ✅ | `game_engagement` — `/giveaway-cancel` |
| Créer un sondage (/poll) | [link](https://docs.carl.gg/#/polls) | ✅ | `game_engagement` — `/poll-create` |
| Sondage multi-choix | [link](https://docs.carl.gg/#/polls) | ✅ | `game_engagement` — option `multi_choice` |
| Starboard | [link](https://docs.carl.gg/#/starboard) | ❌ | **Planifié Phase 12** — pas de starboard |

## 4. Modération

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commande /ban | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` (Phase 1) — [`docs/features/automod.md`](../features/automod.md) |
| Commande /kick | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` |
| Commande /mute | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` — timeout Discord natif |
| Commande /warn | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` — `user_warnings` |
| Tempban / Tempmute | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` — durée parsée |
| Sanctions temporisées | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` — warn → mute → ban progressif |
| Casiers judiciaires (modlogs) | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_logs` — [`docs/features/logs-stats.md`](../features/logs-stats.md) |
| Historique complet (infraction history) | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_logs` — `user_sanctions` |
| Immunité de rôles | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` — rôles protégés |
| Purge de messages | [link](https://docs.carl.gg/#/moderation) | ✅ | `security_automod` — `/mod clear` |
| Sticky roles | [link](https://docs.carl.gg/#/moderation) | ✅ | `community_sticky-roles` — sauvegarde/restauration |
| Rôles persistants (sticky roles) | [link](https://docs.carl.gg/#/roles) | ✅ | `community_sticky-roles` |

## 5. Auto-Modération

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Filtre de mots interdits (slurs) | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — liste configurable |
| Anti-spam | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — détection de répétition |
| Anti-lien (bad links) | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — filtrage d'invitations |
| Anti-mention (@everyone) | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — mass-mention |
| Anti-caps | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — majuscules excessives |
| Anti-attachment spam | [link](https://docs.carl.gg/#/automod) | ❌ | **À planifier** — pas de filtre d'attachments |
| Sanctions par règle | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — warn/mute/ban par filtre |
| Rate limits personnalisés | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — cooldowns configurables |
| Whitelist de salons | [link](https://docs.carl.gg/#/automod) | ✅ | `security_automod` — canaux ignorés |

## 6. Automatisation

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Triggers (text triggers) | [link](https://docs.carl.gg/#/triggers) | ✅ | `game_engagement-advanced` — [`docs/features/engagement-advanced.md`](../features/engagement-advanced.md) |
| Match exact / contains | [link](https://docs.carl.gg/#/triggers) | ✅ | `game_engagement-advanced` — `matchType: exact/contains` |
| Regex triggers | [link](https://docs.carl.gg/#/triggers) | 🟡 | `game_engagement-advanced` — regex prévu mais non activé |
| Exclusion de salons | [link](https://docs.carl.gg/#/triggers) | ✅ | `game_engagement-advanced` — `excludeChannelIds` |
| Exclusion de rôles | [link](https://docs.carl.gg/#/triggers) | ✅ | `game_engagement-advanced` — `excludeRoleIds` |
| Cooldown par trigger | [link](https://docs.carl.gg/#/triggers) | ✅ | `game_engagement-advanced` — `cooldownSeconds` |
| Réponse embed | [link](https://docs.carl.gg/#/triggers) | ✅ | `game_engagement-advanced` — `responseEmbedJson` |
| Messages répétés (repeating messages) | [link](https://docs.carl.gg/#/repeating) | ❌ | **À planifier** — pas de messages répétés |
| Autofeeds (feeds automatiques) | [link](https://docs.carl.gg/#/autofeeds) | ❌ | **À planifier** — pas de flux automatiques |
| Timers (minuteries) | [link](https://docs.carl.gg/#/timers) | ❌ | **À planifier** — pas de minuteries |

## 7. Commandes personnalisées

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commandes custom (TagScript) | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — [`docs/features/engagement-advanced.md`](../features/engagement-advanced.md) |
| Variables utilisateur | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — `{user}`, `{server}` |
| Variables de serveur | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — `{server}` |
| Variables member count | [link](https://docs.carl.gg/#/tagstriggers) | ❌ | **À planifier** — pas de variable `{server.memberCount}` |
| Variables channel topic | [link](https://docs.carl.gg/#/tagstriggers) | ❌ | **À planifier** — pas de variable channel |
| Réponse embed | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — `responseEmbedJson` |
| Cooldown | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — `cooldownSeconds` |
| Restriction de salon | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — `excludeChannelIds` |
| Restriction mod-only | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — permissions |
| Activation/désactivation | [link](https://docs.carl.gg/#/tagstriggers) | ✅ | `game_engagement-advanced` — toggle on/off |

## 8. Réactions de rôles

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Réactions-Emoji → Rôles | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` (Phase 10) — [`docs/features/reaction-roles.md`](../features/reaction-roles.md) |
| Boutons → Rôles | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` v2 — `InteractiveMessageBuilder` |
| Menus déroulants → Rôles | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` v2 — select menus |
| Mode toggle (ajouter/retirer) | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` — ajout et retrait |
| Mode unique (une seule réaction) | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` — mode unique |
| Mode verify (réaction pour vérifier) | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` — template "Verify" |
| Mode reversed (retirer au lieu d'ajouter) | [link](https://docs.carl.gg/#/roles) | ❌ | **À planifier** — pas de mode inversé |
| Mode binding (lier à un autre rôle) | [link](https://docs.carl.gg/#/roles) | ❌ | **À planifier** — pas de mode lié |
| Mode temporary (rôle temporaire) | [link](https://docs.carl.gg/#/roles) | ❌ | **À planifier** — pas de mode temporaire |
| 250 rôles par message | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` — support multi-rôles |
| Rôles multiples par message | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` — plusieurs rôles |

## 9. Notifications sociales

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Alertes Twitch (live) | [link](https://docs.carl.gg/#/twitch) | 🔶 | **Carl-bot premium**, pas pertinent pour un bot interne |
| Alertes YouTube (vidéos) | [link](https://docs.carl.gg/#/youtube) | 🔶 | **Carl-bot premium**, pas pertinent pour un bot interne |

## 10. Logging & Audit

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Message delete/edit | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` (Phase 4) — [`docs/features/logs-stats.md`](../features/logs-stats.md) |
| Join/Leave | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` — arrivées/départs |
| Voice (join/leave) | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` — join/leave vocal |
| Role changes | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` — changements de rôles |
| Nickname changes | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` — changements de pseudo |
| Ban/Unban | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` — modérations |
| Invite tracking | [link](https://docs.carl.gg/#/logging) | ✅ | `util_invites` — [`docs/features/invites.md`](../features/invites.md) |
| Split par salon | [link](https://docs.carl.gg/#/logging) | 🟡 | `security_logs` — logs centralisés, pas de split par canal |
| Exclusion de salons | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` — canaux ignorés |
| Ignorer les actions de bots | [link](https://docs.carl.gg/#/logging) | ✅ | `security_logs` — filtre par source bot |

## 11. Utilitaires

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Suggestions | [link](https://docs.carl.gg/#/suggestions) | ❌ | **Planifié Phase 12** — pas de système de suggestions |
| Reminders (/remindme) | [link](https://docs.carl.gg/#/reminders) | ✅ | `game_engagement-advanced` — `/remind` |
| Announcements | [link](https://docs.carl.gg/#/announcements) | ✅ | `welcome_welcome` — messages d'annonce |
| Embed builder | [link](https://docs.carl.gg/#/embeds) | ❌ | **À planifier** — pas d'embed builder persistant |
| Salons vocaux temporaires | [link](https://docs.carl.gg/#/tempchannels) | ✅ | `util_temp-voice` — [`docs/features/temp-voice.md`](../features/temp-voice.md) |
| Rôles sécurisés (sticky roles) | [link](https://docs.carl.gg/#/roles) | ✅ | `community_sticky-roles` — sauvegarde/restauration |
| Anniversaires | [link](https://docs.carl.gg/#/announcements) | ✅ | `engagement_birthdays` (Phase 7) — [`docs/features/birthdays.md`](../features/birthdays.md) |
| Signalements (reports) | [link](https://docs.carl.gg/#/moderation) | ✅ | `community_reports` — [`docs/features/reports.md`](../features/reports.md) |
| Tickets | [link](https://docs.carl.gg/#/tickets) | ✅ | `community_tickets` (Phase 3) — [`docs/features/tickets.md`](../features/tickets.md) |
| Compteur de membres (voice) | [link](https://docs.carl.gg/#/tempchannels) | ❌ | **À planifier** — pas de salon vocal statistique |
| Route de l'Infini (jeu) | [link](https://docs.carl.gg/#/games) | ✅ | `game_road-to-infinite` — jeu de comptage |
| Countdown (jeu) | [link](https://docs.carl.gg/#/games) | ✅ | `game_count-down` — jeu de décompte |
| Jeux (games) | [link](https://docs.carl.gg/#/games) | 🟡 | `game_*` — 2 jeux seulement |
| Commandes de rôle (add/remove/edit) | [link](https://docs.carl.gg/#/roles) | ✅ | `community_reaction-roles` + `welcome_welcome` |
| Couleurs de rôles | [link](https://docs.carl.gg/#/roles) | ❌ | **À planifier** — pas de gestion de couleurs |
| Ranks (rangs) | [link](https://docs.carl.gg/#/roles) | ❌ | **À planifier** — pas de système de rangs |
| Text transformation | [link](https://docs.carl.gg/#/text) | ❌ | **À planifier** — pas de transformation de texte |
| Server Discovery | [link](https://carl.gg/) | 🔶 | **Carl-bot premium**, hors périmètre |

## 12. Personnalisation du bot

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Préfixe personnalisé | [link](https://docs.carl.gg/#/settings) | ✅ | `config.chienne.yml` — préfixe configurable |
| Avatar personnalisé | [link](https://carl.gg/get-premium) | 🔶 | **Carl-bot premium**, hors périmètre |
| Bannière personnalisée | [link](https://carl.gg/get-premium) | 🔶 | **Carl-bot premium**, hors périmètre |
| Limites augmentées | [link](https://carl.gg/get-premium) | 🔶 | **Carl-bot premium**, hors périmètre |

## 13. Monétisation

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Premium (Patreon) | [link](https://carl.gg/get-premium) | 🔶 | **Carl-bot premium**, hors périmètre (bot gratuit/open) |
| Levels (premium) | [link](https://carl.gg/get-premium) | ✅ | `engagement_xp-level` — déjà implémenté (gratuit dans le Bot) |
| Sticky messages | [link](https://carl.gg/get-premium) | ❌ | **À planifier** — pas de messages persistants |

## 14. Annexes Carl-bot (non-features)

| Page | Doc | Note |
|---|---|---|
| Getting Started | [link](https://docs.carl.gg/) | Documentation utilisateur |
| FAQ | [link](https://docs.carl.gg/#/faq) | Documentation utilisateur |
| Premium info | [link](https://carl.gg/get-premium) | N/A — Bot est gratuit et open |
| Server Discovery | [link](https://carl.gg/) | Hors périmètre |

---

## 15. Statistiques finales

| Catégorie | Total | ✅ | 🟡 | ❌ | 🔶 | % couvert |
|---|---:|---:|---:|---:|---:|---:|
| Accueil des membres | 7 | 5 | 1 | 1 | 0 | 71% |
| Engagement — Niveaux & XP | 10 | 9 | 0 | 1 | 0 | 90% |
| Engagement — Giveaways & Sondages | 9 | 8 | 0 | 1 | 0 | 89% |
| Modération | 12 | 12 | 0 | 0 | 0 | 100% |
| Auto-Modération | 9 | 8 | 0 | 1 | 0 | 89% |
| Automatisation | 10 | 6 | 1 | 3 | 0 | 60% |
| Commandes personnalisées | 10 | 7 | 0 | 3 | 0 | 70% |
| Réactions de rôles | 11 | 8 | 0 | 3 | 0 | 73% |
| Notifications sociales | 2 | 0 | 0 | 0 | 2 | 0% |
| Logging & Audit | 9 | 8 | 1 | 0 | 0 | 89% |
| Utilitaires | 17 | 10 | 1 | 6 | 0 | 59% |
| Personnalisation du bot | 4 | 1 | 0 | 0 | 3 | 25% |
| Monétisation | 3 | 1 | 0 | 1 | 1 | 33% |
| **Total** | **113** | **83** | **4** | **20** | **6** | **73%** |

Note : les features marquées 🔶 sont des fonctionnalités premium de Carl-bot qui ne sont pas pertinentes pour un bot auto-hébergé et open source. Elles sont exclues du calcul de couverture pertinent.

## 16. Couverture ajustée (hors premium)

| Catégorie | Total pertinent | ✅ | 🟡 | ❌ | % couvert |
|---|---:|---:|---:|---:|---:|
| Accueil des membres | 7 | 5 | 1 | 1 | 71% |
| Engagement — Niveaux & XP | 10 | 9 | 0 | 1 | 90% |
| Engagement — Giveaways & Sondages | 9 | 8 | 0 | 1 | 89% |
| Modération | 12 | 12 | 0 | 0 | 100% |
| Auto-Modération | 9 | 8 | 0 | 1 | 89% |
| Automatisation | 10 | 6 | 1 | 3 | 60% |
| Commandes personnalisées | 10 | 7 | 0 | 3 | 70% |
| Réactions de rôles | 11 | 8 | 0 | 3 | 73% |
| Logging & Audit | 9 | 8 | 1 | 0 | 89% |
| Utilitaires | 17 | 10 | 1 | 6 | 59% |
| Personnalisation du bot | 1 | 1 | 0 | 0 | 100% |
| Monétisation | 2 | 1 | 0 | 1 | 50% |
| **Total pertinent** | **107** | **83** | **4** | **20** | **78%** |

## 17. Recommandations

### P0 — Critique (impact utilisateur fort, effort faible à modéré)

| Feature | Raison | Effort |
|---|---|:---:|
| Suggestions | Feature communautaire populaire (Carl-bot signature) | 🟡 |
| Starboard | Feature engagement majeure (MEE6/Dyno/Carl-bot) | 🟡 |
| Regex dans triggers | Flexibilité automatisation | 🟢 |
| Anti-attachment spam | Filtre automod manquant | 🟢 |

### P1 — Haute valeur (complétude fonctionnelle)

| Feature | Raison | Effort |
|---|---|:---:|
| Messages répétés (repeating messages) | Feature automation importante | 🟡 |
| Embed builder persistant | Création d'embeds avancée | 🟡 |
| Mode reversed (reaction roles) | Parité Carl-bot | 🟢 |
| Mode binding (reaction roles) | Parité Carl-bot | 🟢 |
| Mode temporary (reaction roles) | Parité Carl-bot | 🟡 |
| Split logs par salon | Granularité logging | 🟢 |

### P2 — Valeur modérée (amélioration progressive)

| Feature | Raison | Effort |
|---|---|:---:|
| Autofeeds | Feature automation | 🟡 |
| Timers (minuteries) | Planification | 🟡 |
| Couleurs de rôles | Gestion rôles avancée | 🟢 |
| Ranks (rangs) | Gestion rôles avancée | 🟡 |
| Text transformation | Fun/UX | 🟢 |
| Compteur de membres (voice) | Feature utilitaire | 🟢 |
| Sticky messages | Messages persistants | 🟡 |
| Leaderboard XP public (web) | Visibilité serveur | 🟡 |

### Résumé des priorités

| Priorité | Features | Phase recommandée | Effort total |
|---|---|:---:|:---:|
| P0 | 4 features | 14-15 | 🟡 |
| P1 | 6 features | 15-16 | 🟡 |
| P2 | 8 features | 16-18 | 🟡 |

## 18. Voir aussi

- [`audit-draftbot.md`](./audit-draftbot.md) — analyse stratégique + roadmap recommandée
- [`draftbot-feature-list.md`](./draftbot-feature-list.md) — extraction similaire pour Draftbot
- [`mee6-feature-list.md`](./mee6-feature-list.md) — extraction similaire pour MEE6
- [`dyno-feature-list.md`](./dyno-feature-list.md) — extraction similaire pour Dyno
- [`migration-impact.md`](./migration-impact.md) — impact sur le schéma DB
- [`../features/`](../features/) — documentation des features Bot déjà livrées
