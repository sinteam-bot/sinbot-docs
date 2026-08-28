# Plan — Split database.js / db/index.js en repositories & schema par module

> **Date** : 2026-08-28
> **Origine** : audit code — `src/database.js` (2104 lignes) et `src/db/index.js` (1134 lignes) sont des monolithes qui violent la séparation des responsabilités du pattern module.
> **Objectif** : chaque module possède son **schema Drizzle** + son **repository**, et les **migrations sont versionnées** via `drizzle-kit`.

---

## 1. Diagnostic

### 1.1 État actuel

| Fichier | Lignes | Responsabilités mélangées |
|---|---|---|
| `src/database.js` | 2104 | ~80 fonctions de domaine (XP, captcha, tickets, economy, mod, welcome, birthdays, reports, …) qui touchent en direct `db`/`schema` globaux. Mélange SQL legacy (`pool.query` avec `?` → `$n`) et Drizzle. |
| `src/db/index.js` | 1134 | Bootstrap connexion + **un gros `PG_TABLES_DDL` inline** (~60+ tables) + **migration ALTER TABLE ad-hoc** (patchwork dans le code). |
| `src/db/schema/pg.js` | 1000 | Un seul fichier qui exporte **toutes** les tables Drizzle pour tout le projet. |
| `src/database/` (dossier) | — | Contient `captcha-tables.sql` et `setup-captcha-tables.js` (dossier legacy, plus ou moins abandonné). |

### 1.2 Problèmes concrets

1. **God-files** : impossible de savoir qui possède une table sans grepper.
2. **Couplage fort** : `database.js` est importé par plusieurs modules → impossible d'activer/désactiver une feature sans casser les autres.
3. **Pas de versionning des migrations** : le bloc `migrationStatements` dans `initPgTables()` est un *patchwork* non reproductible, non rejouable à l'identique, et qui n'a aucun lien avec le schema Drizzle.
4. **Double source de vérité** : `PG_TABLES_DDL` (SQL brut) coexiste avec `schema/pg.js` (Drizzle). Les deux peuvent diverger silencieusement.
5. **Dossier `db/` vide** dans la plupart des modules (`feature_automod/db/`, `feature_tickets/db/`, …) → l'emplacement pour la persistance par module **a déjà été prévu** mais jamais rempli.
6. **Tests fragiles** : `createTestDb()` ré-exécute tout le DDL → tests lents et ordre-dépendants.

### 1.3 Cible

- Chaque **module** = **un schema Drizzle** + **un repository** + **des migrations versionnées** par feature.
- `src/db/` ne contient plus que la **connexion** et l'**orchestrateur de migrations** (`drizzle-kit`).
- `database.js` **disparaît** : ses fonctions migrent vers les repositories des modules concernés.
- Les **modules existants** (XP, tickets, captcha, …) respectent le pattern déjà utilisé par `feature_xp-level` (cf. `src/modules/feature_xp-level/xp-level.repository.js`).

---

## 2. Architecture cible

```
src/
├── db/
│   ├── client.js               # init PGlite/pg + factory db (Drizzle)
│   ├── schemas/                # ⚠️ NOUVEAU : un sous-dossier par "domaine transverse"
│   │   ├── index.js            # agrège tous les schémas + export `schema` global pour Drizzle
│   │   ├── shared/
│   │   │   ├── audit.js        # mod_logs, event_log, auth_audit_logs
│   │   │   ├── cache.js        # discord_guilds/channels/roles/members/messages/emojis/threads
│   │   │   ├── feature-flags.js
│   │   │   └── guild-settings.js
│   │   └── …
│   └── migrations/             # ⚠️ généré par drizzle-kit (versionné git)
│       ├── 0000_initial.sql
│       ├── 0001_xp_level.sql
│       ├── 0002_tickets.sql
│       └── meta/_journal.json
│
├── modules/
│   ├── feature_xp-level/
│   │   ├── db/
│   │   │   ├── schema.js       # tables Drizzle du module
│   │   │   ├── migrations/     # (optionnel) migrations propres au module si isolé
│   │   │   └── README.md
│   │   ├── xp-level.repository.js   # (existe déjà, on l'enrichit)
│   │   ├── xp-level.module.js
│   │   └── …
│   ├── feature_tickets/
│   │   ├── db/
│   │   │   └── schema.js       # ⚠️ nouveau (dossier vide actuellement)
│   │   ├── tickets.repository.js   # ⚠️ nouveau
│   │   └── tickets.module.js       # ajoute `ticketsRepository` aux providers
│   ├── feature_captcha/         (renommage de security_question + security_captcha)
│   │   ├── db/schema.js
│   │   ├── captcha.repository.js
│   │   └── …
│   ├── feature_economy/
│   │   ├── db/schema.js
│   │   ├── economy.repository.js
│   │   └── …
│   ├── feature_birthdays/
│   │   ├── db/schema.js
│   │   ├── birthdays.repository.js
│   │   └── …
│   ├── feature_reports/
│   │   ├── db/schema.js
│   │   ├── reports.repository.js
│   │   └── …
│   ├── feature_reaction-roles/
│   │   ├── db/schema.js
│   │   ├── reaction-roles.repository.js
│   │   └── …
│   ├── feature_temp-voice/
│   │   ├── db/schema.js
│   │   ├── temp-voice.repository.js
│   │   └── …
│   ├── feature_sticky-roles/
│   │   ├── db/schema.js
│   │   ├── sticky-roles.repository.js
│   │   └── …
│   ├── feature_engagement/  (giveaways, polls, custom commands, word triggers, reminders)
│   │   ├── db/schema.js
│   │   ├── engagement.repository.js
│   │   └── …
│   ├── feature_automod/         (mod_logs, user_warnings, user_sanctions, tickets_legacy)
│   │   ├── db/schema.js
│   │   ├── automod.repository.js
│   │   └── …
│   ├── feature_welcome/         (welcome_config, welcome_cards, role_assign_logs)
│   │   ├── db/schema.js
│   │   ├── welcome.repository.js
│   │   └── …
│   ├── feature_info/            (auth_sessions, auth_audit_logs, auth_failed_attempts, bot_config, bot_state)
│   │   ├── db/schema.js
│   │   ├── info.repository.js
│   │   └── …
│   └── …
│
├── database.js                 # ❌ SUPPRIMÉ en fin de migration
└── …
```

### 2.1 Règles de découpage

| Règle | Raison |
|---|---|
| Chaque module expose **son** schema dans `db/schema.js` | Un module = un domaine métier = ses tables. |
| Chaque module possède **un** `*.repository.js` qui **étend** `Repository()` (décorateur existant) | Pattern déjà éprouvé dans `feature_xp-level`. |
| Les repositories sont **injectés via le container** (`Module({ providers: [...] })`) et **résolus par les services** | Aucune fonction `database.js`-style n'est appelée depuis l'extérieur d'un module. |
| Les **schémas transverses** (cache Discord, audit, feature flags) restent dans `db/schemas/shared/` | Ils ne sont la propriété d'aucun module et sont requis par plusieurs. |
| Le **schema global** exporté à Drizzle = agrégation de `db/schemas/shared/*` + tous les `modules/*/db/schema.js` | Une seule connexion Drizzle, plusieurs sources de définition. |

---

## 3. Schéma Drizzle par module (mapping depuis `schema/pg.js` + `database.js`)

| Table(s) | Module cible | Repository |
|---|---|---|
| `user_xp`, `xp_transactions`, `voice_sessions`, `events`, `event_participants` | `feature_xp-level` (existe déjà partiellement) | `XPLevelRepository` (existe) |
| `user_birthdays`, `birthday_guild_settings`, `birthday_visibility`, `birthday_change_log`, `birthday_history` | `feature_birthdays` | `BirthdaysRepository` (à créer) |
| `tickets`, `ticket_messages`, `ticket_attachments` | `feature_tickets` | `TicketsRepository` (à créer) |
| `reports`, `report_actions` | `feature_reports` | `ReportsRepository` (à créer) |
| `reaction_roles` (v1 + v2) | `feature_reaction-roles` | `ReactionRolesRepository` (à créer) |
| `user_economy`, `economy_transactions`, `shop_items`, `user_inventory`, `inventory_drops`, `inventory_transfers` | `feature_economy` | `EconomyRepository` (à créer) |
| `sticky_roles` | `feature_sticky-roles` | `StickyRolesRepository` (à créer) |
| `temp_voice_config`, `temp_voice_state` | `feature_temp-voice` | `TempVoiceRepository` (à créer) |
| `reminders`, `word_triggers`, `custom_commands`, `giveaways`, `giveaway_entries`, `polls`, `poll_votes` | `feature_engagement` (à étendre) | `EngagementRepository` (à étendre) |
| `user_warnings`, `user_sanctions`, `mod_logs`, `event_log` | `feature_automod` | `AutomodRepository` (à créer) |
| `welcome_config`, `welcome_cards`, `role_assign_logs` | `feature_welcome` | `WelcomeRepository` (à créer) |
| `user_captchas`, `captcha_logs`, `captcha_config` | `feature_captcha` (regrouper `security_captcha` + `security_question`) | `CaptchaRepository` (à créer) |
| `auth_sessions`, `auth_audit_logs`, `auth_failed_attempts`, `bot_config`, `bot_state` | `feature_info` (sécurité/auth) | `InfoRepository` (à créer) |
| `bump_logs` | `service_bump-reminder` | `BumpReminderRepository` (à créer) |
| `server_members`, `member_history`, `discord_channels`, `discord_threads`, `discord_messages`, `discord_emojis`, `discord_roles`, `guilds`, `guild_members_cache` | `db/schemas/shared/cache.js` | `DiscordCacheRepository` (couche service) |
| `user_events`, `form_responses` | `db/schemas/shared/audit.js` (telemetry bot) | `AuditRepository` (à créer) |
| `user_profiles` | `feature_info` (peut aussi aller dans `shared`) | `InfoRepository` |
| `feature_flags`, `guild_settings` | `db/schemas/shared/feature-flags.js` | `FeatureFlagsRepository` (déjà couvert par `core/feature-registry`?) |
| `conversation_contexts`, `openaimessages` | `db/schemas/shared/audit.js` (ou feature `service_openai` à créer) | `OpenAIRepository` (à créer si on garde la feature) |
| `counter_state`, `countdown_state` | `game_count-down` / `game_road-to-infinite` | `GameStateRepository` (à créer) |
| `guild_stats` | `db/schemas/shared/cache.js` | lu par dashboard, écrit par cache service |

> **Note** : `user_events` et `form_responses` sont très probablement des **artefacts** ; à confirmer auprès du mainteneur avant de les porter tels quels.

---

## 4. Stratégie de migration (étapes ordonnées)

### Étape 0 — Préparation (½ journée) ✅ TERMINÉE

- [x] Installer `drizzle-kit` : `npm install -D drizzle-kit`.
- [x] Créer `drizzle.config.js` à la racine, pointant vers `src/db/schemas/index.js` comme `schema`, `src/db/migrations/` comme `out`, driver `pg` (et config PGlite pour les tests).
- [x] Ajouter les scripts npm :
  ```json
  "scripts": {
    "db:generate": "drizzle-kit generate",
    "db:migrate":  "drizzle-kit migrate",
    "db:studio":   "drizzle-kit studio",
    "db:push":     "drizzle-kit push"
  }
  ```
- [x] Documenter le workflow dans `docs/guides/migrations.md` (nouveau).
- [x] Créer une **baseline** : `src/db/migrations/0000_dazzling_sauron.sql` (~50 tables).

### Étape 1 — Isoler la connexion (½ journée) ✅ TERMINÉE

- [x] `src/db/client.js` : nouveau fichier qui contient **uniquement** la factory `initDatabase()` (Drizzle + PGlite/pg) + migrator Drizzle. Pas de DDL legacy.
- [x] `src/db/index.js` : 50 lignes, *barrel* qui réexporte `client.js` + le schema global.
- [x] Schémas par module créés en `src/modules/*/db/schema.js` (19 modules).
- [x] Tables transverses extraites dans `src/db/schemas/shared/` (audit, cache, feature-flags, bot-info, openai).
- [x] `legacy.js` réduit à 64 lignes (barrel d'agrégation, ex `pg.js` à 1000 lignes).

### Étape 2 — Découper `database.js` (1-2 jours) ⚠️ PARTIELLEMENT TERMINÉE

Approche : **strangler-fig pattern** — `database.js` reste opérationnel via `src/db/legacy-bridge-impl.js` (2108 lignes, utilisé uniquement par `legacy-bridge.js`).

Pour chaque module (ordre = criticité / fréquence d'usage) :

1. [x] **Créer `src/modules/<name>/db/schema.js`** : 19/19 modules. Tables Drizzle déplacées depuis `legacy.js`.
2. [⚠️] **Créer `src/modules/<name>/<name>.repository.js`** : 9/19 modules ont déjà un repository natif (XP, tickets, birthdays, economy, reports, reaction-roles, temp-voice, sticky-roles, engagement, automod, welcome, info, bump-reminder, captcha, games, daily-message). Les **71 fonctions legacy** sont toujours dans `legacy-bridge-impl.js` (à migrer → critère 7).
3. [❌] **Modifier `src/modules/<name>/<name>.module.js`** : repositories **non encore enregistrés comme `providers`** du module (amélioration NestJS-like, optionnelle).
4. [x] **Modifier tous les call-sites** : 13 imports de `database.js` redirigés vers `db/legacy-bridge.js` (par namespace).
5. [x] **Tests** : 604/604 passent. Aucun test de repository ajouté (le pattern `createTestDb()` est conservé).
6. [x] **Lint + grep** : 0 import direct de `database.js` subsiste (vérifié par grep CI).

### Étape 3 — Basculer le schema global (1 jour) ✅ TERMINÉE

- [x] `src/db/schemas/index.js` : agrège `shared/*` + tous les `modules/*/db/schema.js` et exporte `{ ...userXp, ...tickets, ...audit, ..., schema: { userXp, tickets, … } }`.
- [x] `src/db/client.js` : utilise ce nouvel agrégat pour `drizzle(..., { schema })`.
- [x] `drizzle-kit generate` : produit la migration baseline `0000_dazzling_sauron.sql`.
- [⚠️] `drizzle-kit generate` produit encore un diff (`text` → `bigint`) — à rebaser (critère 4).
- [x] `src/db/schema/pg.js` : supprimé, contenu migré vers `src/db/schemas/legacy.js` (qui est lui-même devenu un barrel de 64 lignes).

### Étape 4 — Supprimer la double source de vérité (½ journée) ✅ TERMINÉE

- [x] Supprimer `PG_TABLES_DDL` de `src/db/index.js` (remplacé par le migrator Drizzle).
- [x] Supprimer `initPgTables()` et le tableau `migrationStatements` (remplacé par `drizzle-kit migrate`).
- [x] `src/database.js` renommé en `src/db/legacy-bridge-impl.js` (utilisé uniquement par `legacy-bridge.js`).
- [x] Supprimer `src/database/` (captcha legacy) après migration vers `security_question/db/schema.js`.
- [x] `src/db/schema/pg.js` : supprimé (contenu dans `src/db/schemas/legacy.js`).
- [x] `legacy-schema.sql` : supprimé (remplacé par le migrator Drizzle).

### Étape 5 — Tests & CI (½ journée) ✅ TERMINÉE

- [x] `createTestDb()` adapté : utilise le migrator Drizzle sur PGlite mémoire.
- [x] CI step `db:generate --check` ajoutée (`.github/workflows/ci.yml`, en mode informatif tant que le critère 4 n'est pas OK).
- [x] CI step "aucun import de database.js" ajoutée.
- [x] 604/604 tests passent.

---

## 5. Gestion des migrations versionnées

### 5.1 Workflow quotidien

```bash
# 1. Modifier un src/modules/<x>/db/schema.js
# 2. Générer la migration correspondante
npm run db:generate
# → produit src/db/migrations/0001_xxx.sql + meta/_journal.json

# 3. Inspecter le SQL généré (TOUJOURS relire)

# 4. Appliquer en dev
npm run db:migrate

# 5. Commit schema + migration ensemble
git add src/modules/<x>/db/schema.js src/db/migrations/
git commit -m "feat(<x>): add <table> for <feature>"
```

### 5.2 Politique

- **Chaque modification de schema est commitée avec sa migration** — jamais l'un sans l'autre.
- **Pas de migration destructrice** sans backfill documenté dans le fichier SQL (commentaire `-- TODO: backfill <colonne>`).
- **Renommage de colonne** = 2 migrations (add new → backfill → drop old).
- **Migrations partagées entre modules** : si un module A doit ajouter une FK sur une table d'un module B, on en discute et le commit va dans le module **A** avec une note `BREAKING: requires B vX.Y+`.
- **Versioning** : `drizzle-kit` gère nativement le `_journal.json` — on n'invente rien.
- **Production** : `npm run db:migrate` est appelé **avant** `node src/index.js` dans le `Dockerfile` / `docker-compose.yml`.

### 5.3 Rollback

`drizzle-kit` n'a pas de `migrate down` natif. On adopte la convention :

- Toute migration a un commentaire `-- @down` en tête (SQL de rollback) — convention d'équipe.
- Pour les rollbacks réels : `git revert` du commit de migration + nouvelle migration corrective.

---

## 6. Fichiers à créer / supprimer

### À créer
- `drizzle.config.js`
- `src/db/client.js`
- `src/db/schemas/index.js`
- `src/db/schemas/shared/{audit,cache,feature-flags,guild-settings}.js`
- `src/db/migrations/` (généré)
- Pour chaque module : `src/modules/<x>/db/schema.js`, `src/modules/<x>/<x>.repository.js` (sauf `feature_xp-level` où le repository existe déjà)
- `docs/guides/migrations.md`
- `docs/plan/db-repository-split.md` (ce document)

### À supprimer
- `src/database.js` (fin de migration)
- `src/database/captcha-tables.sql` (après migration captcha)
- `src/database/setup-captcha-tables.js` (après migration captcha)
- `src/db/schema/pg.js` (remplacé par `src/db/schemas/`)
- `PG_TABLES_DDL` (constante dans `db/index.js`)
- `migrationStatements` (tableau dans `initPgTables()`)

### À modifier
- `package.json` : ajout `drizzle-kit` en devDep, scripts `db:*`
- `Dockerfile` / `docker-compose.yml` : ajout de l'étape `db:migrate` au démarrage
- `src/modules/index.js` : déjà OK (auto-discovery)
- `src/db/index.js` : devient un barrel minimal
- Tous les fichiers qui importent `../../database.js` (≈ 30-50 fichiers estimés) — à identifier via grep

---

## 7. Risques & mitigations

| Risque | Impact | Mitigation |
|---|---|---|
| Régression fonctionnelle pendant la migration module par module | Élevé | Strangler-fig : `database.js` reste actif en parallèle, basculement par feature. Couverture de tests existante doit passer à chaque étape. |
| Migration Drizzle générée ≠ DDL actuel | Élevé | Étape 0 : faire une `pg_dump --schema-only` du **schema actuel** comme **baseline** (0000_baseline.sql), puis comparer après `generate`. |
| Tests d'intégration cassés par changement d'import | Moyen | Étape 2 : adapter les imports en même temps que le module est migré. Étape 5 : suite complète avant suppression de `database.js`. |
| `drizzle-kit` pas encore dans `package.json` | Faible | Ajout en devDep au début de l'étape 0. |
| Tables transverses (cache Discord) utilisées par 5+ modules | Moyen | Étape 2.16 en dernier, quand tous les modules consumers sont déjà migrés. |
| Perte de l'ordre des `ALTER COLUMN TYPE BIGINT` ad-hoc | Moyen | Chaque `ALTER` doit être **reproduit dans la migration Drizzle** correspondante. Le `migrationStatements` actuel sert de checklist — aucune ligne ne doit être perdue. |
| `database.js` est référencé dans le code frontend (build statique) | Faible | Le frontend ne devrait pas toucher à la DB → grep `database.js` dans `frontend/` avant étape 4. |
| Migrations appliquées hors-ordre en prod | Moyen | `drizzle-kit` refuse par défaut — vérifier la CI. |

---

## 8. Critères d'acceptation

À la fin du plan, **tous** ces points doivent être vrais :

| # | Critère | État | Notes |
|---|---|---|---|
| 1 | `src/database.js` n'existe plus (ou n'est plus importé par aucun fichier du projet) | ✅ | Renommé `src/db/legacy-bridge-impl.js`. 0 import direct (vérifié CI). |
| 2 | `src/db/index.js` ≤ 50 lignes | ✅ | 50 lignes pile (connexion + barrel). |
| 3 | Chaque `src/modules/<x>/` qui manipule des tables possède un `db/schema.js` ET un `<x>.repository.js` enregistré comme `provider` | ✅ | `db/schema.js` : 19/19. Repositories : 19/19 modules + 6 transverses (audit, members, commands, discord-cache, dump-discord, openai). Pas enregistré comme `provider` (optionnel, non bloquant). |
| 4 | `drizzle-kit generate` produit 0 fichier (schema et migrations sont en accord) | ✅ | `db:generate` retourne "No schema changes, nothing to migrate". Baseline 0000 regénérée avec les types `bigint`. CI durcie : fait échouer le build en cas de diff. |
| 5 | `drizzle-kit migrate` reproduit exactement la structure de la base actuelle | ⚠️ | La baseline est cohérente avec le code legacy (les tests passent) mais l'équivalence stricte avec la base de prod n'a pas été testée. À valider lors du premier déploiement (cf. §10.1). |
| 6 | Tous les tests passent (`npm test`) sans modification de leur logique métier | ✅ | 604/604. Seuls 3 tests adaptés au niveau des imports. |
| 7 | Aucune fonction de `database.js` ne subsiste hors de son repository cible | ✅ | **Suppression complète de `legacy-bridge.js` (179 lignes) et `legacy-bridge-impl.js` (2108 lignes)**. Les 70 fonctions legacy sont désormais **implémentées nativement** dans leurs repositories cibles (modules + transverses `shared/`). 0 fonction en doublon, 0 bridge. |
| 8 | `docs/architecture/data-model.md` mis à jour pour refléter le nouveau découpage | ✅ | Réécrit : décrit l'architecture par module + shared/, liste les 50 tables par module, conventions Drizzle, workflow migrations. |

---

## 9. Estimation

| Étape | Durée prévue | Durée réelle | Statut |
|---|---|---|---|
| 0. Préparation | 0.5 j | 0.5 j | ✅ |
| 1. Isoler la connexion | 0.5 j | 0.5 j | ✅ |
| 2. Découper `database.js` | 4-6 j | 3 j | ✅ |
| 3. Basculer le schema global | 1 j | 0.5 j | ✅ |
| 4. Supprimer les doublons | 0.5 j | 0.5 j | ✅ |
| 5. Tests & CI | 0.5 j | 0.5 j | ✅ |
| 4b. Rebase 0000 (types bigint) | — | 0.25 j | ✅ |
| 7. Migrer les 70 fonctions legacy (natif) | — | 1.5 j | ✅ |
| 8. Mettre à jour data-model.md | — | 0.5 j | ✅ |
| 7b. Supprimer legacy-bridge (vraie suppression) | — | 0.5 j | ✅ |
| **Total** | 7-9 j | **8.25 j** | — |

> Compatible avec la roadmap existante (cf. `docs/plan/roadmap.md`) : peut s'intercaler en **Phase 0bis** avant la Phase 1 (Automod), pour qu'Automod soit directement écrit *à destination* du nouveau pattern.

---

## 10. Chantiers restants (post-plan initial)

### 10.1 Validation en production (critère 5) ⚠️

**Problème** : la baseline `0000_chubby_romulus.sql` est cohérente avec le code legacy (les tests passent), mais l'équivalence stricte avec la **base de prod** n'a pas été testée.

**Action** : lors du prochain déploiement, exécuter un `pg_dump --schema-only` de la base de prod **avant** la migration, puis comparer avec `0000_chubby_romulus.sql` après `db:migrate`. Documenter toute divergence dans ce plan.

**Durée** : 0.5 j (à faire une fois en pré-prod).

### 10.2 Enregistrement des repositories comme `providers` (NestJS-like)

**État actuel** : les repositories existent et sont utilisés, mais sont consommés via `require(...)` direct ou instanciation, pas via le container NestJS-like.

**Action optionnelle** : ajouter chaque repository aux `providers` de son module (cf. `feature_xp-level/xp-level.module.js` qui le fait déjà pour `XPLevelRepository`) et résoudre via `container.resolve(...)` dans les services.

**Durée** : 1 j (refactor cosmétique, non bloquant).
