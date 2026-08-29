# Dyno — Liste exhaustive des features

> **Date** : 2026-08-29
> **Source** : https://docs.dyno.gg/en/modules (sidebar)
>
> Ce document liste **toutes** les features publiques Dyno, avec :
> - Le lien direct vers la doc
> - Le status dans le Bot (✅ / 🟡 / ❌ / 🔶)
> - Une référence au commit / module qui l'implémente (ou `docs/plan/` si manquant)

---

## 1. Accueil des membres

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Messages de bienvenue (embed) | [link](https://docs.dyno.gg/en/modules/welcome) | ✅ | `welcome_welcome` — message embed personnalisable |
| Messages de bienvenue (DM) | [link](https://docs.dyno.gg/en/modules/welcome) | ✅ | `welcome_welcome` — DM de bienvenue |
| Messages de bienvenue (carte) | [link](https://docs.dyno.gg/en/modules/welcome) | ✅ | `welcome_cards` — carte SVG de bienvenue |
| Messages d'annonce (join/leave/ban) | [link](https://docs.dyno.gg/en/modules/announcements) | ✅ | `welcome_welcome` — événements join/leave |
| Rôles automatiques (on join) | [link](https://docs.dyno.gg/en/modules/autoroles) | ✅ | `welcome_welcome` — `config.auto_roles` |
| Rôles temporisés (timed roles) | [link](https://docs.dyno.gg/en/modules/autoroles) | ❌ | **À planifier** — pas de rôles temporisés |
| Rangs rejoignables (joinable ranks) | [link](https://docs.dyno.gg/en/modules/autoroles) | ❌ | **À planifier** — pas de système de rangs |

## 2. Engagement — Niveaux & XP

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| XP par message | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` — [`docs/features/xp-level.md`](../features/xp-level.md) |
| XP vocal | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` — `voice_sessions` |
| Leaderboard | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` — endpoint `/api/xp/leaderboard` |
| Leaderboard public (web) | [link](https://docs.dyno.gg/en/modules/levels) | ❌ | **À planifier** — pas de page web publique |
| Récompenses de rôles | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` — attribution par palier |
| Messages de level-up | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` + `welcome_cards` |
| Carte de rang (/rank) | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `welcome_cards` — SVG rank card |
| Taux d'XP configurable | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` — `xp.config.js` |
| Rôles sans XP | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` — exclusion par rôle |
| Salons sans XP | [link](https://docs.dyno.gg/en/modules/levels) | ✅ | `engagement_xp-level` — exclusion par salon |

## 3. Engagement — Économie & Jeux

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Monnaie globale (Dyno Coins) | [link](https://docs.dyno.gg/en/modules/economy) | ✅ | `engagement_economy` — [`docs/features/economy.md`](../features/economy.md) |
| Commande /daily | [link](https://docs.dyno.gg/en/modules/economy) | ✅ | `engagement_economy` — récompense quotidienne avec streak |
| Commande /work | [link](https://docs.dyno.gg/en/modules/economy) | ❌ | **À planifier** — pas de /work |
| Commande /balance | [link](https://docs.dyno.gg/en/modules/economy) | ✅ | `engagement_economy` — solde portefeuille + banque |
| Commande /pay | [link](https://docs.dyno.gg/en/modules/economy) | ✅ | `engagement_economy` — transfert avec taxe |
| Boutique (shop) | [link](https://docs.dyno.gg/en/modules/economy) | ✅ | `engagement_economy` — `shop_items` |
| Inventaire | [link](https://docs.dyno.gg/en/modules/economy) | ✅ | `engagement_economy` — `user_inventory` |
| Leaderboard économie | [link](https://docs.dyno.gg/en/modules/economy) | ✅ | `engagement_economy` — `/leaderboard` |
| Boosts économiques | [link](https://docs.dyno.gg/en/modules/economy) | ❌ | **À planifier** — pas de boost temporaire |
| Jeux fun (RPS, etc.) | [link](https://docs.dyno.gg/en/modules/fun) | ❌ | **À planifier** — pas de mini-jeux |

## 4. Engagement — Giveaways & Sondages

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Créer un giveaway | [link](https://docs.dyno.gg/en/modules/giveaways) | ✅ | `game_engagement` (Phase 5) — [`docs/features/giveaways-polls.md`](../features/giveaways-polls.md) |
| Terminer un giveaway | [link](https://docs.dyno.gg/en/modules/giveaways) | ✅ | `game_engagement` — `/giveaway-end` |
| Reroll giveaway | [link](https://docs.dyno.gg/en/modules/giveaways) | ✅ | `game_engagement` — `/giveaway-reroll` |
| Lister les giveaways | [link](https://docs.dyno.gg/en/modules/giveaways) | ✅ | `game_engagement` — `/giveaway-list` |
| Annuler un giveaway | [link](https://docs.dyno.gg/en/modules/giveaways) | ✅ | `game_engagement` — `/giveaway-cancel` |
| Giveaway partageable (social) | [link](https://docs.dyno.gg/en/modules/giveaways) | ❌ | **À planifier** — pas de partage social |
| Créer un sondage (/poll) | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `game_engagement` — `/poll-create` |
| Sondage multi-choix | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `game_engagement` — option `multi_choice` |

## 5. Modération

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commande /ban | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` (Phase 1) — [`docs/features/automod.md`](../features/automod.md) |
| Commande /kick | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` |
| Commande /mute | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` — timeout Discord natif |
| Commande /warn | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` — `user_warnings` |
| Tempban / Tempmute | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` — durée parsée |
| Autopunish (sanctions auto) | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` — warn → mute → ban progressif |
| Casiers judiciaires (modlogs) | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_logs` — [`docs/features/logs-stats.md`](../features/logs-stats.md) |
| Immunité de rôles | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` — rôles protégés |
| Purge de messages | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `security_automod` — `/mod clear` |
| Slowmode | [link](https://docs.dyno.gg/en/modules/slowmode) | ❌ | **À planifier** — pas de slowmode automatique |

## 6. Auto-Modération

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Filtre de mots interdits | [link](https://docs.dyno.gg/en/modules/automod) | ✅ | `security_automod` — liste configurable |
| Anti-spam | [link](https://docs.dyno.gg/en/modules/automod) | ✅ | `security_automod` — détection de répétition |
| Anti-lien | [link](https://docs.dyno.gg/en/modules/automod) | ✅ | `security_automod` — filtrage d'invitations |
| Anti-mention (@everyone) | [link](https://docs.dyno.gg/en/modules/automod) | ✅ | `security_automod` — mass-mention |
| Anti-caps | [link](https://docs.dyno.gg/en/modules/automod) | ✅ | `security_automod` — majuscules excessives |
| Anti-zalgo | [link](https://docs.dyno.gg/en/modules/automod) | ❌ | **À planifier** — pas de filtre zalgo |
| Anti-sticker | [link](https://docs.dyno.gg/en/modules/automod) | ❌ | **À planifier** — pas de filtre sticker |
| Auto Delete (filtres par salon) | [link](https://docs.dyno.gg/en/modules/autodelete) | 🟡 | `security_automod` — suppression auto basique |
| Autoban (règles nouveaux membres) | [link](https://docs.dyno.gg/en/modules/autoban) | ❌ | **À planifier** — pas de règles automatiques nouveaux membres |
| Whitelist/blacklist de liens | [link](https://docs.dyno.gg/en/modules/automod) | ✅ | `security_automod` — listes configurables |

## 7. Automatisation

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Autoresponder (text triggers) | [link](https://docs.dyno.gg/en/modules/autoresponder) | ✅ | `util_reminders` — [`docs/features/engagement-advanced.md`](../features/engagement-advanced.md) |
| Match exact / contains | [link](https://docs.dyno.gg/en/modules/autoresponder) | ✅ | `util_reminders` — `matchType: exact/contains` |
| Regex triggers | [link](https://docs.dyno.gg/en/modules/autoresponder) | 🟡 | `util_reminders` — regex prévu mais non activé |
| Exclusion de salons | [link](https://docs.dyno.gg/en/modules/autoresponder) | ✅ | `util_reminders` — `excludeChannelIds` |
| Exclusion de rôles | [link](https://docs.dyno.gg/en/modules/autoresponder) | ✅ | `util_reminders` — `excludeRoleIds` |
| Cooldown par trigger | [link](https://docs.dyno.gg/en/modules/autoresponder) | ✅ | `util_reminders` — `cooldownSeconds` |
| Réponse embed | [link](https://docs.dyno.gg/en/modules/autoresponder) | ✅ | `util_reminders` — `responseEmbedJson` |
| Auto Message (messages programmés) | [link](https://docs.dyno.gg/en/modules/automessage) | ❌ | **À planifier** — pas de messages programmés |
| Auto Purge (purge programmée) | [link](https://docs.dyno.gg/en/modules/autopurge) | ❌ | **À planifier** — pas de purge programmée |

## 8. Commandes personnalisées

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commandes custom (préfixe) | [link](https://docs.dyno.gg/en/modules/customcommands) | ✅ | `util_reminders` — [`docs/features/engagement-advanced.md`](../features/engagement-advanced.md) |
| Variables utilisateur | [link](https://docs.dyno.gg/en/modules/customcommands) | ✅ | `util_reminders` — `{user}`, `{server}` |
| Variables de serveur | [link](https://docs.dyno.gg/en/modules/customcommands) | ✅ | `util_reminders` — `{server}` |
| Variables XP/économie | [link](https://docs.dyno.gg/en/modules/customcommands) | ❌ | **À planifier** — pas de variables `{user.level}` |
| Réponse embed | [link](https://docs.dyno.gg/en/modules/customcommands) | ✅ | `util_reminders` — `responseEmbedJson` |
| Cooldown | [link](https://docs.dyno.gg/en/modules/customcommands) | ✅ | `util_reminders` — `cooldownSeconds` |
| Activation/désactivation | [link](https://docs.dyno.gg/en/modules/customcommands) | ✅ | `util_reminders` — toggle on/off |
| Tags (commandes courtes) | [link](https://docs.dyno.gg/en/modules/tags) | ❌ | **À planifier** — pas de système de tags |

## 9. Réactions de rôles

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Réactions-Emoji → Rôles | [link](https://docs.dyno.gg/en/modules/reactionroles) | ✅ | `community_reaction-roles` (Phase 10) — [`docs/features/reaction-roles.md`](../features/reaction-roles.md) |
| Boutons → Rôles | [link](https://docs.dyno.gg/en/modules/reactionroles) | ✅ | `community_reaction-roles` v2 — `InteractiveMessageBuilder` |
| Menus déroulants → Rôles | [link](https://docs.dyno.gg/en/modules/reactionroles) | ✅ | `community_reaction-roles` v2 — select menus |
| Mode toggle (ajouter/retirer) | [link](https://docs.dyno.gg/en/modules/reactionroles) | ✅ | `community_reaction-roles` — ajout et retrait |
| Rôles multiples par message | [link](https://docs.dyno.gg/en/modules/reactionroles) | ✅ | `community_reaction-roles` — plusieurs rôles |

## 10. Notifications sociales

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Alertes Twitch (live) | [link](https://docs.dyno.gg/en/modules/twitch) | 🔶 | **Dyno premium**, pas pertinent pour un bot interne |
| Alertes YouTube (vidéos) | [link](https://docs.dyno.gg/en/modules/youtube) | 🔶 | **Dyno premium**, pas pertinent pour un bot interne |
| Alertes TikTok | [link](https://docs.dyno.gg/en/modules/tiktok) | 🔶 | **Dyno premium**, pas pertinent pour un bot interne |
| Alertes Kick (streaming) | [link](https://docs.dyno.gg/en/modules/kick) | 🔶 | **Dyno premium**, pas pertinent pour un bot interne |
| Alertes Reddit (posts) | [link](https://docs.dyno.gg/en/modules/reddit) | 🔶 | **Dyno premium**, pas pertinent pour un bot interne |

## 11. Logging & Audit

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Action Log (message delete/edit) | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` (Phase 4) — [`docs/features/logs-stats.md`](../features/logs-stats.md) |
| Action Log (join/leave) | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` — arrivées/départs |
| Action Log (voice) | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` — join/leave vocal |
| Action Log (roles) | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` — changements de rôles |
| Action Log (nickname) | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` — changements de pseudo |
| Action Log (ban/kick) | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` — modérations |
| Exclusion de salons | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` — canaux ignorés |
| Ignorer les actions de bots | [link](https://docs.dyno.gg/en/modules/actionlog) | ✅ | `security_logs` — filtre par source bot |
| Pagination (Components V2) | [link](https://docs.dyno.gg/en/modules/actionlog) | 🟡 | `security_logs` — logs basiques, pas de pagination embed |

## 12. Utilitaires

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| AFK (statut d'absence) | [link](https://docs.dyno.gg/en/modules/afk) | ❌ | **À planifier** — pas de système AFK |
| AFK (laisser un message) | [link](https://docs.dyno.gg/en/modules/afk) | ❌ | **À planifier** — pas de messagerie AFK |
| AFK (notification retour) | [link](https://docs.dyno.gg/en/modules/afk) | ❌ | **À planifier** — pas de notification retour |
| Rappels (/remindme) | [link](https://docs.dyno.gg/en/modules/reminders) | ✅ | `util_reminders` — `/remind` |
| Message Embedder | [link](https://docs.dyno.gg/en/modules/messageembedder) | ❌ | **À planifier** — pas d'embed builder persistant |
| Forms (formulaires) | [link](https://docs.dyno.gg/en/modules/forms) | ❌ | **À planifier** — pas de système de formulaires |
| Highlights (notifications mots-clés) | [link](https://docs.dyno.gg/en/modules/highlights) | ❌ | **À planifier** — pas de notifications mots-clés |
| Starboard | [link](https://docs.dyno.gg/en/modules/starboard) | ❌ | **Planifié Phase 12** — pas de starboard |
| Tickets | [link](https://docs.dyno.gg/en/modules/tickets) | ✅ | `community_tickets` (Phase 3) — [`docs/features/tickets.md`](../features/tickets.md) |
| Tickets (panneau personnalisé) | [link](https://docs.dyno.gg/en/modules/tickets) | ✅ | `community_tickets` — panel avec formulaire |
| Tickets (transcript) | [link](https://docs.dyno.gg/en/modules/tickets) | ✅ | `community_tickets` — transcript HTML |
| Tickets (rôles ping) | [link](https://docs.dyno.gg/en/modules/tickets) | ✅ | `community_tickets` — rôles notifiés |
| Salons vocaux temporaires | [link](https://docs.dyno.gg/en/modules/voicetextlinking) | ✅ | `util_temp-voice` — [`docs/features/temp-voice.md`](../features/temp-voice.md) |
| Voice Text Linking | [link](https://docs.dyno.gg/en/modules/voicetextlinking) | ✅ | `util_temp-voice` — texte lié au vocal |
| Rôles sécurisés (sticky roles) | [link](https://docs.dyno.gg/en/modules/autoroles) | ✅ | `community_sticky-roles` — sauvegarde/restauration |
| Anniversaires | [link](https://docs.dyno.gg/en/modules/announcements) | ✅ | `engagement_birthdays` (Phase 7) — [`docs/features/birthdays.md`](../features/birthdays.md) |
| Signalements (reports) | [link](https://docs.dyno.gg/en/modules/moderation) | ✅ | `community_reports` — [`docs/features/reports.md`](../features/reports.md) |
| Compteur de membres (voice) | [link](https://docs.dyno.gg/en/modules/autoroles) | ❌ | **À planifier** — pas de salon vocal statistique |
| Route de l'Infini (jeu) | [link](https://docs.dyno.gg/en/modules/fun) | ✅ | `game_road-to-infinite` — jeu de comptage |
| Countdown (jeu) | [link](https://docs.dyno.gg/en/modules/fun) | ✅ | `game_count-down` — jeu de décompte |
| Fun commands (memes, etc.) | [link](https://docs.dyno.gg/en/modules/fun) | ❌ | **À planifier** — pas de commandes fun/meme |

## 13. Personnalisation du bot

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Nickname personnalisé | [link](https://docs.dyno.gg/en/dashboard/settings) | ✅ | Configuration Discord native |
| Préfixe personnalisé | [link](https://docs.dyno.gg/en/dashboard/settings) | ✅ | `config.chienne.yml` — préfixe configurable |
| Avatar personnalisé | [link](https://docs.dyno.gg/premium) | 🔶 | **Dyno Custom premium**, hors périmètre |
| Nom personnalisé | [link](https://docs.dyno.gg/premium) | 🔶 | **Dyno Custom premium**, hors périmètre |
| Statut personnalisé | [link](https://docs.dyno.gg/premium) | 🔶 | **Dyno Custom premium**, hors périmètre |
| Localisation (langue du bot) | [link](https://docs.dyno.gg/en/whats-new) | ❌ | **À planifier** — pas de multi-langue |

## 14. Monétisation

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Premium (abononnement) | [link](https://docs.dyno.gg/premium) | 🔶 | **Dyno premium**, hors périmètre (bot gratuit/open) |
| Battle Pass | [link](https://docs.dyno.gg/premium) | 🔶 | **Dyno premium**, hors périmètre |
| Dyno Membership | [link](https://docs.dyno.gg/premium) | 🔶 | **Dyno premium**, hors périmètre |

## 15. Annexes Dyno (non-features)

| Page | Doc | Note |
|---|---|---|
| Dashboard | [link](https://docs.dyno.gg/en/dashboard) | Interface web, hors périmètre (auto-hébergement) |
| FAQ | [link](https://docs.dyno.gg/en/faq) | Documentation utilisateur |
| Premium info | [link](https://docs.dyno.gg/premium) | N/A — Bot est gratuit et open |
| Releases | [link](https://docs.dyno.gg/en/whats-new) | Historique de versions Dyno |

---

## 16. Statistiques finales

| Catégorie | Total | ✅ | 🟡 | ❌ | 🔶 | % couvert |
|---|---:|---:|---:|---:|---:|---:|
| Accueil des membres | 7 | 5 | 0 | 2 | 0 | 71% |
| Engagement — Niveaux & XP | 10 | 9 | 0 | 1 | 0 | 90% |
| Engagement — Économie & Jeux | 10 | 6 | 0 | 4 | 0 | 60% |
| Engagement — Giveaways & Sondages | 8 | 7 | 0 | 1 | 0 | 88% |
| Modération | 10 | 9 | 0 | 1 | 0 | 90% |
| Auto-Modération | 10 | 7 | 0 | 3 | 0 | 70% |
| Automatisation | 9 | 6 | 1 | 2 | 0 | 67% |
| Commandes personnalisées | 8 | 6 | 0 | 2 | 0 | 75% |
| Réactions de rôles | 5 | 5 | 0 | 0 | 0 | 100% |
| Notifications sociales | 5 | 0 | 0 | 0 | 5 | 0% |
| Logging & Audit | 9 | 8 | 1 | 0 | 0 | 89% |
| Utilitaires | 20 | 12 | 0 | 8 | 0 | 60% |
| Personnalisation du bot | 6 | 2 | 0 | 1 | 3 | 33% |
| Monétisation | 3 | 0 | 0 | 0 | 3 | 0% |
| **Total** | **120** | **82** | **2** | **25** | **11** | **68%** |

Note : les features marquées 🔶 sont des fonctionnalités premium de Dyno qui ne sont pas pertinentes pour un bot auto-hébergé et open source. Elles sont exclues du calcul de couverture pertinent.

## 17. Couverture ajustée (hors premium)

| Catégorie | Total pertinent | ✅ | 🟡 | ❌ | % couvert |
|---|---:|---:|---:|---:|---:|
| Accueil des membres | 7 | 5 | 0 | 2 | 71% |
| Engagement — Niveaux & XP | 10 | 9 | 0 | 1 | 90% |
| Engagement — Économie & Jeux | 10 | 6 | 0 | 4 | 60% |
| Engagement — Giveaways & Sondages | 8 | 7 | 0 | 1 | 88% |
| Modération | 10 | 9 | 0 | 1 | 90% |
| Auto-Modération | 10 | 7 | 0 | 3 | 70% |
| Automatisation | 9 | 6 | 1 | 2 | 67% |
| Commandes personnalisées | 8 | 6 | 0 | 2 | 75% |
| Réactions de rôles | 5 | 5 | 0 | 0 | 100% |
| Logging & Audit | 9 | 8 | 1 | 0 | 89% |
| Utilitaires | 20 | 12 | 0 | 8 | 60% |
| Personnalisation du bot | 3 | 2 | 0 | 1 | 67% |
| **Total pertinent** | **109** | **82** | **2** | **25** | **75%** |

## 18. Recommandations

### P0 — Critique (impact utilisateur fort, effort faible à modéré)

| Feature | Raison | Effort |
|---|---|:---:|
| Commande /work | Compléter l'économie (daily existe déjà) | 🟢 |
| Regex dans autoresponder | Flexibilité triggers | 🟢 |
| Slowmode automatique | Feature modération basique | 🟢 |
| Tags (commandes courtes) | UX commandes personnalisées | 🟢 |

### P1 — Haute valeur (complétude fonctionnelle)

| Feature | Raison | Effort |
|---|---|:---:|
| Auto Message (messages programmés) | Feature automation importante | 🟡 |
| Auto Purge (purge programmée) | Modération proactive | 🟢 |
| AFK (statut d'absence) | Feature communautaire populaire | 🟡 |
| Variables XP/économie (custom cmd) | Parité variables MEE6/Dyno | 🟡 |
| Starboard | Feature communautaire MEE6/Dyno | 🟡 |
| Message Embedder | Création d'embeds persistants | 🟡 |

### P2 — Valeur modérée (amélioration progressive)

| Feature | Raison | Effort |
|---|---|:---:|
| Autoban (règles nouveaux membres) | Sécurité renforcée | 🟡 |
| Anti-zalgo | Filtre automod manquant | 🟢 |
| Anti-sticker | Filtre automod manquant | 🟢 |
| Auto Delete avancé | Filtres par salon | 🟡 |
| Forms (formulaires) | Feature engagement | 🟡 |
| Highlights (notifications mots-clés) | Feature engagement | 🟡 |
| Fun commands (memes, etc.) | Divertissement | 🟢 |
| Compteur de membres (voice) | Feature utilitaire | 🟢 |
| Localisation (langue) | Accessibilité | 🟡 |
| Leaderboard XP public (web) | Visibilité serveur | 🟡 |

### Résumé des priorités

| Priorité | Features | Phase recommandée | Effort total |
|---|---|:---:|:---:|
| P0 | 4 features | 14-15 | 🟢 |
| P1 | 6 features | 15-16 | 🟡 |
| P2 | 10 features | 16-18 | 🟡 |

## 19. Voir aussi

- [`audit-draftbot.md`](./audit-draftbot.md) — analyse stratégique + roadmap recommandée
- [`draftbot-feature-list.md`](./draftbot-feature-list.md) — extraction similaire pour Draftbot
- [`mee6-feature-list.md`](./mee6-feature-list.md) — extraction similaire pour MEE6
- [`migration-impact.md`](./migration-impact.md) — impact sur le schéma DB
- [`../features/`](../features/) — documentation des features Bot déjà livrées
