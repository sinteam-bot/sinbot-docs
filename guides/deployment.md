# Guide — Déploiement

## 1. Environnements

| Env | DB | Build | URL |
|---|---|---|---|
| **Dev local** | SQLite (`./data/bot.db`) | `npm run dev` | localhost:3000 |
| **CI** | SQLite in-memory | `npm test` | — |
| **Prod** | PostgreSQL 16 | Docker Compose | https://bot.example.com |

## 2. Variables d'environnement

`.env` (jamais commité) :

```bash
# Discord
DISCORD_TOKEN=...
DISCORD_CLIENT_ID=...
DISCORD_GUILD_ID=...

# Web
PORT=3000
NODE_ENV=production
API_KEY=changez_cette_cle_secrete

# Database
DATABASE_URL=postgresql://user:pass@db:5432/bot
# ou pour SQLite :
DB_PATH=./data/bot.db

# OpenRouter
OPENROUTER_API_KEY=sk-or-v1-...

# Optionnel
STARTUP_NOTIFIER_GITHUB_TOKEN=...
LOG_LEVEL=info
```

## 3. Docker Compose (prod)

`docker-compose.yml` (extrait) :

```yaml
version: '3.8'
services:
  bot:
    build: .
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./config.yml:/app/config.yml:ro
      - ./data:/app/data
      - ./logs:/app/logs
    ports:
      - "3000:3000"
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: bot
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: bot
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./frontend/.output/public:/usr/share/nginx/html:ro
    ports:
      - "80:80"
      - "443:443"

volumes:
  pgdata:
```

## 4. Dockerfile

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build:ui

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app /app
ENV NODE_ENV=production
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:3000/healthz || exit 1
CMD ["node", "src/index.js"]
```

## 5. Migrations DB

```bash
# Générer une migration
npx drizzle-kit generate

# Appliquer (SQLite)
node scripts/migrate.js

# Appliquer (PostgreSQL)
node scripts/migrate.js --driver=pg

# Migration SQLite → PostgreSQL (one-shot)
node scripts/migrate-sqlite-to-postgres.js
```

## 6. Healthcheck

`src/web/routes/health.js` :

```js
const { Get, Controller } = require('../../core');
const { db } = require('../../db');

@Controller('/healthz')
class HealthController {
  @Get('/')
  async check(req, res) {
    const dbOk = await db.run('SELECT 1').then(() => true).catch(() => false);
    return { status: 'ok', db: dbOk, uptime: process.uptime() };
  }
}
```

## 7. Logs

- stdout/stderr capturés par Docker
- Rotation via `logrotate` ou `pm2`
- Format JSON recommandé pour l'agrégation (Loki, ELK)

```json
{"ts":1693142400000,"level":"info","scope":"[ModuleManager]","msg":"Module chargé : AutoModModule"}
```

## 8. Backups

### PostgreSQL

```bash
# Cron quotidien
0 3 * * * pg_dump -U bot bot | gzip > /backups/bot-$(date +\%F).sql.gz
```

### SQLite

```bash
0 3 * * * sqlite3 /app/data/bot.db ".backup '/backups/bot-$(date +\%F).db'"
```

## 9. Monitoring (recommandé)

- **Métriques** : `prom-client` exposé sur `/metrics`
- **Uptime** : UptimeRobot / BetterStack
- **Erreurs** : Sentry (`@sentry/node`)
- **Alertes Discord** : webhook vers un salon admin

## 10. Mise à jour

```bash
# 1. Pull
git pull origin main

# 2. Install
npm ci

# 3. Migrations
node scripts/migrate.js

# 4. Re-deploy commands Discord
npm run deploy

# 5. Restart
docker compose restart bot
```

## 11. Rollback

```bash
# 1. Revenir au commit précédent
git checkout HEAD~1

# 2. Re-build
docker compose up -d --build bot

# 3. Rollback DB si nécessaire
psql -U bot -d bot -f /backups/bot-2026-08-26.sql
```

## 12. Checklist pré-déploiement

- [ ] `npm test` passe
- [ ] `npm run lint` passe
- [ ] `npm run build:ui` réussit
- [ ] Variables d'env à jour sur le serveur
- [ ] `config.yml` sauvegardé
- [ ] DB backup effectué
- [ ] Healthcheck répond `200 OK`
- [ ] Slash commands redéployées
- [ ] Au moins 1 message test posté pour valider
