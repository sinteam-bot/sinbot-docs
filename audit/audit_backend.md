# 🔍 Audit Complet du Backend

## Résumé Exécutif

Le backend est un bot Discord multifonction (Node.js / Express / Discord.js v14) avec un dashboard web, ~21 modules, un système d'injection de dépendances maison et une base PostgreSQL (avec fallback PGlite en dev). L'architecture est ambitieuse et bien structurée, mais plusieurs **vulnérabilités de sécurité critiques** et des problèmes de qualité nécessitent une attention immédiate.

---

## 🚨 1. SÉCURITÉ — Problèmes Critiques


### 1.1 ❌ Comparaison de clé API non sécurisée (Timing Attack)

> [!WARNING]
> **Sévérité : HAUTE** — [`src/index.js` L117](file://../../src/index.js#L117)

```javascript
// ❌ Vulnérable au timing attack
if (providedKey && expectedKey && providedKey === expectedKey) {
    return next();
}
```

La comparaison `===` de chaînes est vulnérable aux attaques temporelles. Un attaquant peut deviner la clé caractère par caractère en mesurant le temps de réponse.

**Fix :** Utiliser `crypto.timingSafeEqual()` :
```javascript
const crypto = require('crypto');
if (providedKey && expectedKey) {
    const a = Buffer.from(providedKey);
    const b = Buffer.from(expectedKey);
    if (a.length === b.length && crypto.timingSafeEqual(a, b)) {
        return next();
    }
}
```

Même problème dans [`/api/auth/verify`](file://../../src/index.js#L158) :
```javascript
const isValid = !authConfig.enabled || (apiKey && apiKey === authConfig.api_key);
```

---

### 1.2 ❌ API Key transmise en query string

> [!WARNING]
> **Sévérité : HAUTE** — [`src/index.js` L97](file://../../src/index.js#L97)

```javascript
let providedKey = req.headers['x-api-key'] || req.query.api_key || req.query.token;
```

Les paramètres de query string sont **loggés dans les access logs**, **visibles dans l'historique du navigateur**, et **stockés dans les proxys intermédiaires**. Cela expose la clé API.

**Fix :** Supprimer `req.query.api_key` et `req.query.token`, n'accepter que les headers.

---

### 1.3 ❌ Absence de Rate Limiting

> [!WARNING]
> **Sévérité : HAUTE** — Aucun rate limiting n'est implémenté sur :

- Les endpoints API (GET/POST/PATCH/DELETE)
- Le webhook `/webhook/send-message` (permet le spam de messages Discord)
- L'endpoint `/api/auth/verify` (brute-force de la clé API)
- Les endpoints de génération AI (`/api/daily-messages/generate*`) qui consomment des tokens

**Fix :** Ajouter `express-rate-limit` :
```javascript
const rateLimit = require('express-rate-limit');
app.use('/api/', rateLimit({ windowMs: 60000, max: 100 }));
app.use('/webhook/', rateLimit({ windowMs: 60000, max: 10 }));
app.use('/api/auth/verify', rateLimit({ windowMs: 900000, max: 5 }));
```

---

### 1.4 ❌ Pas de validation/sanitisation des entrées

> [!WARNING]
> **Sévérité : HAUTE**

| Endpoint | Paramètre non validé | Risque |
|---|---|---|
| `POST /webhook/send-message` | `channelId`, `message` | Envoi de messages arbitraires dans n'importe quel salon |
| `POST /api/channels/:channelId/messages` | `content` | Injection de contenu Discord (mentions @everyone) |
| `PATCH /api/channels/:channelId/messages/:messageId` | `content` | Modification arbitraire de messages |
| `DELETE /api/channels/:channelId/messages/:messageId` | Aucune restriction | Suppression de messages sans vérification d'ownership |
| `POST /api/config` | `module`, `config` | Écrasement de la configuration complète |
| `POST /api/channels/:channelId/posts` | `title`, `content` | Création de posts forum arbitraires |

Le webhook `/webhook/send-message` ne vérifie pas que le `channelId` correspond au serveur du bot — un attaquant pourrait envoyer des messages dans **n'importe quel serveur** accessible par le bot.

---

### 1.5 ⚠️ Injection SQL potentielle dans le fallback BDD

> [!WARNING]
> **Sévérité : MOYENNE** — [`webRouter.js` L767-769](file://../../src/web/webRouter.js#L767-L769)

```javascript
await db.pool.query(
    'UPDATE discord_messages SET content = ?, updated_at = CURRENT_TIMESTAMP WHERE message_id = ?',
    [editedContent, messageId]
);
```

Ce code utilise des placeholders `?` alors que le pool PostgreSQL utilise des placeholders `$1, $2`. L'adaptateur de compatibilité ([`database.js` L13-24](file://../../src/database.js#L13-L24)) convertit automatiquement, mais cette double couche de conversion est fragile et pourrait être contournée avec des payloads contenant des `?`.

---

### 1.6 ⚠️ WebSocket sans validation stricte

[`wsLogsServer.js` L19-20](file://../../src/utils/wsLogsServer.js#L19-L20) :

```javascript
const token = new URL(req.url, 'http://x').searchParams.get('api_key')
    || (req.headers['sec-websocket-protocol'] || '').split(',')[0].trim();
```

Le token est passé soit en query param (loggé), soit détourné via le header `sec-websocket-protocol` (hack). Pas de comparaison timing-safe non plus.

---

### 1.7 ⚠️ Headers de sécurité manquants

Aucun header de sécurité HTTP n'est défini :
- Pas de `helmet` middleware
- Pas de `Content-Security-Policy`
- Pas de `X-Content-Type-Options: nosniff`
- Pas de `X-Frame-Options`
- Pas de `Strict-Transport-Security`
- `Access-Control-Allow-Origin: *` est trop permissif sur le proxy d'images

---

### 1.8 ⚠️ `CORS: *` sur le proxy d'images

[`imageProxyService.js` L202](file://../../src/services/imageProxyService.js#L202) :
```javascript
res.setHeader('Access-Control-Allow-Origin', '*');
```

Bien que le proxy valide les domaines sources, le `*` permet à n'importe quel site d'utiliser ce proxy comme service d'images gratuit.

---

### 1.9 ⚠️ Exposition d'informations sensibles dans les erreurs

Tous les endpoints retournent `error.message` brut :
```javascript
res.status(500).json({ success: false, error: error.message });
```

En production, cela peut exposer des informations internes (chemins de fichiers, détails de connexion BDD, stack traces).

---

## 🏗️ 2. ARCHITECTURE — Analyse

### 2.1 ✅ Points Forts

| Aspect | Évaluation | Détail |
|---|:---:|---|
| **Module System** | ✅ Excellent | Architecture modulaire NestJS-like avec décorateurs, DI container, EventBus |
| **IoC Container** | ✅ Bon | [`container.js`](file://../../src/core/container.js) — Singleton, factory, résolution automatique |
| **EventBus** | ✅ Bon | [`event-bus.js`](file://../../src/core/event-bus.js) — Découplage des événements Discord |
| **Feature Registry** | ✅ Bon | [`feature-registry.js`](file://../../src/core/feature-registry.js) — Flags par guild avec persistance |
| **Resilience Policy** | ✅ Excellent | [`resiliencePolicy.js`](file://../../src/utils/resiliencePolicy.js) — Pattern Polly .NET avec retry/backoff/timeout |
| **Image Proxy** | ✅ Bon | [`imageProxyService.js`](file://../../src/services/imageProxyService.js) — Cache LRU, protection SSRF, ETag |
| **Config unifiée** | ✅ Bon | [`config/index.js`](file://../../src/config/index.js) — YAML + env overrides + fallback |
| **Test coverage** | ✅ Bon | 37 fichiers de test couvrant la majorité des services |
| **DB Abstraction** | ✅ Bon | Drizzle ORM + fallback PGlite pour tests |

### 2.2 ⚠️ Points d'Attention

| Aspect | Évaluation | Détail |
|---|:---:|---|
| **Fichier Monolithique** | ⚠️ | [`webRouter.js`](file://../../src/web/webRouter.js) = **2030 lignes, 91 Ko** — À splitter par domaine |
| **Fichier Monolithique** | ⚠️ | [`database.js`](file://../../src/database.js) = **2105 lignes, 70 Ko** — Idem |
| **`index.js` trop chargé** | ⚠️ | [`src/index.js`](file://../../src/index.js) mélange auth middleware, routes, Express setup et bot Discord |
| **`require()` inline** | ⚠️ | Nombreux `require()` à l'intérieur des handlers au lieu du top-level (ex: L1347, L1380...) |
| **Duplication** | ⚠️ | `database.js` (legacy) et `db/index.js` (Drizzle) coexistent avec du code dupliqué |
| **DDL inline** | ⚠️ | [`db/index.js`](file://../../src/db/index.js#L15-L856) contient **856 lignes de SQL DDL brut** — Devrait utiliser les migrations Drizzle |

---

## ⚙️ 3. FONCTIONNALITÉS — Inventaire Complet

### 3.1 Modules métier (21 modules)

| Module | Type | Description |
|---|---|---|
| `feature_automod` | Feature | Auto-modération (warnings, sanctions) |
| `feature_birthdays` | Feature | Gestion des anniversaires multi-guild |
| `feature_cards` | Feature | Cartes de bienvenue SVG |
| `feature_daily-message` | Feature | Message du jour IA (OpenRouter) avec brouillon/validation |
| `feature_economy` | Feature | Économie virtuelle (balance, banque, shop, inventaire, drops) |
| `feature_engagement` | Feature | Engagement utilisateurs |
| `feature_engagement-advanced` | Feature | Engagement avancé (giveaways, polls) |
| `feature_info` | Feature | Informations serveur/utilisateur |
| `feature_logs` | Feature | Logs Discord en temps réel (SSE + WebSocket) |
| `feature_reaction-roles` | Feature | Rôles par réaction/boutons |
| `feature_reports` | Feature | Signalements utilisateurs |
| `feature_sticky-roles` | Feature | Persistance des rôles |
| `feature_tickets` | Feature | Système de tickets support |
| `feature_welcome` | Feature | Accueil nouveaux membres |
| `feature_xp-level` | Feature | Système XP/niveaux avec leaderboard |
| `game_count-down` | Game | Jeu countdown |
| `game_road-to-infinite` | Game | Jeu compteur infini |
| `security_captcha` | Security | Captcha visuel |
| `security_question` | Security | Question de sécurité |
| `service_bump-reminder` | Service | Rappels bump Disboard |
| `notifier_startup` | Notifier | Notification de démarrage |

### 3.2 API REST (Dashboard Web)

| Endpoint | Méthode | Fonction |
|---|---|---|
| `/api/guild` | GET | Informations serveur |
| `/api/channels` | GET | Liste des salons avec catégories |
| `/api/channels/:id/messages` | GET/POST | Messages (pagination infinie) + envoi |
| `/api/channels/:id/messages/:id` | PATCH/DELETE | Modifier/supprimer messages |
| `/api/channels/:id/threads` | GET | Fils de discussion |
| `/api/channels/:id/posts` | GET/POST | Posts forum |
| `/api/users` | GET | Liste membres (filtres, tri, pagination) |
| `/api/roles` | GET | Rôles serveur |
| `/api/logs` | GET/DELETE | Logs bot |
| `/api/logs/stream` | GET (SSE) | Flux logs en direct |
| `/api/config` | GET/POST | Configuration modules |
| `/api/modules/status` | GET | État des modules |
| `/api/daily-messages/*` | GET/POST | Gestion messages IA |
| `/api/openrouter/*` | GET/POST | Config OpenRouter |
| `/api/captcha-logs` | GET | Historique captcha |
| `/api/bump/*` | GET/POST | Service bump |
| `/api/games/*` | GET | Stats jeux |
| `/api/commands` | GET | Liste commandes |
| `/api/commands/sync` | POST | Sync commandes Discord |
| `/api/events/archive` | GET | Archive événements |
| `/api/template/render` | POST | Rendu templates |
| `/api/template/presets` | GET | Presets templates |
| `/api/features` | GET/PATCH | Feature flags |
| `/api/proxy/*` | GET | Proxy images Discord |
| `/api/cache/sync` | POST | Sync cache Discord |
| `/api/emojis` | GET | Emojis serveur |
| `/api/auth/status` | GET | État auth |
| `/api/auth/verify` | POST | Vérification auth |
| `/webhook/send-message` | POST | Webhook n8n |
| `/health` | GET | Health check |
| `/ws/logs` | WebSocket | Live feed logs |

### 3.3 Infrastructure

| Composant | Technologie | Observations |
|---|---|---|
| Runtime | Node.js 24 | ✅ Moderne |
| Framework API | Express 5.2 | ✅ Dernière version |
| Bot Discord | discord.js v14.27 | ✅ À jour |
| ORM | Drizzle ORM 0.45 | ✅ Bon choix |
| BDD Production | PostgreSQL 16 (pg 8.23) | ✅ |
| BDD Dev/Test | PGlite (WASM in-memory) | ✅ Excellent pour tests |
| IA | OpenRouter (SDK OpenAI 7.8) | ✅ |
| Scheduler | node-cron 4.6 | ✅ |
| Tests | Vitest 3.2 + coverage v8 | ✅ |
| Container | Docker multi-stage Alpine | ✅ |
| Frontend | Nuxt.js (SSG) | ✅ |

---

## 📊 4. QUALITÉ DE CODE

### 4.1 Bonnes pratiques appliquées ✅

- Gestion d'erreurs globale (`unhandledRejection`, `client.error`)
- Pattern Builder pour la configuration (ResiliencePolicy)
- Fallback DB gracieux (Discord live → cache BDD)
- Logs structurés avec catégories
- Tests unitaires étendus (37 fichiers)
- Docker multi-stage optimisé
- Vérification d'ownership pour l'édition de messages bot (L753)
- Protection SSRF dans le proxy d'images
- Sauvegarde atomique de config (write tmp → rename)

### 4.2 Problèmes de qualité ⚠️

| Problème | Fichier | Impact |
|---|---|---|
| `webRouter.js` = 2030 lignes | [`webRouter.js`](file://../../src/web/webRouter.js) | Maintenabilité très difficile |
| `database.js` = 2105 lignes | [`database.js`](file://../../src/database.js) | Idem |
| `db/index.js` = 1070 lignes de DDL | [`db/index.js`](file://../../src/db/index.js) | Pas de migrations versionnées |
| `catch (e) {}` silencieux | Multiple (L135, L303, L905...) | Erreurs avalées sans trace |
| Mixage `?` et `$N` placeholders | [`webRouter.js` L768](file://../../src/web/webRouter.js#L768) | Incohérence ORM vs raw SQL |
| `better-sqlite3` encore en dépendance | [`package.json` L23](file://../../package.json#L23) | Dead dependency (migration vers PG terminée) |
| Commentaire "SQLite" dans le code PG | [`webRouter.js` L765](file://../../src/web/webRouter.js#L765-L766) | Commentaire trompeur |
| `NODE_ENV=production` dans `.env` | [`.env` L3](file://../../.env#L3) | Mais `config.yml` dit `test` (L13) — Incohérence |
| `client.login(process.env.DISCORD_TOKEN)` | [`index.js` L334](file://../../src/index.js#L334) | Pourrait utiliser `config.discord.token` pour cohérence |
| Docker installe `python3 make g++` puis les supprime | [`Dockerfile` L28-37](file://../../Dockerfile#L28-L37) | Pour `better-sqlite3` qui n'est plus nécessaire |

---

## 🔧 5. RECOMMANDATIONS PRIORITAIRES

### 🔴 Priorité Critique (immédiat)

1. **Utiliser `crypto.timingSafeEqual()`** pour la comparaison des clés API
2. **Supprimer les clés API des query strings** — headers uniquement
3. **Ajouter un rate limiter** (`express-rate-limit`) sur tous les endpoints

### 🟠 Priorité Haute (cette semaine)

5. **Ajouter `helmet`** pour les headers de sécurité
6. **Valider les entrées** — `channelId` doit être un snowflake Discord, `content` limité en taille
7. **Restreindre le webhook** — vérifier que le channel appartient au guild configuré
8. **Masquer les erreurs en production** — retourner des messages génériques
9. **Sécuriser le CORS** — remplacer `*` par l'origin du dashboard

### 🟡 Priorité Moyenne (ce mois)

10. **Splitter `webRouter.js`** en sous-routeurs par domaine (messages, users, config, games, daily...)
11. **Splitter `database.js`** en repositories par module (utiliser le pattern existant du module system)
12. **Migrer le DDL** vers des fichiers de migration Drizzle versionnés
13. **Retirer `better-sqlite3`** du `package.json` et simplifier le Dockerfile
14. **Éliminer les `catch (e) {}`** silencieux — au minimum logger un warning
15. **Harmoniser les `require()` inline** — les déplacer en haut de fichier

### 🟢 Priorité Basse (backlog)

16. Ajouter des tests d'intégration pour les routes API
17. Implémenter un système de permissions granulaire sur l'API (RBAC)
18. Ajouter un audit log des actions effectuées via l'API web
19. Ajouter la validation de schéma (Zod/Joi) pour les `req.body`
20. Monitorer la taille du cache LRU de l'image proxy en production

---

## 📈 Score Global

| Catégorie | Note | Détail |
|---|:---:|---|
| 🔒 **Sécurité** | **4/10** | pas de rate limiting, timing attacks, pas de validation |
| 🏗️ **Architecture** | **7/10** | Excellent module system, mais fichiers monolithiques |
| ⚙️ **Fonctionnalités** | **9/10** | Très riche — 21 modules, API complète, dashboard, IA, games |
| 📊 **Qualité de Code** | **6/10** | Bons patterns mais dette technique accumulée |
| 🧪 **Tests** | **7/10** | 37 fichiers — bon coverage unitaire, manque l'intégration |
| 🐳 **DevOps** | **7/10** | Docker multi-stage, mais dépendances mortes et incohérences env |
| **Score Global** | **6.5/10** | Fondations solides, sécurité à renforcer en priorité |
