# Audit — Comparaison DraftBot vs Chienne

> **Date** : 2026-08-27
> **Source** : https://www.draftbot.fr/docs
> **Périmètre** : toutes les features publiques listées dans la sidebar de Draftbot

## 1. Méthodologie

Chaque feature Draftbot est évaluée selon 3 critères :
- **Status** : ✅ livrée | 🟡 partielle | ❌ manquante | 🔶 premium-only
- **Effort** : 🟢 faible (≤ 2j) | 🟡 moyen (3-5j) | 🔴 élevé (> 5j)
- **Notes** : particularités, écarts, ou références

## 2. Résumé exécutif

| Catégorie | Total | ✅ Livré | 🟡 Partiel | ❌ Manquant | 🔶 Premium |
|---|---:|---:|---:|---:|---:|
| Accueil des membres | 5 | 4 | 1 | 0 | 0 |
| Engagement | 5 | 4 | 0 | 1 | 0 |
| Jeux & événements | 6 | 3 | 1 | 2 | 0 |
| Communauté | 8 | 1 | 1 | 6 | 1 |
| Sécurité | 6 | 5 | 1 | 0 | 0 |
| Utilitaires | 5 | 0 | 1 | 4 | 0 |
| **Total** | **35** | **17** | **5** | **13** | **1** |

**Taux de couverture Draftbot : ~63% (22/35)** — solide, mais il reste 13 features à implémenter pour atteindre la parité.

## 3. Détail par catégorie

### 3.1 Accueil des membres

| Feature | Status | Effort | Notes |
|---|:---:|:---:|---|
| Arrivées & départs | 🟡 | 🟢 | Welcome existe (Phase 6), leave aussi via `CardEventListeners`. Manque : le panneau de config dédié côté panel web est sommaire |
| Gestion des rôles (auto-rôles) | 🟡 | 🟡 | Le code legacy `welcome.AUTO_ROLES` + `CardEventListeners` ajoute les rôles à l'arrivée. Manque : règles conditionnelles (rôle X si Y) + un panel dédié |
| Règlement | ✅ | 🟢 | `/config` (legacy) permet de définir une chaîne + un message. Pas d'acceptance screen, mais conforme à la spec basique |
| Captcha | ✅ | 🟢 | `security_captcha` (legacy) + `feature_automod` (Phase 1) a déjà une couche anti-raid. Le captcha math est fonctionnel |
| Captcha — variants (image, bouton) | 🟡 | 🟡 | Seul le captcha math est implémenté. Variants (image CAPTCHA, bouton "Je ne suis pas un robot") = premium Draftbot, on peut s'en passer |

### 3.2 Engagement

| Feature | Status | Effort | Notes |
|---|:---:|:---:|---|
| Niveaux (XP) | ✅ | — | Phase 2 — complet avec LevelUpService, rôles auto, leaderboard paginé, config deep-merge |
| Économie (monnaie virtuelle) | ❌ | 🔴 | **Manquant**. Pas de table `user_money`, pas de service. Impacte aussi le système d'inventaire (qui s'appuie dessus) |
| Objets & inventaires | ❌ | 🔴 | **Manquant** (dépend de l'économie). Draftbot a 12+ sub-commands : `/inventaire`, `/objet donner/vendre/échanger/drop`, `/dropobjet`, `/topitems`, `/admininventaire *` |
| Anniversaires | ✅ | — | Phase 7 — complet, conforme à la spec Draftbot (cooldown 1j/2j/6m/1an, mode public/privé, gifts, rôle temporaire) |
| Réactions de mots (auto-réponses) | ❌ | 🟡 | **Manquant**. Pas de table `word_triggers`, pas de service. Simple à ajouter : table de triggers + listener messageCreate |

### 3.3 Jeux & événements

| Feature | Status | Effort | Notes |
|---|:---:|:---:|---|
| Giveaways | ✅ | — | Phase 5 — complet avec CSPRNG, cron, multi-choice, re-roll |
| Sondages (polls) | ✅ | — | Phase 5 — complet avec single/multi-choice, toggles, résultats live |
| Calendrier de l'Avent | ❌ | 🟡 | **Manquant**. Pas de feature dédiée. Simple : 24 cases, ouverture quotidienne, contenu custom. Pas critique |
| Route de l'Infini | ✅ | — | `game_road-to-infinite` (legacy) — complet, classements, multi-erreurs |
| Bingo | ❌ | 🟡 | **Manquant**. Pas de service bingo. Draftbot propose du bingo multijoueur avec tirage automatique. Effort moyen |
| Statistiques de jeux | 🟡 | 🟢 | `/api/stats/games` n'existe pas. On a les counts dans `counterState` / `countdownState` mais pas d'agrégats dashboard. À brancher rapidement |
| Commandes de jeux & fun | 🟡 | 🟢 | `/roll`, `/coinflip`, `/8ball` etc. absents. Trivia, pierre-feuille-ciseaux, etc. aussi. Effort faible par commande |

### 3.4 Communauté

| Feature | Status | Effort | Notes |
|---|:---:|:---:|---|
| Suggestions | ❌ | 🟡 | **Manquant**. Pas de table, pas de commande. Draftbot a `/suggestion créer/voter/admin`. Effort moyen |
| Tickets | ✅ | — | Phase 3 — complet (modal, claim, transcript HTML) |
| Starboards | ❌ | 🟡 | **Manquant**. Table `starboard_entries` + listener messageReactionAdd avec seuil. Effort moyen |
| Rôles-Réactions (reaction roles) | ❌ | 🟡 | **Manquant**. Très demandé. Message épinglé avec réactions ↔ rôles. Effort moyen |
| Interserveurs | ❌ | 🔴 | **Manquant**. Multi-guild broadcast, profil global, amis cross-serveur. Effort élevé (privacy, GDPR) |
| Salons vocaux temporaires | ❌ | 🟡 | **Manquant**. Listener voiceStateUpdate : crée un salon à l'arrivée de qqn dans "Join to Create", supprime quand vide. Effort moyen |
| Messages (auto-messages) | 🟡 | 🟢 | Le `DailyMessageModule` existe (legacy) et fait un message par jour avec IA. Draftbot a aussi des messages channel-bound (par salon). Pas de panel unifié |
| Messages récurrents | ❌ | 🟡 | **Manquant**. Cron-like schedule : envoyer X dans Y tous les Z. Table `recurring_messages` + cron. Effort moyen |
| Notifications sociales (social feed) | 🔶 | 🔴 | **Premium-only** sur Draftbot. Système de feed (Twitch, YouTube, Reddit) + watch parties. Pas pertinent pour un bot interne |

### 3.5 Sécurité

| Feature | Status | Effort | Notes |
|---|:---:|:---:|---|
| Modération (commandes) | ✅ | — | Phase 1 — complet : `/mod warn/mute/kick/ban/unban/history/clear`, sanctions progressives, audit log |
| Auto-Modération | ✅ | — | Phase 1 — 7 règles (spam, badwords, anti-raid, anti-invite, anti-link, mass-mention, anti-caps) |
| Gestion des messages (purge) | ✅ | — | `/mod clear` (Phase 1) + `purge` (legacy) |
| Signalements (reports) | ❌ | 🟡 | **Manquant**. Table `reports` + bouton "Signaler" sur les messages + queue staff. Effort moyen. À fort impact sur la modération communautaire |
| Rôles sécurisés (sticky roles) | ❌ | 🟢 | **Manquant** (mais simple). Sticky roles : un user qui quitte et revient retrouve ses rôles. Table `sticky_roles` + listener guildMemberAdd/Remove. Effort faible |
| Logs | ✅ | — | Phase 4 — 17 events Discord, WebSocket live, sanitization, stats dashboard. Le plus complet possible |

### 3.6 Utilitaires

| Feature | Status | Effort | Notes |
|---|:---:|:---:|---|
| Commandes d'informations | ❌ | 🟢 | **Manquant** : `/serverinfo`, `/userinfo`, `/avatar`, `/roleinfo`, `/channelinfo`. Effort faible, données déjà en BDD |
| Commandes personnalisées (custom commands) | ❌ | 🟡 | **Manquant** : `/customcmd add/list/remove` (texte/réponse/embed). Table `custom_commands`. Effort moyen |
| Rappels (reminders) | 🟡 | 🟡 | Pas de feature dédiée. Draftbot a `/remind` + cron pour les DM. Effort moyen mais à fort usage |
| Salons de statistiques | ❌ | 🟡 | **Manquant** : voice channel auto-update (membres en ligne, messages totaux). Listener + cron. Effort moyen |
| Sauvegardes du serveur | ❌ | 🔴 | **Manquant** : `/backup create/restore` (rôles, salons, perms). Effort élevé, sensible aux rate limits Discord |

## 4. Top 10 des features manquantes par valeur

Classement par **impact utilisateur / effort** (V/E = valeur / effort) :

| # | Feature | Valeur | Effort | V/E |
|---|---|:---:|:---:|:---:|
| 1 | Économie (monnaie) | 🔴 | 🔴 | 🟡 |
| 2 | Objets & inventaires | 🔴 | 🔴 | 🟡 |
| 3 | Rôles-Réactions | 🔴 | 🟡 | 🟢 |
| 4 | Commandes d'informations (`/serverinfo` etc.) | 🟡 | 🟢 | 🟢 |
| 5 | Sticky roles | 🟡 | 🟢 | 🟢 |
| 6 | Réactions de mots (auto-réponses) | 🟡 | 🟡 | 🟢 |
| 7 | Commandes personnalisées | 🟡 | 🟡 | 🟢 |
| 8 | Rappels (`/remind`) | 🟡 | 🟡 | 🟢 |
| 9 | Starboards | 🟡 | 🟡 | 🟡 |
| 10 | Signalements | 🟡 | 🟡 | 🟡 |

## 5. Roadmap recommandée (suggérée)

### Phase 8 — Quick wins UX (≈ 1 semaine)
- Commandes d'informations (`/serverinfo`, `/userinfo`, `/avatar`)
- Sticky roles
- Stats de jeux (brancher `counterState` + `countdownState` sur le dashboard)

### Phase 9 — Économie & Inventaire (≈ 3-4 semaines)
- Table `user_money`, `items`, `user_inventory`
- `EconomyService` + `InventoryService` + `ItemService`
- Slash : `/balance`, `/daily`, `/pay`, `/shop`, `/buy`, `/sell`
- `/inventaire`, `/objet donner/vendre/échanger`, `/dropobjet`, `/topitems`
- `/admininventaire *` (10 sub-commands)

### Phase 10 — Réaction Roles (≈ 1 semaine)
- Table `reaction_roles`
- Slash : `/reactionrole create/list/delete`
- Listener messageReactionAdd / messageReactionRemove
- Embed + select menu component

### Phase 11 — Engagement avancé (≈ 2 semaines)
- Réactions de mots (auto-réponses)
- Rappels (`/remind`)
- Commandes personnalisées (`/customcmd add/list/remove`)

### Phase 12 — Modération communautaire (≈ 2 semaines)
- Starboards
- Signalements (button "Signaler" + queue staff)
- Salons vocaux temporaires (Join-to-Create)
- Sauvegardes serveur

### Phase 13 — Jeux additionnels (≈ 1 semaine)
- Calendrier de l'Avent
- Bingo
- Commandes fun (`/roll`, `/coinflip`, `/8ball`, etc.)

### Phase 14 — Premium-like avancé
- Interserveurs (privacy, GDPR, opt-in)
- Notifications sociales (Twitch, YouTube) — décision stratégique

## 6. Conclusion

**Chienne couvre ~63% des features publiques Draftbot**, avec un focus marqué sur :
- ✅ **Sécurité** (5/6 features) — la plus complète
- ✅ **Engagement** (4/5) — il manque l'économie
- ✅ **Accueil des membres** (4/5) — l'écart est sur les variantes du captcha
- 🟡 **Jeux** (3/6 + 2 partiels) — le calendrier de l'avent et le bingo manquent
- 🟡 **Communauté** (1/8 + 1 partiel) — c'est le parent pauvre
- ❌ **Utilitaires** (0/5 + 1 partiel) — pas de feature dédiée

**Recommandation stratégique** : attaquer **Phase 9 (Économie + Inventaire)** car :
- C'est un **fondationnel** : 2autres features (cadeaux d'anniversaire, giveaways) en dépendent
- C'est un **engagement fort** : la monnaie virtuelle est le moteur de l'économie serveur
- Cela débloque immédiatement `/daily`, `/pay`, `/shop` côté utilisateur

## 7. Voir aussi

- [`draftbot-feature-list.md`](./draftbot-feature-list.md) — table exhaustive avec liens vers la doc Draftbot
- [`migration-impact.md`](./migration-impact.md) — impact des features à ajouter sur le schéma DB et l'API
- [`../features/`](../features/) — docs détaillées des features Chienne déjà livrées
