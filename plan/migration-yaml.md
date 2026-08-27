# Migration YAML → `features.*` (FeatureRegistry)

> **Objectif** : migrer progressivement la configuration YAML existante vers le nouveau format `features.*` pilotable par DB et par le dashboard, **sans casser** les déploiements en production.

## 1. État actuel (avant migration)

`config.yml` contient des blocs top-level par feature :

```yaml
captcha:
  enabled: true
  ...
welcome:
  enabled: true
  ...
xp:
  enabled: false
  ...
counter:
  enabled: true
  ...
```

Chaque module lit son bloc via `getConfig().<featureName>`.

## 2. Cible

Deux formats coexistent, avec une **précédence claire** :

```
1. Base de données (feature_flags)         ← plus haute priorité
2. config.yml > features.<name>            ← surcharge DB si DB vide
3. config.yml > <name>                     ← legacy, lu par défaut
4. Defaults définis dans le code           ← fallback final
```

## 3. Stratégie de migration

### Étape 1 — Wrapper de lecture (transparent)

Créer un helper `getFeatureConfig(name)` dans `src/core/feature-registry.js` qui :
1. Tente de lire `feature_flags` en DB
2. Sinon, lit `config.features.<name>` dans le YAML
3. Sinon, lit `config.<name>` (legacy)
4. Sinon, retourne les `defaults` du code

```js
async function getFeatureConfig(name) {
  const cached = cache.get(name);
  if (cached && Date.now() - cached.at < 30_000) return cached.value;

  // 1. DB
  const dbRow = await db.select().from(featureFlags)
    .where(and(eq(featureFlags.guildId, GUILD_ID), eq(featureFlags.featureName, name)))
    .limit(1);
  if (dbRow[0]) {
    const value = JSON.parse(dbRow[0].config_json);
    cache.set(name, { at: Date.now(), value });
    return value;
  }

  // 2. YAML features.*
  const yamlFeatures = getConfig().features || {};
  if (yamlFeatures[name] !== undefined) {
    return { ...DEFAULT_CONFIG[name], ...yamlFeatures[name] };
  }

  // 3. YAML legacy
  const yamlLegacy = getConfig()[name];
  if (yamlLegacy !== undefined) {
    return { ...DEFAULT_CONFIG[name], ...yamlLegacy };
  }

  // 4. Defaults code
  return DEFAULT_CONFIG[name];
}
```

### Étape 2 — Remplacement progressif des lectures

Module par module, remplacer `getConfig().<featureName>` par `featureRegistry.get(GUILD_ID, '<featureName>')`.

| Module | Avant | Après | Risque |
|---|---|---|---|
| `feature_daily-message` | `config.daily_message` | `featureRegistry.get(guildId, 'daily_message')` | Faible (config inchangée) |
| `feature_welcome` | `config.welcome` | `featureRegistry.get(guildId, 'welcome')` | Faible |
| `feature_xp-level` | `config.xp` | `featureRegistry.get(guildId, 'xp')` | Faible |
| `game_road-to-infinite` | `config.counter` | `featureRegistry.get(guildId, 'counter')` | Faible |
| `game_count-down` | `config.countdown` | `featureRegistry.get(guildId, 'countdown')` | Faible |
| `service_bump-reminder` | `config.bump_reminder` | `featureRegistry.get(guildId, 'bump_reminder')` | Faible |
| `security_captcha` | `config.captcha` | `featureRegistry.get(guildId, 'captcha')` | Faible |

> **Rétrocompatibilité garantie** : tant que la DB n'a pas de ligne pour la feature, le YAML legacy est lu.

### Étape 3 — Outil de migration

Script `scripts/migrate-config-to-features.js` :

```js
// Pour chaque bloc legacy dans config.yml, créer une ligne dans feature_flags
const legacyFeatures = ['captcha', 'welcome', 'xp', 'counter', 'countdown', 'bump_reminder', 'daily_message'];

for (const name of legacyFeatures) {
  const yamlValue = config[name];
  if (!yamlValue) continue;

  const exists = await db.select().from(featureFlags)
    .where(and(eq(featureFlags.guildId, GUILD_ID), eq(featureFlags.featureName, name)))
    .limit(1);

  if (!exists[0]) {
    await db.insert(featureFlags).values({
      guildId: GUILD_ID,
      featureName: name,
      enabled: yamlValue.enabled ? 1 : 0,
      configJson: JSON.stringify(yamlValue),
      allowedRoles: JSON.stringify(yamlValue.allowed_roles || []),
      updatedBy: 'migration-script',
      updatedAt: Date.now()
    });
    console.log(`✅ ${name} migré vers feature_flags`);
  } else {
    console.log(`⏭️  ${name} déjà présent en DB, ignoré`);
  }
}
```

Exécution :

```bash
node scripts/migrate-config-to-features.js
```

> **Idempotent** : ne réécrit pas les lignes existantes. Le YAML reste la source de vérité tant que la migration n'est pas validée.

### Étape 4 — Validation

Après migration, **garder le YAML en lecture seule** comme fallback. Tester en parallèle :

```bash
# 1. Lancer le bot avec le YAML en place
npm start

# 2. Vérifier que toutes les features marchent comme avant
# (compteur, captcha, welcome, xp…)

# 3. Depuis le dashboard, toggle une feature
# → doit prendre effet immédiatement (cache invalidé)

# 4. Tuer la DB, redémarrer
# → le bot doit retomber sur le YAML (fallback)
```

### Étape 5 — Nettoyage (optionnel, après 1 mois de validation)

Une fois la migration validée :
- Supprimer la lecture legacy `getConfig()[name]`
- Documenter dans `config.example.yml` que les blocs `captcha:`, `welcome:`… sont obsolètes
- Section `features:` devient la seule valide

## 4. Exemple complet

### Avant

```yaml
# config.yml
captcha:
  enabled: true
  captcha_channel_name: "✅-verification-captcha"
  captcha_timeout: 10
```

### Pendant la migration

```yaml
# config.yml
captcha:
  enabled: true
  captcha_channel_name: "✅-verification-captcha"
  captcha_timeout: 10

features:
  captcha:
    # surcharge optionnelle, prend le dessus sur la racine
    captcha_timeout: 15
```

### Après migration (DB)

```sql
-- feature_flags
guild_id: "123456789"
feature_name: "captcha"
enabled: 1
config_json: '{"captcha_timeout":15,"captcha_channel_name":"✅-verification-captcha"}'
allowed_roles: '["MOD_ROLE_ID"]'
```

> Le `config.yml` peut désormais omettre le bloc `captcha:` (defaults appliqués).

## 5. Tests de non-régression

`tests/migration.test.js` :

```js
test('la lecture legacy fonctionne toujours', () => {
  process.env.CONFIG = './fixtures/config.legacy-only.yml';
  const config = loadConfig();
  expect(config.captcha.enabled).toBe(true);
});

test('la lecture features.* surcharge legacy', () => {
  process.env.CONFIG = './fixtures/config.features-only.yml';
  const config = loadConfig();
  expect(config.features.captcha.captcha_timeout).toBe(15);
});

test('la DB prend le dessus sur YAML', async () => {
  await db.insert(featureFlags).values({ ... });
  const state = await featureRegistry.get(GUILD_ID, 'captcha');
  expect(state.source).toBe('db');
});
```

## 6. Critères de succès de la migration

- [ ] Aucune régression visible côté utilisateurs pendant la transition
- [ ] Toggle depuis le dashboard prend effet sans restart
- [ ] Si la DB est down, le bot continue de fonctionner avec le YAML
- [ ] Script de migration idempotent
- [ ] Tests de non-régression verts
- [ ] Documentation utilisateur à jour
