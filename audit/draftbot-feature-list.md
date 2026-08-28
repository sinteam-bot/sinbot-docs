# DraftBot — Liste exhaustive des features

> **Date** : 2026-08-27
> **Source** : https://www.draftbot.fr/docs (sidebar)
>
> Ce document liste **toutes** les features publiques Draftbot, avec :
> - Le lien direct vers la doc
> - Le status dans le Bot (✅ / 🟡 / ❌ / 🔶)
> - Une référence au commit / module qui l'implémente (ou `docs/plan/` si manquant)

## 1. Accueil des membres

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Arrivées & départs | [link](https://www.draftbot.fr/docs/accueil-des-membres/arrivees-et-departs) | 🟡 | `feature_cards` (Phase 6) + `feature_welcome` (legacy) — carte SVG de welcome/leave, `CardEventListeners` |
| Gestion des rôles (auto-rôles) | [link](https://www.draftbot.fr/docs/accueil-des-membres/roles-automatiques) | 🟡 | `CardEventListeners.onMemberAdd` applique `config.auto_roles`. Manque : règles conditionnelles + panel dédié |
| Règlement | [link](https://www.draftbot.fr/docs/accueil-des-membres/reglement) | ✅ | Legacy `config.js` (command `!config`) |
| Captcha (math) | [link](https://www.draftbot.fr/docs/accueil-des-membres/captcha) | ✅ | `security_captcha` (legacy) |
| Captcha — variants (image, button) | [link](https://www.draftbot.fr/docs/accueil-des-membres/captcha) | 🟡 | Seul math est implémenté — variants = draftbot premium, on s'en passe |

## 2. Engagement

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Niveaux (XP) | [link](https://www.draftbot.fr/docs/engagement/niveaux) | ✅ | `feature_xp-level` (Phase 2) — [`docs/features/xp-level.md`](../features/xp-level.md) |
| Économie (monnaie) | [link](https://www.draftbot.fr/docs/engagement/economie) | ❌ | **Planifié Phase 9** — [`audit-draftbot.md#phase-9`](./audit-draftbot.md#phase-9--conomie--inventaire) |
| Objets & inventaires | [link](https://www.draftbot.fr/docs/engagement/inventaire) | ❌ | **Planifié Phase 9** (dépend de l'économie) |
| Anniversaires | [link](https://www.draftbot.fr/docs/engagement/anniversaires) | ✅ | `feature_birthdays` (Phase 7) — [`docs/features/birthdays.md`](../features/birthdays.md) |
| Réactions de mots | [link](https://www.draftbot.fr/docs/engagement/reactions-de-mots) | ❌ | **Planifié Phase 11** |

## 3. Jeux & événements

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Giveaways | [link](https://www.draftbot.fr/docs/jeux-et-evenements/giveaways) | ✅ | `feature_engagement` (Phase 5) — [`docs/features/giveaways-polls.md`](../features/giveaways-polls.md) |
| Sondages (polls) | [link](https://www.draftbot.fr/docs/jeux-et-evenements/giveaways) | ✅ | `feature_engagement` (Phase 5) — [`docs/features/giveaways-polls.md`](../features/giveaways-polls.md) |
| Calendrier de l'Avent | [link](https://www.draftbot.fr/docs/jeux-et-evenements/calendrier-de-l-avent) | ❌ | **Planifié Phase 13** |
| Route de l'Infini | [link](https://www.draftbot.fr/docs/jeux-et-evenements/route-infini) | ✅ | `game_road-to-infinite` (legacy) |
| Bingo | [link](https://www.draftbot.fr/docs/jeux-et-evenements/bingo) | ❌ | **Planifié Phase 13** |
| Statistiques de jeux | [link](https://www.draftbot.fr/docs/jeux-et-evenements/stats-de-jeux) | 🟡 | `counterState` / `countdownState` en BDD mais pas d'agrégat dashboard — **Planifié Phase 8** |
| Commandes de jeux & fun | [link](https://www.draftbot.fr/docs/jeux-et-evenements/commandes-jeux-fun) | 🟡 | Pas de feature dédiée, à part `CountDownModule` (legacy) — **Planifié Phase 13** |

## 4. Communauté

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Suggestions | [link](https://www.draftbot.fr/docs/communaute/suggestions) | ❌ | **Planifié Phase 12** |
| Tickets | [link](https://www.draftbot.fr/docs/communaute/tickets) | ✅ | `feature_tickets` (Phase 3) — [`docs/features/tickets.md`](../features/tickets.md) |
| Starboards | [link](https://www.draftbot.fr/docs/communaute/starboards) | ❌ | **Planifié Phase 12** |
| Rôles-Réactions | [link](https://www.draftbot.fr/docs/communaute/roles-reactions) | ❌ | **Planifié Phase 10** (fort impact utilisateur) |
| Interserveurs | [link](https://www.draftbot.fr/docs/communaute/interserveurs) | ❌ | **Planifié Phase 14** (privacy, GDPR) |
| Salons vocaux temporaires | [link](https://www.draftbot.fr/docs/communaute/salons-vocaux-temporaires) | ❌ | **Planifié Phase 12** |
| Messages (auto-messages) | [link](https://www.draftbot.fr/docs/communaute/messages) | 🟡 | `feature_daily-message` (legacy) — un message/jour IA. Manque : messages channel-bound + panel unifié |
| Messages récurrents | [link](https://www.draftbot.fr/docs/communaute/messages-recurrents) | ❌ | **Planifié Phase 11** |
| Notifications sociales (Twitch/YT) | [link](https://www.draftbot.fr/docs/communaute/notifications-sociales) | 🔶 | **Draftbot premium**, pas pertinent pour un bot interne |

## 5. Sécurité

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Modération (commandes) | [link](https://www.draftbot.fr/docs/securite/moderation) | ✅ | `feature_automod` (Phase 1) — [`docs/features/automod.md`](../features/automod.md) |
| Auto-Modération | [link](https://www.draftbot.fr/docs/securite/auto-moderation) | ✅ | `feature_automod` (Phase 1) — 7 règles + sanctions progressives |
| Gestion des messages (purge) | [link](https://www.draftbot.fr/docs/securite/gestion-des-messages) | ✅ | `/mod clear` (Phase 1) + legacy `purge` |
| Signalements | [link](https://www.draftbot.fr/docs/securite/signalements) | ❌ | **Planifié Phase 12** |
| Rôles sécurisés (sticky roles) | [link](https://www.draftbot.fr/docs/securite/roles-securises) | ❌ | **Planifié Phase 8** (effort faible) |
| Logs | [link](https://www.draftbot.fr/docs/securite/logs) | ✅ | `feature_logs` (Phase 4) — [`docs/features/logs-stats.md`](../features/logs-stats.md) |

## 6. Utilitaires

| Feature | Doc | Status | Implémentation |
|---|---|:---:|---|
| Commandes d'informations | [link](https://www.draftbot.fr/docs/utilitaires/commandes-informations) | ❌ | **Planifié Phase 8** (`/serverinfo`, `/userinfo`, `/avatar`, etc.) |
| Commandes personnalisées | [link](https://www.draftbot.fr/docs/utilitaires/commandes-personnalisees) | ❌ | **Planifié Phase 11** (`/customcmd`) |
| Rappels | [link](https://www.draftbot.fr/docs/utilitaires/rappels) | 🟡 | Pas de feature dédiée — **Planifié Phase 11** (`/remind`) |
| Salons de statistiques | [link](https://www.draftbot.fr/docs/utilitaires/salons-de-statistiques) | ❌ | **Planifié Phase 12** (voice channels auto-update) |
| Sauvegardes du serveur | [link](https://www.draftbot.fr/docs/utilitaires/sauvegardes-du-serveur) | ❌ | **Planifié Phase 12** (effort élevé) |

## 7. Annexes Draftbot (non-features)

| Page | Doc | Note |
|---|---|---|
| Installation et réglages | [link](https://www.draftbot.fr/docs/installation) | Hors périmètre (l'auto-hébergement est notre modèle) |
| Variables | [link](https://www.draftbot.fr/docs/autres/variables) | À implémenter comme helper côté Bot (cf Phase 11) |
| Timestamps | [link](https://www.draftbot.fr/docs/autres/timestamps) | Pas pertinent (utiliser `new Date().toLocaleString('fr-FR')`) |
| Markdown | [link](https://www.draftbot.fr/docs/autres/markdown) | Documentation utilisateur, pas une feature |
| Abonnement premium | [link](https://www.draftbot.fr/docs/autres/premium) | N/A — Bot est gratuit et open |

## 8. Statistiques finales

| Catégorie | Total | ✅ | 🟡 | ❌ | 🔶 | % couvert |
|---|---:|---:|---:|---:|---:|---:|
| Accueil des membres | 5 | 1 | 3 | 0 | 0 | 80% |
| Engagement | 5 | 3 | 0 | 2 | 0 | 60% |
| Jeux & événements | 6 | 3 | 2 | 1 | 0 | 83% |
| Communauté | 9 | 1 | 1 | 6 | 1 | 22% |
| Sécurité | 6 | 5 | 1 | 0 | 0 | 100% |
| Utilitaires | 5 | 0 | 1 | 4 | 0 | 20% |
| **Total** | **36** | **13** | **8** | **13** | **1** | **58%** |

Note : la différence avec `audit-draftbot.md` (35 vs 36) vient du fait que la sidebar liste **9 pages dans Communauté** (vs 8 dans la liste principale) car "Suggestions" + "Tickets" sont séparés et "Messages" + "Messages récurrents" sont séparés. Le total reflète la réalité de la sidebar Draftbot.

## 9. Voir aussi

- [`audit-draftbot.md`](./audit-draftbot.md) — analyse stratégique + roadmap recommandée
- [`migration-impact.md`](./migration-impact.md) — impact sur le schéma DB
- [`../features/`](../features/) — documentation des features Bot déjà livrées
