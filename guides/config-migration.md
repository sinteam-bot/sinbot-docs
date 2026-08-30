# Guide — Migration du système de config (c12)

> **Date** : 2026-08-29
> **Statut** : migration terminée (10 phases terminées)
> **Plan** : [`../plan/migrate-to-c12.md`](../plan/migrate-to-c12.md)
> **Audit** : [`../audit/config-usage.md`](../audit/config-usage.md)

## 1. Résumé

Le système de config a été migré de **DB (table `feature_flags`) + YAML legacy** vers **fichiers YAML gérés par c12** avec hot reload.

### Avant

```
config.yml (racine)           ← config globale
src/modules/*/config/defaults.js   ← defaults par feature (en mémoire)
DB table feature_flags          ← state runtime par guilde
```

### Après

```
data/base.config.yml           ← defaults config (versionné)
data/{NODE_ENV}.config.yml         ← config env (gitignore)
data/example/<feature>.config.yml   ← defaults code (versionné, par feature)
data/default/<feature>.config.yml   ← defaults admin (gitignore, par feature)
data/{guildId}/<feature>.config.yml ← override guilde (gitignore, par feature)
```

## 2. Architecture

### 2.1 Hiérarchie de chargement (c12)

c12 cascade automatiquement :

1. `data/example/<feature>.config.yml` (defaults code, versionné)
2. `data/default/<feature>.config.yml` (defaults admin, gitignore)
3. `data/{guildId}/<feature>.config.yml` (override guilde, gitignore)
4. `CONFIG_<FEATURE>_<KEY>` env vars (runtime override)

**Règle** : chaque niveau écrase le précédent.

### 2.2 Composant principal : `src/config/c12-loader.js`

API exposée :

| Fonction | Description |
|---|---|
| `getGlobalConfig()` | Charge la config globale depuis `data/common/*` |
| `getFeatureConfig(guildId, feature)` | Charge la config d'une feature pour une guilde (cascade) |
| `setFeatureConfig(guildId, feature, patch)` | Écrit un patch dans `data/{guildId}/<feature>.config.yml` (atomique) |
| `initGuildDataDir(guildId)` | Copie `data/default/*.config.yml` vers `data/{guildId}/` (sans écrasement) |
| `watchFeatureConfig(guildId, feature)` | Active le hot reload (dev only) |

### 2.3 Initialisation automatique

À chaque nouveau serveur (`guildCreate`), le dossier `data/{guildId}/` est créé et peuplé avec les fichiers de `data/default/`. Aucune intervention manuelle requise.

## 3. Migration pour les devs

### 3.1 Setup initial (déjà fait)

```bash
npm install c12 yaml
```

### 3.2 Générer les configs par feature

Après avoir modifié `src/modules/<feature>/config/defaults.js` :

```bash
node scripts/migrate-configs.js
```

Cela régénère `data/example/<feature>.config.yml` (versionné) et `data/default/<feature>.config.yml` (gitignore) à partir du `defaults.js`.

### 3.3 Lire une config (back-end)

```js
const { getFeatureConfig, setFeatureConfig } = require('./config/c12-loader.js');

// Lire
const cfg = await getFeatureConfig(guildId, 'invites');
if (cfg.enabled) { /* ... */ }

// Écrire (merge automatique)
await setFeatureConfig(guildId, 'invites', {
    enabled: true,
    join_log_channel_id: '1234567890'
});
```

### 3.4 Lire une config via l'API

```bash
# GET
curl http://localhost:3000/api/features/invites?guild_id=702103057898668072

# PATCH
curl -X PATCH http://localhost:3000/api/features/invites \
  -H "Content-Type: application/json" \
  -H "x-api-key: $WEB_API_KEY" \
  -d '{"enabled": true, "config": {"join_log_channel_id": "1234567890"}}'
```

## 4. Workflow admin

### 4.1 Activer une feature

1. Éditer `data/default/<feature>.config.yml` (modifiable par l'admin)
2. Mettre `enabled: true`
3. Le bot recharge automatiquement (hot reload en dev)
4. Ou redémarrer le bot (en prod)

### 4.2 Personnaliser pour un serveur spécifique

1. Éditer `data/{guildId}/<feature>.config.yml`
2. Le bot recharge automatiquement

### 4.3 Rollback

Supprimer le fichier `data/{guildId}/<feature>.config.yml` → c12 retombe sur `data/default/`.

## 5. Tests

| Test | Fichier |
|---|---|
| Wrapper c12-loader (héritage, écriture, init) | `tests/config/c12-loader.test.js` |
| FeatureRegistry (c12 backend) | `tests/feature-registry.test.js` |

```bash
npm test
```

## 6. Notes techniques

### 6.1 Pourquoi c12 ?

- **Standard unjs** : maintenu activement, écosystème riche.
- **Multi-sources** natif : cascade de fichiers + env vars + dotenv.
- **Hot reload** : `watchConfig()` prêt à l'emploi.
- **YAML, JSON5, TOML, .env** : tous supportés.

### 6.2 Pourquoi pas de DB ?

- **Simplicité** : pas de migration de schéma, pas de backup DB, pas de TTL.
- **Versionnable** : `data/default/` est dans git, l'historique est complet.
- **Hot reload** : modifier un YAML et c'est pris en compte sans restart.
- **Debuggable** : `cat data/{guildId}/invites.config.yml` → on voit la config effective.

### 6.3 Rétrocompatibilité

- L'interface publique de `FeatureRegistry` (`define`, `get`, `set`, `list`, `canUse`, `listForGuild`) est **inchangée**.
- Les 29 fichiers qui utilisaient `featureRegistry.get/set` fonctionnent **sans modification**.
- L'API REST (`/api/features/:name`) expose la même interface, mais lit/écrit maintenant des fichiers YAML au lieu de la DB.

## 7. Limitations connues

### 7.1 Pas de validation joi/zod

Les anciennes configs utilisaient `joi` (cf. `src/modules/util_invites/config/schema.js` qui a été supprimé). La validation se fait maintenant au moment de l'écriture (le `defaults.js` doit respecter le schema). Si une valeur invalide est écrite, le `defaults.js` merge peut donner un état inattendu.

**TODO** : ajouter une validation c12 (`configSchema`) dans une phase ultérieure.

### 7.2 Hot reload en prod

Le `WATCH_ENABLED` est désactivé en prod (`process.env.NODE_ENV === 'production'`). En prod, les modifications YAML prennent effet au prochain `setFeatureConfig` (qui invalide le cache manuellement). Pour les modifications manuelles, un restart est nécessaire.

### 7.3 Pas de migration automatique des `defaults.js` vers `data/`

Le script `scripts/migrate-configs.js` doit être lancé manuellement après chaque modification de `defaults.js`. À automatiser dans un futur pre-commit hook.

## 8. Fichiers créés / modifiés

| Fichier | Rôle |
|---|---|
| `src/config/c12-loader.js` | Wrapper c12 (load, save, watch, init) |
| `tests/config/c12-loader.test.js` | Tests du wrapper |
| `src/core/feature-registry.js` | Réécrit pour utiliser c12 au lieu de la DB |
| `tests/feature-registry.test.js` | Tests adaptés au nouveau backend |
| `src/index.js` | Ajout des listeners `guildCreate` + `clientReady` pour `initGuildDataDir` |
| `data/base.config.yml` | Defaults infra (versionné) |
| `data/local.config.yml` | Dev local (gitignore) |
| `data/example/*.config.yml` | 13 fichiers générés (versionnés) |
| `data/default/*.config.yml` | 13 fichiers générés (gitignore) |
| `scripts/migrate-configs.js` | Génération des configs depuis `defaults.js` |
| `db/schemas/shared/guild-settings.js` | Renommé depuis `feature-flags.js` (sans `featureFlags`) |
| `db/migrations/0000_brief_vapor.sql` | Schéma complet (avec `feature_flags` car existe en prod) |
| `db/migrations/0002_drop_feature_flags.sql` | DROP TABLE (idempotent) |
| `.gitignore` | Règles `data/*` ajoutées (commité) |
| `package.json` | `c12` et `yaml` ajoutés en deps |
| `docs/plan/migrate-to-c12.md` | Plan complet |
| `docs/audit/config-usage.md` | Audit initial |
| `docs/guides/config-migration.md` | Ce guide |

## 9. Prochaines étapes

- [ ] **Phase 10 finale** : retirer les `defaults.js` redondants (la source de vérité est maintenant `data/example/<feature>.config.yml`). Garder un fallback si le fichier YAML n'existe pas.
- [ ] **Validation joi/zod** : ajouter `configSchema` dans c12 pour valider les écritures.
- [ ] **Pre-commit hook** : regénérer automatiquement `data/example/` quand `defaults.js` change.
- [ ] **Tests d'intégration** : simuler un `guildCreate` et vérifier que `data/{guildId}/` est créé.
- [ ] **Documentation utilisateur** : un mini-guide pour les admins "comment personnaliser les features".
