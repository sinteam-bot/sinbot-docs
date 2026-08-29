# Guide — Stratégie de tests

> Le projet utilise `node --test` (runner natif de Node 20+). Pas de Jest, pas de Mocha.

## 1. Lancer les tests

```bash
# Tous les tests
npm test

# Un fichier
node --test tests/spam-detector.test.js

# Avec couverture
node --test --experimental-test-coverage tests/**/*.test.js

# En mode watch (dev)
node --test --watch tests/
```

## 2. Structure

```
tests/
├── spam-detector.test.js
├── raid-detector.test.js
├── sanctions.test.js
├── bad-words.test.js
├── feature-registry.test.js
└── migration.test.js
```

Les tests sont **co-localisés** avec le code source dans certains cas (cf. `src/modules/security_automod/tests/`).

## 3. Anatomie d'un test

```js
const { test, describe, before, after, beforeEach } = require('node:test');
const assert = require('node:assert');

describe('SpamDetector', () => {
  let detector;

  beforeEach(() => {
    detector = new SpamDetector();
  });

  test('détecte 5 messages en 5s', () => {
    const guildId = 'g1';
    const userId = 'u1';

    for (let i = 0; i < 4; i++) {
      const r = detector.check(guildId, userId, { content: 'spam' });
      assert.strictEqual(r.isSpam, false);
    }
    const r = detector.check(guildId, userId, { content: 'spam' });
    assert.strictEqual(r.isSpam, true);
  });

  test('ignore les bots', () => {
    // ...
  });
});
```

## 4. Mocks

### 4.1 Mock d'un service

```js
const mockRemindersRepo = {
  create: async (data) => ({ id: 'fake-id', ...data }),
  list: async () => [],
  remove: async () => true
};

const service = new RemindersService(mockRemindersRepo);
```

### 4.2 Mock du Container

```js
const { container } = require('../src/core/container');
container.register('RemindersRepo', { create: async () => ({ id: 'x' }) });
```

### 4.3 Mock d'interaction Discord

```js
function mockInteraction(opts = {}) {
  return {
    guildId: 'g1',
    user: { id: 'u1' },
    options: {
      getString: (name) => opts[name] || null,
      getUser: (name) => opts[name] || null
    },
    reply: async (msg) => { this.lastReply = msg; },
    lastReply: null
  };
}
```

## 5. Tests d'intégration (DB)

```js
const { test } = require('node:test');
const assert = require('node:assert');
const { db } = require('../src/db');
const { featureFlags } = require('../src/database/schema/feature_flags');
const { featureRegistry } = require('../src/core/feature-registry');

test('featureRegistry.set persiste en DB', async () => {
  const guildId = `test-${Date.now()}`;  // unique pour éviter les collisions
  await featureRegistry.set(guildId, 'automod', {
    enabled: true,
    config: { spam: { max_messages: 10 } },
    allowedRoles: ['role1'],
    updatedBy: 'test'
  });

  const state = await featureRegistry.get(guildId, 'automod');
  assert.strictEqual(state.enabled, true);
  assert.strictEqual(state.config.spam.max_messages, 10);
  assert.deepStrictEqual(state.allowedRoles, ['role1']);

  // Cleanup
  await db.delete(featureFlags).where(eq(featureFlags.guildId, guildId));
});
```

## 6. Tests d'API (Express)

```js
const { test } = require('node:test');
const assert = require('node:assert');
const request = require('supertest');
const { app } = require('../src/web/app');

test('GET /api/features retourne la liste', async () => {
  const res = await request(app)
    .get('/api/features')
    .set('x-api-key', 'test-key');

  assert.strictEqual(res.status, 200);
  assert.ok(Array.isArray(res.body));
});
```

> `supertest` n'est pas encore installé. À ajouter dans `package.json` pour les tests HTTP.

## 7. Bonnes pratiques

- **Un test = un comportement** (pas de test fourre-tout)
- **Nommage clair** : `test('détecte 5 messages en 5s', ...)` plutôt que `test('cas 1', ...)`
- **AAA** : Arrange / Act / Assert
- **Pas de dépendances entre tests** : utiliser `beforeEach` pour réinitialiser
- **Cleanup** : supprimer les données créées en BDD en fin de test
- **Pas de magic strings** : utiliser des constantes ou des factories
- **Tests rapides** : < 100ms par test unitaire, < 1s pour l'intégration
- **Tests parallélisables** : `node --test --test-concurrency=4`

## 8. CI

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test
      - run: npm run lint
```

## 9. Roadmap tests

- [ ] Tests unitaires pour tous les nouveaux services (Automod Phase 1)
- [ ] Tests d'intégration DB
- [ ] Tests HTTP via supertest
- [ ] Tests E2E Discord (via mock client)
- [ ] Coverage > 70% sur les services critiques
- [ ] CI GitHub Actions
