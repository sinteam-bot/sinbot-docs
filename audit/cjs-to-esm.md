# Audit migration CJS → ESM (CommonJS → ECMAScript Modules)

> **Date** : 2026-08-28
> **Demande** : estimer l'impact d'une migration du projet de CJS vers ESM natif (Node 22+).
> **Décision validée par l'utilisateur** : approche **progressive par fichier** avec **dual-mode** (`"type": "module"` + rename `.cjs` pour le legacy).

## 1. État actuel (mesures directes du code)

### 1.1 Type de module

| Champ | Valeur |
|---|---|
| `package.json` `type` | `commonjs` (absent) |
| `package.json` `main` | `src/index.js` |
| `package.json` `engines.node` | n/a (non spécifié) |
| Runtime testé | Node 22 (PGlite, vitest) |

### 1.2 Compteurs dans le code (`src/` + `tests/` + `scripts/`)

| Métrique | Valeur |
|---|---:|
| Fichiers `.js` au total (src) | 196 |
| Fichiers utilisant `require(...)` | 175 |
| Appels `require(...)` au total | 781 |
| `module.exports` au total | 195 |
| `__dirname` / `__filename` | 6 occurrences (5 fichiers) |
| `require.resolve` / `require.cache` | 0 (pas d'usage) |

### 1.3 Répartition par dossier

| Dossier | require() | module.exports | Fichiers |
|---|---:|---:|---:|
| `src/core` | 21 | 6 | 6 |
| `src/modules` | 589 | 157 | 157 |
| `src/services` | 14 | 5 | 5 |
| `src/utils` | 27 | 12 | 11 |
| `src/web` | 64 | 3 | 3 |
| `src/db` | 9 | 3 | 3 |
| `tests/` | 152 | 0 | 43 |
| `scripts/` | 6 | 0 | 1 |
| `frontend/` | (à part, déjà Vue/Nuxt ESM) | | |

### 1.4 Dépendances — compatibilité ESM

| Dep | Type natif | Entrypoint CJS | Verdict |
|---|---|---|---|
| `discord.js` | CJS | (n/a) | ⚠️ CJS-only, pas d'ESM officiel |
| `discord-api-types` | CJS | n/a | ✅ utilisable depuis ESM |
| `discord-api-types/v10` | CJS | n/a | ✅ |
| `@discordjs/rest` | CJS | n/a | ✅ |
| `drizzle-orm` | ESM (`type: module`) | `./index.cjs` | ✅ interop parfait |
| `drizzle-orm/pg-core` | ESM | (héritent) | ✅ |
| `@electric-sql/pglite` | ESM | `dist/index.cjs` | ✅ |
| `express` | CJS | n/a | ✅ (utilisable depuis ESM) |
| `express-rate-limit` | CJS | n/a | ✅ |
| `helmet` | CJS | n/a | ✅ |
| `cookie-parser` | CJS | n/a | ✅ |
| `better-sqlite3` | CJS (bindings natifs) | n/a | ⚠️ natif CJS |
| `pg` | CJS | n/a | ✅ |
| `node-cron` | CJS | n/a | ✅ |
| `jsonwebtoken` | CJS | n/a | ✅ |
| `js-yaml` | CJS (`type: commonjs`) | n/a | ✅ interop |
| `openai` | ESM (`type: module`) | dual export | ✅ |
| `dotenv` | CJS | n/a | ✅ |
| `vitest` | ESM | dual | ✅ |
| `vite` | ESM | dual | ✅ |

**Conclusion deps** : toutes les dépendances sont soit CJS-compatibles, soit ESM-natives avec un fallback CJS. **Aucun blocker.**

## 2. Les 3 patterns de migration possibles

### Option A — `type: "module"` + rename bulk `.js` → `.mjs`

| Aspect | Impact |
|---|---|
| Setup | `package.json: "type": "module"` + renommage 196 fichiers |
| Charge | **Énorme** (196 renames + tous les `require()` → `import`) |
| **Discord.js** | **Bloqueur** : `discord.js` est CJS-only, pas d'entrypoint ESM |
| Risque régression | **Très haut** (toute la base de code changée d'un coup) |
| Rollback | **Difficile** (revert massif) |
| **Verdict** | ❌ **Rejeté** par l'utilisateur (trop brutal) |

### Option B — Renommer seulement les nouveaux fichiers en `.mjs`

| Aspect | Impact |
|---|---|
| Setup | Aucun (on garde `"type": "commonjs"`) |
| Discord.js | ✅ OK |
| Rollback | Trivial (rename back) |
| **Verdict** | ❌ Trop lent, pas de momentum |

### Option C (retenue) — Dual-mode avec `"type": "module"` + rename progressif `.cjs`

| Aspect | Impact |
|---|---|
| Setup | `package.json: "type": "module"` + **tous les nouveaux fichiers en `.mjs`** (ou `.js` mais ESM) + **legacy CJS en `.cjs`** |
| Discord.js | ✅ OK (`discord.js` reste chargé via `import()` dynamic ou `createRequire`) |
| Rollback | Trivial (rename `.mjs` → `.cjs` + reverter `package.json`) |
| Migration | **Progressive, fichier par fichier** |
| Tests | ✅ `vitest` continue de marcher (il charge CJS et ESM) |
| **Verdict** | ✅ **Recommandé** |

## 3. Plan de migration concret (Option C)

### 3.1 Étape 0 — prérequis (1h)

- Ajouter `"type": "module"` dans `package.json` (mais c'est **incompatible** avec `discord.js` qui est CJS-only)
- **Alternative** : ne PAS mettre `type: module` dans le `package.json` racine. À la place, créer un sous-`package.json` dans `src/core/` avec `"type": "module"` → seul ce dossier devient ESM. Cette approche est rare mais évite de toucher au reste.

**Recommandation finale** : **ne PAS mettre `"type": "module"` globalement** (à cause de discord.js). Procéder par **dual-mode fichier-par-fichier** :

- Quand un fichier devient ESM, le **renommer en `.mjs`** et le réécrire (le plus simple)
- OU ajouter `"type": "module"` dans un `package.json` local (pour les sous-dossiers) — la méthode Node 22 recommande le rename + `.mjs`

### 3.2 Stratégie de rename adoptée

Pour **chaque** module CJS, on le convertit ainsi :

1. **Renommer** `foo.js` → `foo.mjs`
2. **Convertir** toutes les dépendances internes :
   - `const x = require('./y')` → `import x from './y.js'`
   - `const { y } = require('./z')` → `import { y } from './z.js'`
   - `const x = require('discord.js')` → `import x from 'discord.js'` (CJS interop OK)
   - `module.exports = { foo, bar }` → `export { foo, bar }`
   - `module.exports = function() {}` → `export default function() {}`
3. **Pas d'incidence** sur discord.js : `import { Client } from 'discord.js'` fonctionne (interop CJS)

### 3.3 Ordre de migration (fichiers sans dépendances d'abord)

1. **Feuilles d'abord** (1 par 1, ceux qui n'ont pas de dépendances locales)
2. **Puis les middlewares** (FeatureRegistry, EventBus, Container)
3. **Puis les services** (cards, reports, temp-voice, etc.)
4. **Puis les modules** (feature_reports, feature_economy, etc.)
5. **`src/index.js` en dernier** (le top-level qui charge tout)

### 3.4 Estimation de l'effort

| Étape | Fichiers | Effort |
|---|---:|---|
| Prérequis + helper (bot) | 1 | 30 min |
| Conversion des services purs (cards, reports, temp-voice, shop, sticky-roles, info) | ~10 | 2-3h |
| Conversion des repositories (drizzle + abstractions DB) | ~5 | 1h |
| Conversion des middlewares core (FeatureRegistry, EventBus, Container, decorators) | ~6 | 1-2h |
| Conversion des modules (controllers + commands + listeners) | ~50 | 4-6h |
| Conversion de `src/index.js` et tests | 1 + 43 | 1-2h |
| **Total** | **~116 fichiers** | **10-15h** |

### 3.5 Risques et mitigations

| Risque | Mitigation |
|---|---|
| Discord.js ESM-import bugs | Garder discord.js chargé via `createRequire` dans un module `.cjs` qui ré-exporte |
| Cycle d'imports (A → B → A) | Éviter en convertissant les feuilles d'abord |
| `__dirname` / `__filename` indisponibles en ESM | Remplacer par `fileURLToPath(import.meta.url)` |
| Tests qui cassent | `vitest` charge CJS et ESM, mais un test qui require un module CJS qui require un module ESM ne marche pas. Garder les tests en CJS le plus longtemps possible. |
| `__dirname` pour `path.join(__dirname, ...)` | Remplacer par `import.meta.url` + `fileURLToPath` |

### 3.6 Bénéfices attendus

| Bénéfice | Quantification |
|---|---|
| **Type-safety** | `import { foo } from './bar.js'` vs `require('./bar')` — erreurs de typage détectées plus tôt (si on couple avec TypeScript) |
| **Tree-shaking** | Webpack / Vite / esbuild peuvent mieux éliminer le code mort |
| **Top-level await** | Possibilité d'initialiser une DB asynchrone au top-level d'un module |
| **Standard moderne** | ESM est le standard depuis Node 14 (sortie 2020) ; ESM-only progressivement (ex: `node:test` natif) |
| **ESLint / Prettier** | Meilleure compréhension des imports, plus de `require.resolve` hacks |

### 3.7 Inconvénients attendus

- **Migration lourde** (10-15h, pas de gain immédiat pour les users finaux)
- **Discord.js** : tant qu'il n'a pas d'ESM officiel, on doit passer par `import` qui **transpile en CJS** sous le capot (perf négligeable, mais pas de gain perf)
- **Risque de régression** : des modules qui ne sont pas testés peuvent casser
- **Coût de maintenance** : plus de `__dirname`, plus de `require.resolve`, plus de `module.exports` mixés dans la même base de code

## 4. Ma recommandation

1. **Phase A — Setup** (1h)
   - Pas de `"type": "module"` global (à cause de discord.js)
   - Créer un helper `src/utils/import.cjs` qui wrap `createRequire` pour les fichiers `.mjs` qui ont besoin de charger un module CJS arbitraire
   - Documenter la convention (`.mjs` = ESM, `.cjs` = CJS, `.js` = legacy CJS)

2. **Phase B — Conversion feuille par feuille** (10-15h)
   - Commencer par les services purs (cards, reports, temp-voice, shop, sticky-roles, info)
   - Puis les repositories
   - Puis les middlewares core
   - Puis les modules
   - `src/index.js` en dernier

3. **Phase C — Tests et CI** (2-3h)
   - `vitest` couvre déjà les deux modes (CJS + ESM)
   - Ajouter un test E2E qui charge `src/index.js` après conversion
   - Vérifier que la CI passe en ESM-only (si possible)

4. **Phase D — Cleanup final** (1h)
   - Renommer tous les `.js` legacy en `.cjs` une fois convertis (sauf discord.js qui reste CJS natif)
   - Mettre à jour la documentation

**Total** : ~15-20h de travail, à étaler sur plusieurs sessions.

## 5. Ce que je peux faire maintenant (build)

Si tu me donnes le feu vert, je peux commencer par la **Phase A** : ajouter le helper `import.cjs` et convertir **un seul service pur** (par exemple `src/modules/util_info/services/info.service.js` qui n'a aucune dépendance discord.js) en `.mjs` pour valider la procédure.

Sinon, je peux continuer à implémenter des features (Starboards, Sauvegardes, Commandes fun) en CJS.
