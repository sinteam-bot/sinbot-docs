# Plan — Migration du système de config vers c12

> **Date** : 2026-08-29
> **Origine** : complexité croissante de la config (DB + YAML + bridge `featureRegistry`) + bugs récurrents (DB désynchronisée, lecture depuis `config.yml` qui ne reflète pas la DB, etc.)
> **Objectif** : unifier la config sur fichiers YAML, multi-niveaux, avec [c12](https://github.com/unjs/c12) comme loader, hot reload, et plus de table `feature_flags` en base.

---

## 1. Choix validés (décisions utilisateur)

| Question | Choix |
|---|---|
| Statut `enabled` des features | Stocké **dans le YAML** de chaque feature (pas de tracking DB) |
| Format c12 | **c12 + custom loader** multi-fichiers (un par feature, par guilde) |
| Héritage des configs | `default/` = fallback admin (modifiable), `example/` = versionné dev (jamais touché par l'admin) |
| Hot reload | **Oui**, via `c12({ watch: true })` |
| Format | **YAML** (lisible, typable) |
| Découpage | **En plusieurs phases/PRs** avec commits progressifs |

---

## 2. Architecture cible

### 2.1 Structure des fichiers

```
data/
├── base.config.yml           # valeurs par défaut (versionné, infra)
├── production.config.yml     # overrides prod (auto si NODE_ENV=production)
├── test.config.yml           # overrides test (auto si NODE_ENV=test)
└── local.config.yml          # .gitignore, overrides dev local
├── default/                # ⚠️ GITIGNORÉ - templates par feature, modifiables par l'admin
│   ├── xp.config.yml
│   ├── captcha.config.yml
│   ├── invites.config.yml
│   └── ...
├── example/                # VERSIONNÉ - exemples pour le dev, jamais touché par l'admin
│   ├── xp.config.example.yml
│   └── ...
└── {guildId}/              # ⚠️ GITIGNORÉ - un dossier par serveur, créé au guildCreate
    ├── xp.config.yml
    ├── captcha.config.yml
    └── ...
```

### 2.2 Règle de chargement (par feature, par guilde)

Ordre de priorité (du moins prioritaire au plus prioritaire) :

```
1. data/example/<feature>.config.example.yml    # valeurs par défaut du code
2. data/default/<feature>.config.yml             # valeurs admin (override example)
3. data/{guildId}/<feature>.config.yml           # override guilde
4. env vars: CONFIG_<FEATURE>_<KEY>              # runtime override
```

c12 gère cette cascade nativement via `defaults` + `sources`.

### 2.3 Évolution du `FeatureRegistry`

L'interface publique reste identique (`define`, `get`, `set`, `list`) mais le backend passe de DB à c12 :

```
AVANT : FeatureRegistry → table `feature_flags` (DB)
APRÈS : FeatureRegistry → c12 loader → fichier YAML
```

---

## 3. Phases ordonnées

### Phase 0 — Audit & inventaire (~0.5 j)

**Livrables** :
- [ ] Liste de **tous les `featureRegistry.define(...)`** (16 modules)
- [ ] Liste de **tous les `featureRegistry.get(...)`** (services, controllers, commands, listeners)
- [ ] Liste de **tous les `featureRegistry.set(...)`** (PATCH API)
- [ ] Liste de **tous les `defaults.js`** par feature
- [ ] Compteur des références à la table `feature_flags` (DB)
- [ ] Mapping `config.yml` (racine) → `data/local.config.yml`

**Critère de succès** : document `docs/audit/config-usage.md` à jour.

### Phase 1 — Installer c12 + scaffolding (~1 h)

**Livrables** :
- [ ] `npm install c12`
- [ ] `src/config/c12-loader.js` : wrapper squelette (sans logique métier)
- [ ] `tests/config/c12-loader.test.js` : tests squelette (skipped)
- [ ] Commit : `chore(deps): add c12`

**Critère de succès** : `c12` dans `package.json`, build OK.

### Phase 2 — Migration config commune (~0.5 j)

**Livrables** :
- [ ] `data/base.config.yml` : valeurs par défaut infra (DB pool, logger, port API)
- [ ] `data/local.config.yml` : overrides dev — `.gitignore`
- [ ] `src/config/index.js` : `getConfig()` lit via c12 (au lieu de `require('config.yml')`)
- [ ] Tous les tests existants qui dépendent de `getConfig()` doivent passer
- [ ] `gitignore` mis à jour : `data/local.config.yml`, `data/{guildId}/`
- [ ] Commit : `refactor(config): migrate global config to c12`

**Critère de succès** : 615/615 tests backend passent, le bot démarre, la config se charge.

### Phase 3 — Script de migration des defaults.js (~1 j)

**Livrables** :
- [ ] `scripts/migrate-configs.js` : scanne tous les `src/modules/<feature>/config/defaults.js`, génère `data/example/<feature>.config.example.yml` (versionné)
- [ ] Génère aussi `data/default/<feature>.config.yml` (copie identique, `.gitignore`)
- [ ] Exécuté une seule fois, résultat commité pour les fichiers `example/`
- [ ] Commit : `chore(config): generate example configs from defaults.js`

**Critère de succès** : 16 fichiers YAML générés (1 par feature), tous versionnés dans `data/example/`.

### Phase 4 — Wrapper c12-loader.js complet (~1 j)

**Livrables** :
- [ ] `getGlobalConfig()` : charge `data/*.config.yml` avec c12 (cascade base → env)
- [ ] `getFeatureConfig(guildId, feature)` : charge le YAML fusionné de la feature (example → default → guild)
- [ ] `setFeatureConfig(guildId, feature, patch)` : écriture atomique (temp + rename)
- [ ] `initGuildDataDir(guildId)` : copie `data/default/<feature>.*` → `data/{guildId}/<feature>.*` (sans écraser)
- [ ] Hot reload via `c12({ watch: true })` → émet `feature.updated` sur EventBus
- [ ] Tests unitaires : héritage, hot reload, écriture atomique, initGuildDataDir
- [ ] Commit : `feat(config): c12 multi-file loader with hot reload`

**Critère de succès** : tous les tests passent, hot reload fonctionne (modifier un YAML → l'event est émis).

### Phase 5 — Brancher `guildCreate` (~0.5 j)

**Livrables** :
- [ ] Event listener sur `client.on('guildCreate', ...)` dans `src/index.js`
- [ ] Appelle `initGuildDataDir(guild.id)` à l'arrivée d'un nouveau serveur
- [ ] Log : `📁 [DataDir] Initialisé ./data/{guildId}/ avec N fichiers`
- [ ] Test : simuler un guildCreate et vérifier que le dossier est créé
- [ ] Commit : `feat(config): auto-init data dir on guildCreate`

**Critère de succès** : test d'intégration passe, logs corrects au boot d'un nouveau serveur.

### Phase 6 — Réécrire FeatureRegistry (~1 j)

**Livrables** :
- [ ] `src/core/feature-registry.js` : 
  - `define()` : ne stocke plus en `this.features`, passe les aliases à c12
  - `get()` : appelle `getFeatureConfig(guildId, name)` (c12) au lieu de la DB
  - `set()` : appelle `setFeatureConfig(guildId, name, patch)` (c12) + émet `feature.updated`
  - `list()` : lit les `data/default/` pour lister les features disponibles
- [ ] Plus de `this._dbAvailable` ni de requêtes SQL
- [ ] Tests : `feature-registry.test.js` mis à jour
- [ ] Commit : `refactor(feature-registry): replace DB with c12 file backend`

**Critère de succès** : 615/615 tests passent, plus aucune référence à `feature_flags` dans le code.

### Phase 7 — Migration API REST (~0.5 j)

**Livrables** :
- [ ] `src/web/featuresRouter.js` : 
  - `GET /api/features/:name` → `getFeatureConfig(guildId, name)`
  - `PATCH /api/features/:name` → `setFeatureConfig(guildId, name, patch)`
- [ ] Deprecated : `POST /api/config` (legacy) → redirige vers c12
- [ ] Tests : `tests/web/features.test.js` (à créer ou adapter)
- [ ] Commit : `refactor(api): routes use c12 file backend`

**Critère de succès** : toutes les routes API fonctionnent avec le backend c12, curl/PowerShell testent OK.

### Phase 8 — Suppression table feature_flags (~0.5 j)

**Livrables** :
- [ ] Migration Drizzle : `DROP TABLE feature_flags;`
- [ ] Suppression du barrel `src/db/schemas/legacy.js` (section featureFlags)
- [ ] Suppression du fichier `src/db/schemas/shared/feature-flags.js`
- [ ] `npm run db:generate` produit une migration `0002_drop_feature_flags.sql` (ou le tag approprié)
- [ ] Tests : vérifier qu'aucun test n'utilise la table
- [ ] Commit : `chore(db): drop feature_flags table`

**Critère de succès** : `db:generate` retourne "No schema changes", la table n'est plus dans le schema.

### Phase 9 — Mise à jour dashboard frontend (~1 j)

**Livrables** :
- [ ] `frontend/composables/useInvites.ts` : `updateConfig` appelle `PATCH /api/config/invites` (nouvelle URL, rétrocompatible via redirect côté backend)
- [ ] `frontend/composables/useCaptcha.ts` : idem
- [ ] `frontend/composables/useFeatures.ts` : adapter
- [ ] Test manuel : sauvegarder une config → recharger la page → la valeur est conservée
- [ ] Commit : `refactor(frontend): use new config API endpoint`

**Critère de succès** : les pages `/modules/*/config` sauvegardent et rechargent correctement.

### Phase 10 — Tests & documentation finale (~1 j)

**Livrables** :
- [ ] Tests d'intégration : ajout d'un nouveau serveur → data dir créé
- [ ] Tests de migration : un `defaults.js` manquant est correctement récupéré depuis `data/example/`
- [ ] Documentation `docs/guides/config-migration.md` : nouveau guide
- [ ] Mise à jour de `docs/architecture/data-model.md`
- [ ] Mise à jour de `docs/plan/feature-registry.md` (à créer)
- [ ] Commit : `docs: add config migration guide and update architecture`

**Critère de succès** : tests d'intégration passent, doc à jour, équipe onboardée.

---

## 4. Gitignore à modifier

```gitignore
# Config dynamique (générée à l'exécution)
data/*.config.yml
data/{guildId}/

# Templates admin (modifiables, pas versionnés)
data/default/
```

À conserver (versionné) :
- `data/base.config.yml` (infra)
- `data/example.config.yml` (example pour env)
- `data/example/` (exemples dev, jamais touchés par l'admin)

---

## 5. Estimation globale

| Phase | Durée |
|---|---|
| 0. Audit & inventaire | 0.5 j |
| 1. Installer c12 + scaffolding | 1 h |
| 2. Migration config commune | 0.5 j |
| 3. Script migration defaults.js | 1 j |
| 4. Wrapper c12-loader.js complet | 1 j |
| 5. Brancher guildCreate | 0.5 j |
| 6. Réécrire FeatureRegistry | 1 j |
| 7. Migration API REST | 0.5 j |
| 8. Suppression table feature_flags | 0.5 j |
| 9. Mise à jour dashboard frontend | 1 j |
| 10. Tests & documentation | 1 j |
| **Total** | **7.5 à 8 j** |

C'est ~1.5 sprints. Chaque phase = 1 commit (ou plus) avec un message clair.

---

## 6. Risques et mitigations

| Risque | Impact | Mitigation |
|---|---|---|
| **Race condition** lecture/écriture concurrente | Élevé | Écriture atomique : `writeFile(tmp)` + `rename(tmp, dest)` |
| **Hot reload** en prod (modification accidentelle) | Moyen | Watch activé en dev uniquement (`process.env.NODE_ENV !== 'production'`) |
| **Perte de data** si disque plein | Élevé | Backup automatique avant chaque write (`.bak` file) + log d'erreur |
| **Permissions** sur `./data/{guildId}/` | Moyen | `fs.mkdir({ recursive: true, mode: 0o755 })` |
| **Migration ratée** (oubli d'une feature) | Élevé | Script `scripts/migrate-configs.js` qui scanne tous les `defaults.js` automatiquement |
| **Backward compat** : code qui utilise encore l'ancien `FeatureRegistry` interne | Élevé | Wrapper de compat pendant 1 release, marqué deprecated, supprimé après |
| **Tests lents** à cause de lectures fichiers | Faible | Cache LRU en mémoire, invalidé sur write |
| **Ordre de chargement** des features c12 | Faible | Documenter dans le loader avec un test qui vérifie l'ordre |

---

## 7. Notes d'implémentation

### 7.1 Wrapper c12 minimal (squelette)

```js
// src/config/c12-loader.js
const { loadConfig, writeConfig } = require('c12');
const path = require('path');
const fs = require('fs').promises;

const DATA_DIR = path.resolve(__dirname, '../../data');

async function getFeatureConfig(guildId, feature) {
    return loadConfig({
        cwd: path.join(DATA_DIR, String(guildId)),
        name: feature,
        defaults: await loadConfig({
            cwd: path.join(DATA_DIR, 'default'),
            name: feature
        }),
        sources: [
            { ext: 'yml' }
        ]
    });
}

async function setFeatureConfig(guildId, feature, patch) {
    const filePath = path.join(DATA_DIR, String(guildId), `${feature}.config.yml`);
    const current = await getFeatureConfig(guildId, feature);
    const next = { ...current, ...patch };
    // Écriture atomique
    const tmp = `${filePath}.tmp`;
    await fs.writeFile(tmp, JSON.stringify(next, null, 2));
    await fs.rename(tmp, filePath);
}

async function initGuildDataDir(guildId) {
    const targetDir = path.join(DATA_DIR, String(guildId));
    await fs.mkdir(targetDir, { recursive: true });
    const defaultDir = path.join(DATA_DIR, 'default');
    const files = await fs.readdir(defaultDir);
    for (const f of files) {
        const dest = path.join(targetDir, f);
        try {
            await fs.access(dest);
        } catch {
            await fs.copyFile(path.join(defaultDir, f), dest);
        }
    }
}

module.exports = { getFeatureConfig, setFeatureConfig, initGuildDataDir };
```

### 7.2 Hot reload (c12 watch)

c12 supporte `watch: true` nativement :

```js
const { loadConfig } = require('c12');
const { eventBus } = require('../core/event-bus.js');

const watchers = new Map();

function watchFeature(guildId, feature) {
    const key = `${guildId}:${feature}`;
    if (watchers.has(key)) return;
    const stop = loadConfig({
        cwd: path.join(DATA_DIR, String(guildId)),
        name: feature,
        watch: true,
        onUpdate: (newCfg) => {
            console.log(`🔄 [Config] ${key} rechargé`);
            eventBus.emit('feature.updated', { guildId, name: feature, enabled: newCfg.enabled });
        }
    });
    watchers.set(key, stop);
}
```

### 7.3 Migration `defaults.js` → YAML

Le script `scripts/migrate-configs.js` :

```js
const fs = require('fs');
const path = require('path');
const yaml = require('yaml'); // déjà installé via drizzle

const MODULES_DIR = path.resolve(__dirname, '../src/modules');
const EXAMPLES_DIR = path.resolve(__dirname, '../data/example');
const DEFAULTS_DIR = path.resolve(__dirname, '../data/default');

const modules = fs.readdirSync(MODULES_DIR, { withFileTypes: true })
    .filter(d => d.isDirectory() && d.name.startsWith('feature_'));

for (const m of modules) {
    const featureName = m.name.replace('feature_', '').replace(/-/g, '_');
    const defaultsPath = path.join(MODULES_DIR, m.name, 'config/defaults.js');
    if (!fs.existsSync(defaultsPath)) continue;
    
    // Charger le defaults.js (CommonJS)
    delete require.cache[require.resolve(defaultsPath)];
    const defaults = require(defaultsPath);
    
    // Convertir en YAML
    const yamlContent = yaml.stringify(defaults);
    
    // Écrire dans example/ (versionné)
    const examplePath = path.join(EXAMPLES_DIR, `${featureName}.config.example.yml`);
    fs.writeFileSync(examplePath, `# ${featureName} config\n# Generated from src/modules/${m.name}/config/defaults.js\n# DO NOT EDIT MANUALLY - edit the defaults.js instead\n\n${yamlContent}`);
    
    // Écrire dans default/ (.gitignore)
    const defaultPath = path.join(DEFAULTS_DIR, `${featureName}.config.yml`);
    fs.writeFileSync(defaultPath, yamlContent);
    
    console.log(`✓ ${featureName}.config.yml généré`);
}
```

---

## 8. Jalons (milestones)

- **Jalon 1** (fin Phase 2) : `config.yml` migré vers c12, tests passent, bot démarre avec config chargée depuis `data/`
- **Jalon 2** (fin Phase 3) : 16 fichiers `data/example/<feature>.config.example.yml` générés et versionnés
- **Jalon 3** (fin Phase 4) : c12-loader complet avec hot reload, testé unitairement
- **Jalon 4** (fin Phase 6) : `FeatureRegistry` réécrit, plus aucune référence DB pour la config
- **Jalon 5** (fin Phase 8) : table `feature_flags` supprimée, schema Drizzle propre
- **Jalon 6** (fin Phase 10) : dashboard frontend migré, doc à jour, tests d'intégration passent

---

## 9. Rollback strategy

Si la migration échoue en cours de route, on peut rollback par phase :

- **Phase 1-2** : `npm uninstall c12`, restaurer `src/config/index.js` depuis git
- **Phase 3-4** : `git checkout data/example/`, supprimer `data/default/`, supprimer `c12-loader.js`
- **Phase 5-6** : restaurer `FeatureRegistry` (DB), supprimer l'event listener `guildCreate`
- **Phase 7-8** : restaurer l'API REST, `npm run db:generate` produit une migration inverse
- **Phase 9-10** : restaurer les composables frontend

Chaque commit est atomique et revertable.

---

## 10. Démarrage

Commencer par la **Phase 0** (audit) qui est sans risque et fournit la cartographie nécessaire pour estimer plus précisément les phases suivantes.
