# MEE6 — Liste exhaustive des features

> **Date** : 2026-08-29
> **Source** : https://help.mee6.xyz/ (sidebar collections)
>
> Ce document liste **toutes** les features publiques MEE6, avec :
> - Le lien direct vers la doc
> - Le status dans le Bot (✅ / 🟡 / ❌ / 🔶)
> - Une référence au commit / module qui l'implémente (ou `docs/plan/` si manquant)

---

## 1. Accueil des membres

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Messages de bienvenue (embed) | [link](https://help.mee6.xyz/en/articles/615144-mee6-welcome-messages) | ✅ | `welcome_welcome` — message embed personnalisable + variables |
| Messages de bienvenue (DM) | [link](https://help.mee6.xyz/en/articles/615144-mee6-welcome-messages) | ✅ | `welcome_welcome` — DM de bienvenue à l'arrivée |
| Messages de départ (goodbye) | [link](https://help.mee6.xyz/en/articles/615146-send-a-message-when-a-user-leaves-the-server-goodbye-message) | ✅ | `welcome_welcome` — événement `guildMemberRemove` |
| Rôles automatiques (welcome role) | [link](https://help.mee6.xyz/en/articles/615145-how-to-give-a-role-to-new-members-welcome-role) | ✅ | `welcome_welcome` — `config.auto_roles` appliqué à l'arrivée |
| Rôle après acceptation règles | [link](https://help.mee6.xyz/en/articles/615145-how-to-give-a-role-to-new-members-welcome-role) | 🟡 | `welcome_welcome` — appliqué à l'arrivée mais pas conditionné aux règles Discord |

## 2. Engagement — Niveaux & XP

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| XP par message | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | ✅ | `engagement_xp-level` — [`docs/features/xp-level.md`](../features/xp-level.md) |
| XP vocal | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | ✅ | `engagement_xp-level` — `voice_sessions` table, XP en vocal |
| Leaderboard (commande) | [link](https://help.mee6.xyz/en/articles/616232-how-to-make-your-mee6-leaderboard-public) | ✅ | `engagement_xp-level` — endpoint `/api/xp/leaderboard` |
| Leaderboard public (web) | [link](https://help.mee6.xyz/en/articles/616232-how-to-make-your-mee6-leaderboard-public) | ❌ | **Planifié** — pas de page web publique de leaderboard |
| Récompenses de rôles (role rewards) | [link](https://help.mee6.xyz/en/articles/616235-mee6-isn-t-giving-role-rewards) | ✅ | `engagement_xp-level` — attribution automatique par palier |
| Empilement des rôles | [link](https://help.mee6.xyz/en/articles/616235-mee6-isn-t-giving-role-rewards) | ✅ | `engagement_xp-level` — stack ou remplacement configurable |
| Messages de level-up | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | ✅ | `engagement_xp-level` + `welcome_cards` — carte SVG de niveau |
| Carte de rang (/rank) | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | ✅ | `welcome_cards` — SVG rank card avec avatar, XP, niveau |
| Taux d'XP configurable | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | ✅ | `engagement_xp-level` — `xp.config.js` multiplicateurs |
| Rôles sans XP (no-XP roles) | [link](https://help.mee6.xyz/en/articles/616233-how-to-fix-server-members-not-getting-xp) | ✅ | `engagement_xp-level` — exclusion par rôle |
| Salons sans XP (no-XP channels) | [link](https://help.mee6.xyz/en/articles/616233-how-to-fix-server-members-not-getting-xp) | ✅ | `engagement_xp-level` — exclusion par salon |
| Don manuel d'XP (/give-xp) | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | ❌ | **À planifier** — pas de commande /give-xp |
| Retrait manuel d'XP (/remove-xp) | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | ❌ | **À planifier** — pas de commande /remove-xp |

## 3. Engagement — Économie & Jeux

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Monnaie virtuelle (coins) | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — [`docs/features/economy.md`](../features/economy.md) |
| Commande /balance | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — solde portefeuille + banque |
| Commande /daily | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — récompense quotidienne avec streak |
| Commande /work | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ❌ | **À planifier** — pas de /work |
| Commande /pay (transfert) | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — transfert avec taxe configurable |
| Boutique (shop) | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — `shop_items`, `shop.service.js` |
| Inventaire (/inventaire) | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — `user_inventory`, `inventory.service.js` |
| Drop d'objets | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — `inventory_drops` table |
| Vente/Don d'objets | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — `inventory_transfers` |
| Leaderboard économie | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ✅ | `engagement_economy` — `/leaderboard` |
| Boosts économiques | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ❌ | **À planifier** — pas de système de boost temporaire |
| Jeux (RPS, devinettes) | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | ❌ | **À planifier** — pas de mini-jeux intégrés |
| Info économie (/economy-info) | [link](https://help.mee6.xyz/en/articles/604799-mee6-economy-plugin-features-and-limitations) | 🟡 | `engagement_economy` — balance seulement, pas d'info boost |

## 4. Engagement — Giveaways & Sondages

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Créer un giveaway | [link](https://help.mee6.xyz/en/articles/619569-mee6-giveaways-plugin-for-discord) | ✅ | `game_engagement` (Phase 5) — [`docs/features/giveaways-polls.md`](../features/giveaways-polls.md) |
| Terminer un giveaway | [link](https://help.mee6.xyz/en/articles/619569-mee6-giveaways-plugin-for-discord) | ✅ | `game_engagement` — `/giveaway-end` |
| Reroll giveaway | [link](https://help.mee6.xyz/en/articles/619569-mee6-giveaways-plugin-for-discord) | ✅ | `game_engagement` — `/giveaway-reroll` |
| Lister les giveaways | [link](https://help.mee6.xyz/en/articles/619569-mee6-giveaways-plugin-for-discord) | ✅ | `game_engagement` — `/giveaway-list` |
| Annuler un giveaway | [link](https://help.mee6.xyz/en/articles/619569-mee6-giveaways-plugin-for-discord) | ✅ | `game_engagement` — `/giveaway-cancel` |
| Participants invisibles | [link](https://help.mee6.xyz/en/articles/619569-mee6-giveaways-plugin-for-discord) | ✅ | `game_engagement` — pas de liste publique (comme MEE6) |
| Cocréation de giveaway | [link](https://help.mee6.xyz/en/articles/619569-mee6-giveaways-plugin-for-discord) | ❌ | **À planifier** — pas de séléction de rôles gagnants |
| Créer un sondage | [link](https://help.mee6.xyz/en/articles/619991-how-to-disable-polls-plugin-and-commands) | ✅ | `game_engagement` (Phase 5) — `/poll-create` |
| Terminer un sondage | [link](https://help.mee6.xyz/en/articles/619991-how-to-disable-polls-plugin-and-commands) | ✅ | `game_engagement` — `/poll-end` |
| Lister les sondages | [link](https://help.mee6.xyz/en/articles/619991-how-to-disable-polls-plugin-and-commands) | ✅ | `game_engagement` — `/poll-list` |
| Supprimer un sondage | [link](https://help.mee6.xyz/en/articles/619991-how-to-disable-polls-plugin-and-commands) | ✅ | `game_engagement` — `/poll-delete` |
| Sondage multi-choix | [link](https://help.mee6.xyz/en/articles/619991-how-to-disable-polls-plugin-and-commands) | ✅ | `game_engagement` — option `multi_choice` |

## 5. Modération

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commande /ban | [link](https://help.mee6.xyz/en/articles/615178-how-to-ban-users-with-mee6) | ✅ | `security_automod` (Phase 1) — [`docs/features/automod.md`](../features/automod.md) |
| Commande /kick | [link](https://help.mee6.xyz/en/articles/615175-how-to-ban-kick-or-mute-with-mee6) | ✅ | `security_automod` |
| Commande /mute | [link](https://help.mee6.xyz/en/articles/615175-how-to-ban-kick-or-mute-with-mee6) | ✅ | `security_automod` — timeout Discord natif |
| Tempban / Tempmute | [link](https://help.mee6.xyz/en/articles/615175-how-to-ban-kick-or-mute-with-mee6) | ✅ | `security_automod` — durée parsée `1w2d3h4m` |
| Filtre de mots interdits | [link](https://help.mee6.xyz/en/articles/615177-how-to-automatically-remove-bad-words-with-mee6) | ✅ | `security_automod` — liste configurable + wildcard |
| Anti-spam | [link](https://help.mee6.xyz/en/articles/615179-automoderator-is-not-punishing-users-correctly) | ✅ | `security_automod` — détection de répétition |
| Anti-lien | [link](https://help.mee6.xyz/en/articles/615177-how-to-automatically-remove-bad-words-with-mee6) | ✅ | `security_automod` — filtrage d'invitations Discord |
| Anti-mention (@everyone) | [link](https://help.mee6.xyz/en/articles/615177-how-to-automatically-remove-bad-words-with-mee6) | ✅ | `security_automod` — mass-mention |
| Anti-caps | [link](https://help.mee6.xyz/en/articles/615177-how-to-automatically-remove-bad-words-with-mee6) | ✅ | `security_automod` — détection de majuscules excessives |
| Sanctions progressives | [link](https://help.mee6.xyz/en/articles/615179-automoderator-is-not-punishing-users-correctly) | ✅ | `security_automod` — warn → mute → ban |
| Immunité de rôles | [link](https://help.mee6.xyz/en/articles/615175-how-to-ban-kick-or-mute-with-mee6) | ✅ | `security_automod` — liste de rôles immunisés |
| Logs d'audit (message delete/edit) | [link](https://help.mee6.xyz/en/articles/615180-how-to-use-audit-logs-to-track-your-members-actions) | ✅ | `security_logs` (Phase 4) — [`docs/features/logs-stats.md`](../features/logs-stats.md) |
| Logs d'audit (join/leave) | [link](https://help.mee6.xyz/en/articles/615180-how-to-use-audit-logs-to-track-your-members-actions) | ✅ | `security_logs` — arrivées/départs |
| Logs d'audit (voice) | [link](https://help.mee6.xyz/en/articles/615180-how-to-use-audit-logs-to-track-your-members-actions) | ✅ | `security_logs` — join/leave vocal |
| Logs d'audit (roles) | [link](https://help.mee6.xyz/en/articles/615180-how-to-use-audit-logs-to-track-your-members-actions) | ✅ | `security_logs` — changements de rôles |
| Logs d'audit (nickname) | [link](https://help.mee6.xyz/en/articles/615180-how-to-use-audit-logs-to-track-your-members-actions) | ✅ | `security_logs` — changements de pseudo |
| Exclusion de salons des logs | [link](https://help.mee6.xyz/en/articles/615180-how-to-use-audit-logs-to-track-your-members-actions) | ✅ | `security_logs` — canaux ignorés configurables |
| Ignorer les actions de bots | [link](https://help.mee6.xyz/en/articles/615180-how-to-use-audit-logs-to-track-your-members-actions) | ✅ | `security_logs` — filtre par source bot |

## 6. Automatisation

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Triggers de mots (word triggers) | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `util_reminders` — [`docs/features/engagement-advanced.md`](../features/engagement-advanced.md) |
| Match exact / contains | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `util_reminders` — `matchType: exact/contains` |
| Exclusion de salons | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `util_reminders` — `excludeChannelIds` |
| Exclusion de rôles | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `util_reminders` — `excludeRoleIds` |
| Cooldown par trigger | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `util_reminders` — `cooldownSeconds` |
| Boutons dans automations | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ❌ | **Planifié Phase 12** — pas de système de boutons automatisés |
| Conditions sur rôles | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | 🟡 | `util_reminders` — exclusion uniquement, pas de condition positive |
| Vérification par message (captcha) | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `security_captcha` — captcha math à l'arrivée |
| Trigger sur gain/perte de rôle | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ❌ | **À planifier** — pas de trigger événementiel |
| Action envoyer message | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `util_reminders` — réponse text/embed |
| Action donner rôle | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ✅ | `util_reminders` — give-role |
| Action retirer rôle | [link](https://help.mee6.xyz/en/articles/618572-getting-started-with-mee6-automations) | ❌ | **À planifier** — pas d'action remove-role |

## 7. Commandes personnalisées

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commandes custom (préfixe !) | [link](https://help.mee6.xyz/en/articles/615091-mee6-bot-variables) | ✅ | `util_reminders` — [`docs/features/engagement-advanced.md`](../features/engagement-advanced.md) |
| Variables {user} | [link](https://help.mee6.xyz/en/articles/615091-mee6-bot-variables) | ✅ | `util_reminders` |
| Variables {server} | [link](https://help.mee6.xyz/en/articles/615091-mee6-bot-variables) | ✅ | `util_reminders` |
| Variables {args} | [link](https://help.mee6.xyz/en/articles/615091-mee6-bot-variables) | ✅ | `util_reminders` |
| Arguments positionnels {1}, {2}... | [link](https://help.mee6.xyz/en/articles/615092-mee6-custom-commands-arguments) | ✅ | `util_reminders` |
| Arguments catch-all {1...} | [link](https://help.mee6.xyz/en/articles/615092-mee6-custom-commands-arguments) | ✅ | `util_reminders` |
| Valeurs par défaut | [link](https://help.mee6.xyz/en/articles/615092-mee6-custom-commands-arguments) | ❌ | **À planifier** — pas de valeur par défaut |
| Réponse embed | [link](https://help.mee6.xyz/en/articles/615092-mee6-custom-commands-arguments) | ✅ | `util_reminders` — `responseEmbedJson` |
| Attribution de rôles | [link](https://help.mee6.xyz/en/articles/615134-how-can-users-gain-roles-from-custom-commands) | ✅ | `util_reminders` — action `give-role` |
| Retrait de rôles | [link](https://help.mee6.xyz/en/articles/615134-how-can-users-gain-roles-from-custom-commands) | ❌ | **À planifier** — pas d'action remove-role |

## 8. Réactions de rôles

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Réactions-Emoji → Rôles | [link](https://help.mee6.xyz/en/articles/620002-how-to-use-reaction-roles-as-a-verification-gate-for-your-community) | ✅ | `community_reaction-roles` (Phase 10) — [`docs/features/reaction-roles.md`](../features/reaction-roles.md) |
| Boutons → Rôles | [link](https://help.mee6.xyz/en/articles/620002-how-to-use-reaction-roles-as-a-verification-gate-for-your-community) | ✅ | `community_reaction-roles` v2 — `InteractiveMessageBuilder` |
| Menus déroulants → Rôles | [link](https://help.mee6.xyz/en/articles/620002-how-to-use-reaction-roles-as-a-verification-gate-for-your-community) | ✅ | `community_reaction-roles` v2 — select menus |
| Gate de vérification | [link](https://help.mee6.xyz/en/articles/620002-how-to-use-reaction-roles-as-a-verification-gate-for-your-community) | ✅ | `community_reaction-roles` — template "Verify" |
| Mode toggle (ajouter/retirer) | [link](https://help.mee6.xyz/en/articles/620002-how-to-use-reaction-roles-as-a-verification-gate-for-your-community) | ✅ | `community_reaction-roles` — ajout et retrait |
| Rôles multiples | [link](https://help.mee6.xyz/en/articles/620002-how-to-use-reaction-roles-as-a-verification-gate-for-your-community) | ✅ | `community_reaction-roles` — plusieurs rôles par message |

## 9. Notifications sociales

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Alertes Twitch (live) | [link](https://help.mee6.xyz/en/articles/620052-mee6-twitch-alerts-for-discord) | 🔶 | **MEE6 premium**, pas pertinent pour un bot interne |
| Alertes YouTube (vidéos) | [link](https://help.mee6.xyz/en/articles/620049-mee6-youtube-alerts-for-discord) | 🔶 | **MEE6 premium**, pas pertinent pour un bot interne |
| Alertes Reddit (posts) | [link](https://help.mee6.xyz/en/articles/620046-mee6-reddit-alerts-for-discord) | 🔶 | **MEE6 premium**, pas pertinent pour un bot interne |
| Alertes RSS | [link](https://help.mee6.xyz/en/articles/620056-mee6-rss-feed-alerts-for-discord) | 🔶 | **MEE6 premium**, pas pertinent pour un bot interne |

## 10. Personnalisation du bot

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Bot personnalisé (nom/avatar) | [link](https://help.mee6.xyz/en/articles/602672-how-to-set-up-mee6-custom-bot) | 🔶 | **MEE6 premium**, hors périmètre (bot auto-hébergé) |
| Backstory IA du bot | [link](https://help.mee6.xyz/en/articles/602672-how-to-set-up-mee6-custom-bot) | 🔶 | **MEE6 premium**, hors périmètre |

## 11. Intelligence Artificielle

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commande /write (texte IA) | [link](https://help.mee6.xyz/en/articles/618492-using-the-write-command) | 🔶 | **MEE6 premium/AI plan**, hors périmètre |
| Commande /imagine (image IA) | [link](https://help.mee6.xyz/en/articles/618507-using-the-imagine-command) | 🔶 | **MEE6 premium/AI plan**, hors périmètre |
| Personnages IA | [link](https://help.mee6.xyz/en/articles/618556-how-to-make-mee6-ai-characters-speak-in-other-languages) | 🔶 | **MEE6 premium/AI plan**, hors périmètre |
| Message quotidien IA | [link](https://help.mee6.xyz/en/articles/618489-mee6-ai-not-working) | ✅ | `community_daily-message` — message IA quotidien via OpenRouter |

## 12. Suivi d'invitations

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Tracking d'invitations | [link](https://help.mee6.xyz/en/articles/619983-mee6-invite-tracker-plugin-for-discord) | ✅ | `util_invites` — [`docs/features/invites.md`](../features/invites.md) |
| Leaderboard d'inviteurs | [link](https://help.mee6.xyz/en/articles/619983-mee6-invite-tracker-plugin-for-discord) | ✅ | `util_invites` — `/invites-leaderboard` |
| Commande /inviter | [link](https://help.mee6.xyz/en/articles/619983-mee6-invite-tracker-plugin-for-discord) | ✅ | `util_invites` — identifie l'inviteur d'un membre |
| Détection de fausses invitations | [link](https://help.mee6.xyz/en/articles/619983-mee6-invite-tracker-plugin-for-discord) | ✅ | `util_invites` — blacklist + détection de comptes fake |
| Bonus d'invitations | [link](https://help.mee6.xyz/en/articles/619983-mee6-invite-tracker-plugin-for-discord) | ✅ | `util_invites` — `invite_bonuses` |
| Restauration d'invitations | [link](https://help.mee6.xyz/en/articles/619983-mee6-invite-tracker-plugin-for-discord) | ✅ | `util_invites` — `invite_restore` |

## 13. Monétisation

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Plans d'abonnement (Stripe) | [link](https://help.mee6.xyz/en/collections/1405837-monetize-plugin) | 🔶 | **MEE6 premium**, hors périmètre (bot gratuit/open) |
| Coupons de réduction | [link](https://help.mee6.xyz/en/articles/634208-server-owner-how-to-use-coupons-with-monetize-stripe) | 🔶 | **MEE6 premium**, hors périmètre |
| Boost XP via abonnement | [link](https://help.mee6.xyz/en/articles/616234-how-to-set-up-levels-plugin) | 🔶 | **MEE6 premium**, hors périmètre |

## 14. Web3 & Blockchain

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Web3 Gating (rôles NFT) | [link](https://help.mee6.xyz/en/articles/620039-how-to-web3-gating) | 🔶 | **MEE6 premium/Web3**, hors périmètre |
| Canaux statistiques Web3 | [link](https://help.mee6.xyz/en/articles/620025-how-to-use-web3-statistic-channels) | 🔶 | **MEE6 premium/Web3**, hors périmètre |
| NFT Pass | [link](https://help.mee6.xyz/en/articles/620042-nft-pass-not-assigning) | 🔶 | **MEE6 premium/Web3**, hors périmètre |

## 15. Utilitaires

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| /serverinfo | [link](https://help.mee6.xyz/en/articles/611783-how-to-find-discord-user-server-id) | ✅ | `util_info` — infos serveur (membres, rôles, salons) |
| /userinfo | [link](https://help.mee6.xyz/en/articles/611783-how-to-find-discord-user-server-id) | ✅ | `util_info` — infos utilisateur (rôles, arrivée) |
| /avatar | [link](https://help.mee6.xyz/en/articles/611783-how-to-find-discord-user-server-id) | ✅ | `util_info` — avatar agrandi |
| Salons vocaux temporaires | [link](https://help.mee6.xyz/en/articles/611780-what-permissions-does-mee6-need) | ✅ | `util_temp-voice` — [`docs/features/temp-voice.md`](../features/temp-voice.md) |
| Rappels (/remind) | [link](https://help.mee6.xyz/en/articles/611784-how-to-delete-messages-on-your-discord-server) | ✅ | `util_reminders` — `/remind` |
| Rôles sécurisés (sticky roles) | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ✅ | `community_sticky-roles` — sauvegarde/restauration des rôles |
| Rappel de bump Disboard | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ✅ | `util_bump-reminder` — rappel automatique 2h après bump |
| Anniversaires | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ✅ | `engagement_birthdays` (Phase 7) — [`docs/features/birthdays.md`](../features/birthdays.md) |
| Signalements (reports) | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ✅ | `community_reports` — [`docs/features/reports.md`](../features/reports.md) |
| Tickets | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ✅ | `community_tickets` (Phase 3) — [`docs/features/tickets.md`](../features/tickets.md) |
| Compteur de membres (voice) | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ❌ | **À planifier** — pas de salon vocal statistique |
| Route de l'Infini (jeu) | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ✅ | `game_road-to-infinite` — jeu de comptage |
| Countdown (jeu) | [link](https://help.mee6.xyz/en/articles/611787-default-settings-for-mee6-plugins) | ✅ | `game_count-down` — jeu de décompte 900→0 |

## 16. Annexes MEE6 (non-features)

| Page | Doc | Note |
|---|---|---|
| Getting Started | [link](https://help.mee6.xyz/en/articles/605731-getting-started-with-mee6) | Hors périmètre (tutoriel utilisateur) |
| Permissions requises | [link](https://help.mee6.xyz/en/articles/611780-what-permissions-does-mee6-need) | Documentation d'installation |
| Variables MEE6 | [link](https://help.mee6.xyz/en/articles/615091-mee6-bot-variables) | Déjà implémenté comme helper côté Bot |
| Markdown Discord | [link](https://help.mee6.xyz/en/articles/611796-discord-markdown-and-mee6) | Documentation utilisateur |
| Plans Premium vs Free | [link](https://help.mee6.xyz/en/articles/710936-mee6-free-vs-premium-plans-comparison) | N/A — Bot est gratuit et open |

---

## 17. Statistiques finales

| Catégorie | Total | ✅ | 🟡 | ❌ | 🔶 | % couvert |
|---|---:|---:|---:|---:|---:|---:|
| Accueil des membres | 5 | 4 | 1 | 0 | 0 | 80% |
| Engagement — Niveaux & XP | 12 | 9 | 0 | 3 | 0 | 75% |
| Engagement — Économie & Jeux | 12 | 7 | 1 | 4 | 0 | 58% |
| Engagement — Giveaways & Sondages | 14 | 13 | 0 | 1 | 0 | 93% |
| Modération | 18 | 18 | 0 | 0 | 0 | 100% |
| Automatisation | 12 | 6 | 1 | 5 | 0 | 50% |
| Commandes personnalisées | 10 | 7 | 0 | 3 | 0 | 70% |
| Réactions de rôles | 6 | 6 | 0 | 0 | 0 | 100% |
| Notifications sociales | 4 | 0 | 0 | 0 | 4 | 0% |
| Personnalisation du bot | 2 | 0 | 0 | 0 | 2 | 0% |
| Intelligence Artificielle | 4 | 1 | 0 | 0 | 3 | 25% |
| Suivi d'invitations | 6 | 6 | 0 | 0 | 0 | 100% |
| Monétisation | 3 | 0 | 0 | 0 | 3 | 0% |
| Web3 & Blockchain | 3 | 0 | 0 | 0 | 3 | 0% |
| Utilitaires | 12 | 10 | 0 | 2 | 0 | 83% |
| **Total** | **123** | **87** | **3** | **18** | **15** | **71%** |

Note : les features marquées 🔶 sont des fonctionnalités premium de MEE6 qui ne sont pas pertinentes pour un bot auto-hébergé et open source. Elles sont exclues du calcul de couverture pertinent.

## 18. Couverture ajustée (hors premium)

| Catégorie | Total pertinent | ✅ | 🟡 | ❌ | % couvert |
|---|---:|---:|---:|---:|---:|
| Accueil des membres | 5 | 4 | 1 | 0 | 80% |
| Engagement — Niveaux & XP | 12 | 9 | 0 | 3 | 75% |
| Engagement — Économie & Jeux | 12 | 7 | 1 | 4 | 58% |
| Engagement — Giveaways & Sondages | 14 | 13 | 0 | 1 | 93% |
| Modération | 18 | 18 | 0 | 0 | 100% |
| Automatisation | 12 | 6 | 1 | 5 | 50% |
| Commandes personnalisées | 10 | 7 | 0 | 3 | 70% |
| Réactions de rôles | 6 | 6 | 0 | 0 | 100% |
| Intelligence Artificielle | 1 | 1 | 0 | 0 | 100% |
| Suivi d'invitations | 6 | 6 | 0 | 0 | 100% |
| Utilitaires | 12 | 10 | 0 | 2 | 83% |
| **Total pertinent** | **108** | **87** | **3** | **18** | **81%** |

## 19. Recommandations

### P0 — Critique (impact utilisateur fort, effort faible à modéré)

| Feature | Raison | Effort |
|---|---|:---:|
| Action retirer rôle (custom cmd + automation) | Compléter l'attribution de rôles | 🟢 |
| Commande /work | Compléter l'économie (daily existe déjà) | 🟢 |
| Leaderboard XP public (web) | Visibilité serveur, feature MEE6 populaire | 🟡 |
| Trigger sur gain/perte de rôle | Automatisations plus puissantes | 🟡 |

### P1 — Haute valeur (complétude fonctionnelle)

| Feature | Raison | Effort |
|---|---|:---:|
| Don manuel d'XP (/give-xp) | Nécessaire pour récompenser événements | 🟢 |
| Retrait manuel d'XP (/remove-xp) | Correction d'abus | 🟢 |
| Boosts économiques | Engagement économique | 🟡 |
| Jeux RPS / guess-the-number | Économie MEE6 incomplète sans jeux | 🟡 |
| Valeurs par défaut (custom cmd) | UX commandes personnalisées | 🟢 |
| Réponse embed enrichie (variables avancées) | Parité MEE6 variables | 🟡 |

### P2 — Valeur modérée (amélioration progressive)

| Feature | Raison | Effort |
|---|---|:---:|
| Boutons dans automations | Seul ❌ bloquant du scope automation | 🟡 |
| Compteur de membres (voice channel) | Feature utilitaire MEE6 | 🟢 |
| Rôle après acceptation règles | Sécurité renforcée | 🟢 |
| Conditions positives (automation) | Au lieu d'exclusion uniquement | 🟡 |
| Regex dans word triggers | Plus de flexibilité | 🟢 |
| Info économie (/economy-info) | Parité commande MEE6 | 🟢 |

### Résumé des priorités

| Priorité | Features | Phase recommandée | Effort total |
|---|---|:---:|:---:|
| P0 | 4 features | 14-15 | 🟡 |
| P1 | 6 features | 15-16 | 🟡 |
| P2 | 6 features | 16-17 | 🟡 |

## 20. Voir aussi

- [`audit-draftbot.md`](./audit-draftbot.md) — analyse stratégique + roadmap recommandée
- [`draftbot-feature-list.md`](./draftbot-feature-list.md) — extraction similaire pour Draftbot
- [`migration-impact.md`](./migration-impact.md) — impact sur le schéma DB
- [`../features/`](../features/) — documentation des features Bot déjà livrées
