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

### Phase 8 — Automatisation avancée (5-7 jours)

**Objectif** : compléter l'automatisation (messages programmés, regex, triggers avancés).

| # | Feature | Effort | Description |
|---|---|---|---:|
| G02 | Regex dans triggers | 🟢 | Activer `matchType: regex` dans autoresponder |
| G03 | Messages programmés / répétés | 🟡 | Auto Message avec cron (toutes les X minutes/heures) |
| G31 | Trigger gain/perte rôle | 🟢 | Événement `guildMemberRoleAdd/Remove` |
| G30 | Condition positive (automation) | 🟢 | Condition "a le rôle X" (pas juste exclusion) |
| G43 | Action retirer rôle | 🟢 | Ajouter `remove-role` aux custom commands et automations |
| G44 | Valeurs par défaut (custom cmd) | 🟢 | `{1:defaut}` dans les custom commands |
| G19 | Variables XP/économie (cmd) | 🟡 | `{user.level}`, `{user.xp}`, `{user.coins}` |

**Livrables** :
- Regex activé dans `game_engagement-advanced/`
- Module `automation_scheduler/` pour messages programmés
- Conditions positives dans le système de triggers
- Action `remove-role` dans custom commands
- Variables étendues dans le parser de custom commands
- Tests unitaires

**Dépendances** : Phases 0-6 complètes

---

### Phase 9 — Utilitaires & Fun (4-6 jours)

**Objectif** : ajouter les utilitaires manquants et le divertissement.

| # | Feature | Effort | Description |
|---|---|---|---:|
| G04 | Fun commands | 🟢 | `/8ball`, `/roll`, `/coinflip`, `/meme` |
| G06 | AFK | 🟡 | Statut d'absence avec message automatique |
| G08 | Compteur de membres (voice) | 🟢 | Salon vocal `# Membres: 123` auto-update |
| G25 | Couleurs de rôles | 🟢 | Commande `/role color #ff0000` |
| G27 | Text transformation | 🟢 | `/mock`, `/uppercase`, `/reverse`, etc. |
| G41 | Tags (commandes courtes) | 🟢 | Raccourcis `!tag nom` → réponse |

**Livrables** :
- Module `util_fun/` avec commandes fun
- Module `util_afk/` avec statut d'absence
- Module `util_server-stats/` pour compteurs vocaux
- Commande `/role color` dans `community_reaction-roles/`
- Module `util_tags/` pour tags/courtes
- Tests unitaires

**Dépendances** : Phases 0-6 complètes

---

### Phase 10 — Rôles avancés (4-6 jours)

**Objectif** : compléter la gestion des rôles (temporisés, modes avancés).

| # | Feature | Effort | Description |
|---|---|---|---:|
| G07 | Rôles temporisés (timed roles) | 🟢 | Attacher un rôle pour X heures/jours |
| G18 | Modes reaction roles avancés | 🟡 | Reversed, binding, temporary |
| G26 | Ranks (rangs configurables) | 🟡 | Système de rangs avec `/rank join` |
| G42 | Rôle après acceptation règles | 🟢 | Attendre les règles Discord avant donner rôle |

**Livrables** :
- Module `community_timed-roles/` avec cron
- Modes reversed/binding/temporary dans `community_reaction-roles/`
- Module `community_ranks/` pour rôles rejoignables
- Intégration `guildMemberUpdate` pour règles acceptées
- Tests unitaires

**Dépendances** : Phases 0-6 complètes

---

### Phase 11 — Auto-Modération avancée (3-5 jours)

**Objectif** : renforcer l'automod avec les filtres manquants.

| # | Feature | Effort | Description |
|---|---|---|---:|
| G16 | Anti-attachment spam | 🟢 | Limiter les fichiers joints (rate limit) |
| G17 | Autoban | 🟡 | Ban automatique (account age, username, etc.) |
| G36 | Anti-zalgo | 🟢 | Filtrer les caractères zalgo |
| G37 | Anti-sticker | 🟢 | Limiter/bloquer les stickers |
| G38 | Auto Delete avancé (par salon) | 🟡 | Filtres de suppression par canal |

**Livrables** :
- Filtres supplémentaires dans `security_automod/`
- Module `security_autoban/` pour règles nouveaux membres
- Auto Delete par salon dans `security_automod/`
- Tests unitaires

**Dépendances** : Phases 0-6 complètes

---

### Phase 12 — Logging & Embeds (3-4 jours)

**Objectif** : compléter le logging et les embeds.

| # | Feature | Effort | Description |
|---|---|---|---:|
| G20 | Split logs par salon | 🟢 | Logs modération dans un canal, messages dans un autre |
| G40 | Message Embedder | 🟡 | Créer/éditer des embeds persistants via dashboard |
| G39 | Purge programmée | 🟢 | Purger un canal toutes les X heures |

**Livrables** :
- Configuration multi-canaux dans `security_logs/`
- Module `util_embed-builder/` avec persistance
- Auto Purge dans `security_automod/`
- Tests unitaires

**Dépendances** : Phases 0-6 complètes

---

### Phase 13 — Économie avancée (3-4 jours)

**Objectif** : compléter l'économie avec les features manquantes.

| # | Feature | Effort | Description |
|---|---|---|---:|
| G11 | Boosts économiques | 🟡 | Boost temporaire sur gains (1%-500%) |
| G29 | /economy-info | 🟢 | Afficher solde, streak, boost actif |
| G32 | Cocréation giveaway (sélection rôles) | 🟢 | Sélectionner rôles gagnants |
| G33 | Giveaway partageable | 🟢 | Lien partageable pour giveaway |

**Livrables** :
- Système de boost dans `engagement_economy/`
- Commande `/economy-info`
- Sélection de rôles dans `game_engagement/`
- Lien partageable pour giveaways
- Tests unitaires

**Dépendances** : Phases 0-7 (économie de base)

---

### Phase 14 — Features avancées (5-8 jours)

**Objectif** : features complexes restantes.

| # | Feature | Effort | Description |
|---|---|---|---:|
| G21 | Formes (formulaires) | 🔴 | Créer formulaires avec questions, réponses en canal |
| G22 | Highlights | 🟡 | Notifications DM pour mots-clés |
| G23 | Autofeeds | 🟡 | Flux automatiques (RSS, etc.) |
| G24 | Timers (minuteries) | 🟢 | Minuteries avec notification |
| G28 | Sticky messages | 🟡 | Messages persistants en bas de canal |
| G34 | Localisation (langue) | 🟡 | Multi-langue (FR/EN/ES) |

**Livrables** :
- Module `util_forms/` avec builder
- Module `util_highlights/` pour notifications mots-clés
- Module `util_autofeeds/` pour flux
- Module `util_timers/` pour minuteries
- Module `util_sticky-messages/`
- Système i18n avec fichiers de traduction
- Tests unitaires

**Dépendances** : Phases 0-10

---

## 4. Tableau récapitulatif

| # | Phase | Effort | Cumul | Gaps couverts | Dépendances |
|---|---|---|---|---|---:|
| 7 | Engagement & Communauté | 5-8 j | 5-8 j | G01, G05, G09, G12 | 0-6 |
| 8 | Automatisation avancée | 5-7 j | 10-15 j | G02, G03, G19, G30, G31, G43, G44 | 0-6 |
| 9 | Utilitaires & Fun | 4-6 j | 14-21 j | G04, G06, G08, G25, G27, G41 | 0-6 |
| 10 | Rôles avancés | 4-6 j | 18-27 j | G07, G18, G26, G42 | 0-6 |
| 11 | Auto-Modération avancée | 3-5 j | 21-32 j | G16, G17, G36, G37, G38 | 0-6 |
| 12 | Logging & Embeds | 3-4 j | 24-36 j | G20, G39, G40 | 0-6 |
| 13 | Économie avancée | 3-4 j | 27-40 j | G11, G29, G32, G33 | 0-7 |
| 14 | Features avancées | 5-8 j | 32-48 j | G21, G22, G23, G24, G28, G34 | 0-10 |

**Effort total estimé** : 32-48 jours (parallélisable partiellement)

---

## 5. Priorisation alternative (par impact)

Si le temps est limité, voici l'ordre d'impact décroissant :

### Tier 1 — Impact maximum (à faire absolument)

| Feature | Raison | Effort |
|---|---|:---:|
| Starboard | Présent chez 3 bots, feature communautaire majeure | 🟡 |
| Regex triggers | Manquant dans toutes les automatisations | 🟢 |
| Messages programmés | Présent chez 3 bots, automation basique | 🟡 |
| AFK | Présent chez 2 bots, feature sociale populaire | 🟡 |
| Suggestions | Présent chez 2 bots, feature communautaire | 🟡 |
| /work | Complète l'économie (daily existe) | 🟢 |

**Effort Tier 1** : ~12-18 jours

### Tier 2 — Impact élevé (fortement recommandé)

| Feature | Raison | Effort |
|---|---|:---:|
| Leaderboard public | Visibilité serveur, présent chez tous | 🟡 |
| Rôles temporisés | Présent chez 3 bots | 🟢 |
| Modes reaction roles avancés | Parité Carl-bot | 🟡 |
| Compteur vocal | Utilitaire simple, présent chez 3 bots | 🟢 |
| Anti-attachment spam | Sécurité, présent chez Carl-bot | 🟢 |
| Autoban | Sécurité, présent chez Dyno | 🟡 |

**Effort Tier 2** : ~10-15 jours

### Tier 3 — Impact modéré (quand le temps le permet)

| Feature | Raison | Effort |
|---|---|:---:|
| Fun commands | Divertissement | 🟢 |
| Embed builder | Création avancée | 🟡 |
| Formulaires | Engagement | 🔴 |
| Localisation | Accessibilité | 🟡 |
| Highlights | Engagement | 🟡 |

**Effort Tier 3** : ~12-20 jours

---

## 6. Critères de succès

Chaque gap doit être considéré comme complété quand :

- [ ] Feature implémentée et testée (tests unitaires passent)
- [ ] Documentation mise à jour (`docs/features/`)
- [ ] Dashboard configurable (si applicable)
- [ ] Rétrocompatibilité vérifiée (pas de régression)
- [ ] Status mis à jour dans les 4 extractions (`docs/audit/*`)

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
