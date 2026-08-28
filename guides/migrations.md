# Guide — Migrations de base de données

> **Workflow** : `drizzle-kit` (déjà installé en devDep) gère les migrations versionnées de la base PostgreSQL (et PGlite en dev/test).

## Commandes

| Commande | Usage |
|---|---|
| `npm run db:generate` | Génère une migration SQL à partir des schémas Drizzle (à commit). |
| `npm run db:migrate` | Applique les migrations en attente sur la base configurée. |
| `npm run db:push` | Pousse le schema directement (dev uniquement, **jamais en prod**). |
| `npm run db:studio` | Ouvre Drizzle Studio (UI web d'inspection de la base). |

## Configuration

Le fichier `drizzle.config.js` à la racine :
- **Production** : lit `DATABASE_URL` (ou `DB_URL`) ou les variables `PG_HOST/PORT/USER/PASSWORD/DATABASE`.
- **Test** (`NODE_ENV=test`) : driver PGlite mémoire.
- **Dev** : PostgreSQL local via `PG_HOST` par défaut.

## Workflow quotidien

```bash
# 1. Modifier src/modules/<x>/db/schema.js (ou src/db/schemas/shared/<x>.js)
# 2. Générer la migration
npm run db:generate
# → produit src/db/migrations/0001_xxx.sql + maj meta/_journal.json

# 3. TOUJOURS relire le SQL généré (drizzle-kit peut faire des choix surprenants)
# 4. Ajouter un commentaire de rollback en tête du fichier si besoin :
#    -- @down: <SQL inverse>
# 5. Tester en local
npm run db:migrate

# 6. Commit schema + migration ensemble
git add src/modules/<x>/db/schema.js src/db/migrations/
git commit -m "feat(<x>): add <table> for <feature>"
```

## Politique

1. **Jamais l'un sans l'autre** : un changement de schema est toujours commit avec sa migration.
2. **Pas de migration destructrice** sans backfill documenté.
3. **Renommage de colonne** = 2 migrations (add new → backfill → drop old).
4. **CI** : `npm run db:generate --check` doit passer (cf. `package.json`).
5. **Production** : `db:migrate` est exécuté **avant** `node src/index.js` au démarrage du conteneur.

## Rollback

`drizzle-kit` n'a pas de `migrate down` natif. Convention :
- Tout fichier `*.sql` commence par un commentaire `-- @down: ...` décrivant l'inverse.
- Rollback réel = `git revert` du commit + nouvelle migration corrective.

## Tables transverses

Les tables partagées par plusieurs modules vivent dans `src/db/schemas/shared/` :
- `audit.js` — mod_logs, event_log, auth_audit_logs, user_events, form_responses
- `cache.js` — guilds, channels, roles, members, messages, emojis, threads, guild_stats
- `feature-flags.js` — feature_flags
- `guild-settings.js` — guild_settings

Les tables propres à un module vivent dans `src/modules/<x>/db/schema.js`.
