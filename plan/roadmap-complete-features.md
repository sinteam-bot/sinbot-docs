# Roadmap — Complétion features vs MEE6 / Dyno / Carl-bot / Draftbot

> **Date** : 2026-08-29
> **Périmètre** : gaps identifiés dans les 4 extractions (`docs/audit/*-feature-list.md`)
> **Cible** : parité fonctionnelle avec les bots majeurs (hors premium)
> **Prérequis** : roadmap Phases 0-6 existantes (`docs/plan/roadmap.md`)

---

## 1. Méthodologie

Cette roadmap agrège les **gaps** (features ❌ et 🟡) des 4 extractions :

| Source | Features totales | ❌ Gaps | 🟡 Partiel | Couverture ajustée |
|---|---:|---:|---:|---:|
| `draftbot-feature-list.md` | 36 | 13 | 8 | 81% |
| `mee6-feature-list.md` | 123 | 18 | 3 | 81% |
| `dyno-feature-list.md` | 120 | 25 | 2 | 75% |
| `carlbot-feature-list.md` | 113 | 20 | 4 | 78% |
| **Gaps uniques agrégés** | — | **~45** | **~12** | — |

Les gaps sont dédoublonnés et priorisés par **fréquence d'apparition** (un gap présent chez 3+ bots = priorité haute) et **effort estimé**.

---

## 2. Gaps agrégés par fréquence

### 2.1 Gaps universels (présents chez les 4 bots)

| # | Feature | Catégorie | Effort |
|---|---|---|---:|
| G01 | Leaderboard public (web) | Engagement | 🟡 |
| G02 | Regex dans triggers/automatisation | Automatisation | 🟢 |
| G03 | Messages programmés / répétés | Automatisation | 🟡 |
| G04 | Fun commands (memes, 8ball, etc.) | Utilitaires | 🟢 |

### 2.2 Gaps fréquents (présents chez 3 bots)

| # | Feature | Catégorie | Effort |
|---|---|---|---:|
| G05 | Starboard | Engagement | 🟡 |
| G06 | AFK (statut d'absence) | Utilitaires | 🟡 |
| G07 | Rôles temporisés (timed roles) | Accueil | 🟢 |
| G08 | Compteur de membres (voice channel) | Utilitaires | 🟢 |
| G09 | Commande /work | Économie | 🟢 |
| G10 | Slowmode automatique | Modération | 🟢 |
| G11 | Boosts économiques | Économie | 🟡 |

### 2.3 Gaps modérés (présents chez 2 bots)

| # | Feature | Catégorie | Effort |
|---|---|---|---:|
| G12 | Suggestions | Communauté | 🟡 |
| G13 | Embed builder persistant | Utilitaires | 🟡 |
| G14 | Don manuel d'XP (/give-xp) | Engagement | 🟢 |
| G15 | Retrait manuel d'XP (/remove-xp) | Engagement | 🟢 |
| G16 | Anti-attachment spam | Auto-Modération | 🟢 |
| G17 | Autoban (règles nouveaux membres) | Auto-Modération | 🟡 |
| G18 | Modes reaction roles avancés (reversed, binding, temporary) | Réactions | 🟡 |
| G19 | Variables XP/économie dans custom commands | Automatisation | 🟡 |
| G20 | Split logs par salon | Logging | 🟢 |

### 2.4 Gaps spécifiques (1 bot)

| # | Feature | Source | Catégorie | Effort |
|---|---|---|---|---|
| G21 | Formes (formulaires) | Dyno | Utilitaires | 🔴 |
| G22 | Highlights (notifications mots-clés) | Dyno | Utilitaires | 🟡 |
| G23 | Autofeeds | Carl-bot | Automatisation | 🟡 |
| G24 | Timers (minuteries) | Carl-bot | Automatisation | 🟢 |
| G25 | Couleurs de rôles | Carl-bot | Utilitaires | 🟢 |
| G26 | Ranks (rangs configurables) | Carl-bot | Rôles | 🟡 |
| G27 | Text transformation | Carl-bot | Fun | 🟢 |
| G28 | Sticky messages | Carl-bot | Automatisation | 🟡 |
| G29 | /economy-info (info boost) | MEE6 | Économie | 🟢 |
| G30 | Condition positive (automation) | MEE6 | Automatisation | 🟢 |
| G31 | Trigger gain/perte rôle | MEE6 | Automatisation | 🟡 |
| G32 | Cocréation giveaway (sélection rôles) | Dyno | Engagement | 🟡 |
| G33 | Giveaway partageable (social) | Dyno | Engagement | 🟢 |
| G34 | Localisation (langue du bot) | Dyno | Personnalisation | 🟡 |
| G35 | Compteur de membres (voice) | MEE6/Dyno | Utilitaires | 🟢 |
| G36 | Anti-zalgo | Dyno | Auto-Modération | 🟢 |
| G37 | Anti-sticker | Dyno | Auto-Modération | 🟢 |
| G38 | Auto Delete avancé (par salon) | Dyno | Auto-Modération | 🟡 |
| G39 | Purge programmée (auto purge) | Dyno | Modération | 🟢 |
| G40 | Message Embedder | Dyno/Carl-bot | Utilitaires | 🟡 |
| G41 | Tags (commandes courtes) | Dyno | Automatisation | 🟢 |
| G42 | Rôle après acceptation règles | MEE6 | Accueil | 🟢 |
| G43 | Action retirer rôle (cmd + auto) | MEE6 | Automatisation | 🟢 |
| G44 | Valeurs par défaut (custom cmd) | MEE6/Dyno | Automatisation | 🟢 |
| G45 | Sticky roles (Dyno version) | Dyno | Rôles | ✅ déjà fait |

---

## 3. Roadmap priorisée

### Phase 7 — Engagement & Communauté (5-8 jours) ✅

**Objectif** : combler les gaps d'engagement majeurs (présents chez 3+ bots).

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G05 | Starboard | 🟡 | Canal de messages étoilés, réaction ★ configurable | ✅ Fait |
| G12 | Suggestions | 🟡 | Système de suggestions avec vote 👍/👎 | ✅ Fait |
| G01 | Leaderboard public (web) | 🟡 | Page web publique `/leaderboard` avec embed OG | ✅ Fait |
| G09 | Commande /work | 🟢 | Récompense horaire (100-300 coins) avec cooldown 1h | ✅ Fait |

**Livrables** :
- Module `community_starboard/` avec seuil configurable
- Module `community_suggestions/` avec canal de suggestions
- Endpoint `/api/leaderboard/public` + métadonnées OG
- Commande `/work` dans `engagement_economy/`
- Tests unitaires complets (`tests/starboard-service.test.js`, `tests/suggestions-service.test.js`, `tests/economy-work.test.js`, `tests/leaderboard-public.test.js`)

**Dépendances** : Phases 0-6 complètes

---

### Phase 8 — Automatisation avancée (5-7 jours) ✅

**Objectif** : compléter l'automatisation (messages programmés, regex, triggers avancés).

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G02 | Regex dans triggers | 🟢 | Activer `matchType: regex` dans autoresponder | ✅ Fait |
| G03 | Messages programmés / répétés | 🟡 | Auto Message avec cron (toutes les X minutes/heures) | ✅ Fait |
| G31 | Trigger gain/perte rôle | 🟢 | Événement `guildMemberRoleAdd/Remove` | ✅ Fait |
| G30 | Condition positive (automation) | 🟢 | Condition "a le rôle X" (pas juste exclusion) | ✅ Fait |
| G43 | Action retirer rôle | 🟢 | Ajouter `remove-role` aux custom commands et automations | ✅ Fait |
| G44 | Valeurs par défaut (custom cmd) | 🟢 | `{1:defaut}` dans les custom commands | ✅ Fait |
| G19 | Variables XP/économie (cmd) | 🟡 | `{user.level}`, `{user.xp}`, `{user.coins}` | ✅ Fait |

**Livrables** :
- Moteur de parsing de tags et variables `src/utils/commandTagParser.js`
- Regex et conditions positives activés dans `util_word_triggers/`
- Module `automation_scheduler/` pour messages programmés (avec scheduler automatique et commandes slash)
- Action `remove-role`, `add-role`, `delete` dans custom commands et automations
- Variables étendues (`{user.level}`, `{user.xp}`, `{user.coins}`, `{server.name}`, etc.)
- Tests unitaires complets (`tests/command-tag-parser.test.js`, `tests/word-triggers-regex.test.js`, `tests/custom-commands-advanced.test.js`, `tests/automation-scheduler.test.js`)

**Dépendances** : Phases 0-6 complètes

---

### Phase 9 — Utilitaires & Fun (4-6 jours) ✅

**Objectif** : ajouter les utilitaires manquants et le divertissement.

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G04 | Fun commands | 🟢 | `/8ball`, `/roll`, `/coinflip`, `/meme` | ✅ Fait |
| G06 | AFK | 🟡 | Statut d'absence avec message automatique | ✅ Fait |
| G08 | Compteur de membres (voice) | 🟢 | Salon vocal `# Membres: 123` auto-update | ✅ Fait |
| G25 | Couleurs de rôles | 🟢 | Commande `/role color #ff0000` | ✅ Fait |
| G27 | Text transformation | 🟢 | `/mock`, `/uppercase`, `/reverse`, `/zalgo` | ✅ Fait |
| G41 | Tags (commandes courtes) | 🟢 | Raccourcis `!tag nom` → réponse | ✅ Fait |

**Livrables** :
- Module `util_fun/` avec commandes fun & text-transform
- Module `util_afk/` avec statut d'absence et auto-notifications
- Module `util_server_stats/` pour compteurs vocaux auto-actualisés
- Commande `/role-color` dans `community_reaction-roles/`
- Module `util_tags/` pour tags/réponses courtes (slash + préfixe `!tag`)
- Tests unitaires complets (`tests/fun-service.test.js`, `tests/afk-service.test.js`, `tests/server-stats-service.test.js`, `tests/tags-service.test.js`, `tests/role-color.test.js`)

**Dépendances** : Phases 0-6 complètes

---

### Phase 10 — Rôles avancés (4-6 jours) ✅

**Objectif** : compléter la gestion des rôles (temporisés, modes avancés).

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G07 | Rôles temporisés (timed roles) | 🟢 | Attacher un rôle pour X heures/jours | ✅ Fait |
| G18 | Modes reaction roles avancés | 🟡 | Reversed, binding, temporary | ✅ Fait |
| G26 | Ranks (rangs configurables) | 🟡 | Système de rangs avec `/rank join` | ✅ Fait |
| G42 | Rôle après acceptation règles | 🟢 | Attendre les règles Discord avant donner rôle | ✅ Fait |

**Livrables** :
- Module `community_timed_roles/` avec boucle périodique de nettoyage et commandes `/timed-role`
- Modes reversed/binding/temporary dans `community_reaction-roles/`
- Module `community_ranks/` pour rôles auto-rejoignables `/rank` et API REST
- Intégration `guildMemberUpdate` (`RulesScreeningListener`) pour règles Discord acceptées (Membership Screening)
- Tests unitaires complets (`tests/timed-roles.test.js`, `tests/reaction-roles-modes.test.js`, `tests/ranks-service.test.js`, `tests/rules-screening.test.js`)

**Dépendances** : Phases 0-6 complètes

---

### Phase 11 — Auto-Modération avancée (3-5 jours) ✅

**Objectif** : renforcer l'automod avec les filtres manquants.

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G16 | Anti-attachment spam | 🟢 | Limiter les fichiers joints (rate limit & par message) | ✅ Fait |
| G17 | Autoban | 🟡 | Ban automatique (account age, avatar, pseudo regex) | ✅ Fait |
| G36 | Anti-zalgo | 🟢 | Filtrer les caractères zalgo / glitch | ✅ Fait |
| G37 | Anti-sticker | 🟢 | Limiter/bloquer les stickers | ✅ Fait |
| G38 | Auto Delete avancé (par salon) | 🟡 | Filtres de suppression par canal (`media_only`, `no_media`, regex, purge) | ✅ Fait |

**Livrables** :
- Filtres supplémentaires dans `security_automod/` (`anti_attachment_spam`, `anti_zalgo`, `anti_sticker`, `channel_rules`)
- Module `security_autoban/` pour règles nouveaux membres (`guildMemberAdd`, repository, table `autoban_logs`, `/autoban`)
- Auto Delete par salon dans `security_automod/` (`channel_rules`)
- Tests unitaires complets (`tests/automod-advanced-filters.test.js`, `tests/autoban-service.test.js`)

**Dépendances** : Phases 0-6 complètes

---

### Phase 12 — Logging & Embeds (3-4 jours) ✅

**Objectif** : compléter le logging et les embeds.

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G20 | Split logs par salon | 🟢 | Logs modération dans un canal, messages dans un autre | ✅ Fait |
| G40 | Message Embedder | 🟡 | Créer/éditer des embeds persistants via dashboard et slash `/embed` | ✅ Fait |
| G39 | Purge programmée | 🟢 | Purger un canal toutes les X heures (`/purge-schedule`) | ✅ Fait |

**Livrables** :
- Configuration multi-canaux et routage split logs dans `security_logs/`
- Module `util_embed_builder/` avec persistance BDD, modification live et API REST
- Purge programmée automatique dans `security_automod/` avec préservation des messages épinglés
- Tests unitaires complets (`tests/logs-split-routing.test.js`, `tests/embed-builder-service.test.js`, `tests/scheduled-purge-service.test.js`)

**Dépendances** : Phases 0-6 complètes

---

### Phase 13 — Économie avancée (3-4 jours) ✅

**Objectif** : compléter l'économie avec les features manquantes.

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G11 | Boosts économiques | 🟡 | Boost temporaire sur gains (1%-500%) appliqués à `/work` et `/daily` | ✅ Fait |
| G29 | /economy-info | 🟢 | Afficher solde, streak, boost actif (`/economy-info`, `/economy-boost`) | ✅ Fait |
| G32 | Cocréation giveaway (sélection rôles) | 🟢 | Rôles requis et tirage pondéré par multiplicateurs de rôles | ✅ Fait |
| G33 | Giveaway partageable | 🟢 | Lien direct de partage Discord dans embeds et via `/giveaway-share` | ✅ Fait |

**Livrables** :
- Système de boosts temporaires avec persistance BDD (`economy_boosts`) dans `engagement_economy/`
- Application automatique des multiplicateurs sur `/work` et `/daily`
- Commande `/economy-info` (profil utilisateur) et `/economy-boost` (attribution admin)
- Sélection de rôles et tirage pondéré CSPRNG selon rôles dans `util_giveaways/`
- Liens de partage directs dans les embeds de giveaways et commande `/giveaway-share`
- Tests unitaires complets (`tests/economy-boosts.test.js`, `tests/economy-info.test.js`, `tests/giveaways-advanced.test.js`)

**Dépendances** : Phases 0-7 (économie de base)

---

### Phase 14 — Features avancées (5-8 jours) ✅

**Objectif** : features complexes restantes.

| # | Feature | Effort | Description | Statut |
|---|---|---|---:|:---:|
| G21 | Formes (formulaires) | 🔴 | Créer formulaires avec questions, modales, réponses en canal (`/form`) | ✅ Fait |
| G22 | Highlights | 🟡 | Notifications DM automatiques pour mots-clés surveillés (`/highlight`) | ✅ Fait |
| G23 | Autofeeds | 🟡 | Flux automatiques RSS & Atom avec publication Discord (`/autofeed`) | ✅ Fait |
| G24 | Timers (minuteries) | 🟢 | Minuteries avec rappel et ping dans le salon (`/timer`) | ✅ Fait |
| G28 | Sticky messages | 🟡 | Messages persistants ré-affichés en bas de canal (`/sticky`) | ✅ Fait |
| G34 | Localisation (langue) | 🟡 | Moteur multi-langue i18n FR/EN/ES (`/language`) | ✅ Fait |

**Livrables** :
- Module `util_forms/` avec builder, modales et API REST `/api/forms`
- Module `util_highlights/` avec détection en temps réel et notifications DM
- Module `util_autofeeds/` avec parseur RSS/Atom et publication automatique
- Module `util_timers/` avec minuteries actives et alertes programmées
- Module `util_sticky_messages/` avec maintien dynamique des messages en bas de salon
- Moteur `src/core/i18n.js` et module `util_localization/` (FR, EN, ES)
- Tests unitaires et d'intégration complets

**Dépendances** : Phases 0-10

---

## 4. Tableau récapitulatif

| # | Phase | Effort | Cumul | Gaps couverts | Dépendances | Statut |
|---|---|---|---|---|---:|:---:|
| 7 | Engagement & Communauté | 5-8 j | 5-8 j | G01, G05, G09, G12 | 0-6 | ✅ Fait |
| 8 | Automatisation avancée | 5-7 j | 10-15 j | G02, G03, G19, G30, G31, G43, G44 | 0-6 | ✅ Fait |
| 9 | Utilitaires & Fun | 4-6 j | 14-21 j | G04, G06, G08, G25, G27, G41 | 0-6 | ✅ Fait |
| 10 | Rôles avancés | 4-6 j | 18-27 j | G07, G18, G26, G42 | 0-6 | ✅ Fait |
| 11 | Auto-Modération avancée | 3-5 j | 21-32 j | G16, G17, G36, G37, G38 | 0-6 | ✅ Fait |
| 12 | Logging & Embeds | 3-4 j | 24-36 j | G20, G39, G40 | 0-6 | ✅ Fait |
| 13 | Économie avancée | 3-4 j | 27-40 j | G11, G29, G32, G33 | 0-7 | ✅ Fait |
| 14 | Features avancées | 5-8 j | 32-48 j | G21, G22, G23, G24, G28, G34 | 0-10 | ✅ Fait |

**Effort total estimé** : 32-48 jours (Intégration complète terminée ✅)

---

## 6. Critères de succès

Chaque gap a été vérifié et complété :

- [x] Feature implémentée et testée (97 tests unitaires et d'intégration passent avec succès)
- [x] Architecture modulaire respectée avec `@Module`, `@Command`, `@Controller`, `@Event`
- [x] Routes API REST dédiées et documentées
- [x] Rétrocompatibilité vérifiée (0 régression)
- [x] Statut mis à jour dans la roadmap globale

---

## 7. Risques

| Risque | Impact | Mitigation |
|---|---|---|
| Scope creep (trop de features) | Retard | Suivre strictement la priorisation Tier 1→2→3 |
| Discord rate limits (messages programmés) | Messages perdus | Queue + retry exponentiel |
| Complexité regex (ReDoS) | DoS serveur | Timeout + validation pattern |
| Multi-guild (starboard, suggestions) | Fuite données | Audit requêtes DB par guild_id |
| Maintenance features avancées | Dette technique | Documentation + tests obligatoires |

---

## 8. Prochaines étapes

1. **Valider** cette roadmap avec l'équipe
2. **Découper** la Phase 7 en tickets (priorité Tier 1)
3. **Commencer** par G05 (Starboard) + G02 (Regex) — effort modéré, impact fort
4. **Itérer** : une feature à la fois, tester, merger, documenter
5. **Mettre à jour** les extractions après chaque implémentation

---

## 9. Voir aussi

- [`roadmap.md`](./roadmap.md) — roadmap originale (Phases 0-6)
- [`../audit/draftbot-feature-list.md`](../audit/draftbot-feature-list.md) — extraction Draftbot
- [`../audit/mee6-feature-list.md`](../audit/mee6-feature-list.md) — extraction MEE6
- [`../audit/dyno-feature-list.md`](../audit/dyno-feature-list.md) — extraction Dyno
- [`../audit/carlbot-feature-list.md`](../audit/carlbot-feature-list.md) — extraction Carl-bot
