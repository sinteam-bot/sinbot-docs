# Migration TypeScript — Plan futur

> **Date** : 2026-08-28
> **Status** : 📋 planifié, **non prioritaire à court terme**
> **Trigger recommandé** : dans 3-6 mois, quand le projet aura atteint une phase de stabilisation (peu de features à ajouter, peu de bugs type-related)

## 1. Pourquoi différer cette migration

| Argument | |
|---|---|
| **Coût immédiat** | 32-47h de travail (cf. estimation ci-dessous) pour **0 bug fixé** et **0 feature user livrée** |
| **Codebase actuelle** | 11 235 lignes de JavaScript (cf. `wc -l src/**/*.js`) |
| **Couverture tests** | 553 tests, 549 passent — bon filet de sécurité qui n'a pas besoin de typage strict pour fonctionner |
| **ESM déjà écarté** | La migration ESM est trop impactante ; faire les deux en parallèle multiplierait les risques |
| **Dette technique réelle** | Faible (les bugs type-related sont rares dans le code actuel, le code est plutôt défensif) |
| **Stack frontend** | TypeScript natif (Nuxt 3 + Vue 3 TS-first). Le backend sera aligné |

## 2. Quand lancer la migration

| Signal | Action |
|---|---|
| Plus de 20 fichiers dans `src/` | Migration commence à être utile (dette de typage) |
| Bugs récurrents `Cannot read property of undefined` | TypeScript aurait attrapé en CI |
| Refactoring risqué (ex: changement de signature d'un service partagé) | TS = filet de sécurité |
| Onboarding de nouveaux devs | TS accélère la prise en main (autocomplete, types en lecture) |
| Stagnation des features à ajouter (focus sur le polish) | Bon moment pour durcir le code |

## 3. Plan d'exécution (à exécuter en sprint dédié de 1-2 semaines)

### Étape 1 — Setup (2h)

```bash
npm install --save-dev typescript tsx @types/node @types/express
```

Créer `tsconfig.json` en mode **incrémental** :

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "moduleResolution": "node",
    "lib": ["ES2022"],
    "allowJs": true,         // <-- clé : autorise les .js existants
    "checkJs": false,        // pas de check TS sur les .js (rapide)
    "outDir": "./dist",
    "strict": false,         // on durcira plus tard
    "esModuleInterop": true,
    "skipLibCheck": true,
    "noEmitOnError": false,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "frontend", "tests"]
}
```

`package.json` :
```jsonc
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "dev:ts": "tsx watch src/index.js",      // runtime via tsx (ESM-ish)
    "start:ts": "tsx src/index.js"
  }
}
```

### Étape 2 — JSDoc annotations (8-12h, parallèle au dev)

Pas de migration .ts/.js, mais ajout de **types JSDoc** sur les services critiques. VsCode/IntelliJ les reconnaissent en autocomplétion sans aucune compilation.

```js
/**
 * @typedef {Object} Balance
 * @property {string} userId
 * @property {string} guildId
 * @property {number} balance
 * @property {number} bankBalance
 */

/**
 * Crée un rappel DM pour un user
 * @param {Object} opts
 * @param {string} opts.userId
 * @param {string} opts.guildId
 * @param {string} opts.text
 * @param {number} opts.fireAt    epoch ms
 * @returns {Promise<{ok: boolean, error?: string, data?: any}>}
 */
async function createReminder(opts) { ... }
```

C'est **le ROI le plus élevé** pour 2 jours de travail : on a les types dans l'IDE sans toucher au runtime.

### Étape 3 — Mode strict (4h, après 1 semaine d'utilisation des types JSDoc)

```jsonc
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true
}
```

Lance `tsc --noEmit`, **corrige les erreurs** une par une (généralement ~50-100 corrections).

### Étape 4 — Migration .ts fichier-par-fichier (15-25h, sur 2-3 semaines)

1. **Feuilles d'abord** : repositories, services purs (sans discord.js direct)
   - `card-renderer.service.js` (pur)
   - `reports.repository.js`
   - `sticky-roles.repository.js`
   - `temp-voice.repository.js`
   - `birthday.repository.js`
2. **Services intermédiaires** : services qui dépendent des feuilles
3. **Controllers** (avec types Request/Response)
4. **Commands** (avec les types de discord.js)
5. **`src/index.js` en dernier** (le top-level qui orchestre tout)

À chaque étape : `mv foo.js foo.ts` puis ajouter les types explicites. `allowJs: true` permet de garder les .js intacts pendant la transition.

### Étape 5 — Tests TypeScript (2-3h)

Convertir les tests critiques en `.test.ts` (`.test.js` continue de marcher grâce à `allowJs`). `vitest` supporte nativement TS.

### Étape 6 — CI (1h)

Ajouter un step `tsc --noEmit` dans `.github/workflows/*.yml`. Si erreur, le build échoue.

## 4. Estimation totale

| Étape | Effort | Quand |
|---|---:|---|
| 1. Setup | 2h | Sprint dédié |
| 2. JSDoc | 8-12h | Parallelisable (dès aujourd'hui) |
| 3. Strict | 4h | Après 1 semaine de JSDoc |
| 4. Migration .ts | 15-25h | Sur 2-3 semaines |
| 5. Tests | 2-3h | Pendant étape 4 |
| 6. CI | 1h | Final |
| **Total** | **32-47h** | Sprint dédié de 1-2 semaines |

## 5. Bénéfices attendus

| Bénéfice | Quantification |
|---|---|
| Type-safety compile-time | 0 `TypeError` runtime dans le code migré |
| Refactoring plus sûr | 0 régression silencieuse sur changement de signature |
| Autocomplete IDE | x3-5 vitesse de développement (subjectif) |
| Documentation implicite | Les types = doc exécutable |
| Cohérence frontend/backend | Le frontend est déjà en TS, le backend le sera aussi |

## 6. Co-bénéfice avec ESM

Les deux migrations (TS + ESM) peuvent être **complémentaires** :
- Une fois en TS, on peut utiliser `import type` et `import` correctement typés
- L'ESM devient un choix plus simple (le tooling TS gère le bundling)
- Mais on a déjà écarté ESM → on ne fait que TS

## 7. Conclusion

**On ne lance PAS la migration TypeScript maintenant.** On l'a planifiée, budgétée, et on a un trigger clair pour la lancer (cf. section 2).

**En attendant** : on peut commencer par l'**étape 2 (JSDoc)** dès aujourd'hui car elle ne casse rien et améliore l'IDE. C'est ~2 jours de travail pour un gain immédiat.

**Quand on lance** : prendre un sprint dédié (1-2 semaines) après stabilisation des features, en utilisant `allowJs: true` pour une migration incrémentale sans big-bang.

## 8. Voir aussi

- `cjs-to-esm.md` — analyse de la migration ESM (écartée pour le moment)
- `audit-draftbot.md` — backlog des features Draftbot
- `backlog.md` — backlog global
