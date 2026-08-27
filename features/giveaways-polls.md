# Feature : Giveaways & Sondages

> **Phase** : 5 — **Priorité** : 🟢 Basse — **Statut** : à implémenter

## 1. Objectif

Deux features de pur engagement communautaire, inspirées de Draftbot.

## 2. Giveaways

### 2.1 Cas d'usage

- Un staff lance un giveaway avec `/giveaway start`
- Les participants cliquent sur 🎉
- À la fin, N gagnants sont tirés au sort
- Possibilité de relancer le tirage (`/giveaway reroll`)

### 2.2 Configuration

```yaml
features:
  giveaways:
    enabled: false
    allowed_roles: ["STAFF_ROLE_ID"]
    defaults:
      color: "#5865F2"
      emoji: "🎉"
      dm_winners: true
      ping_role: null
    constraints:
      max_duration_days: 30
      min_duration_minutes: 5
      max_winners: 20
      max_prize_length: 256
```

### 2.3 Slash commands

| Commande | Description |
|---|---|
| `/giveaway start` | Modal : prix, gagnants, durée, salon |
| `/giveaway end <id>` | Terminer immédiatement |
| `/giveaway reroll <id>` | Retirer N nouveaux gagnants |
| `/giveaway list` | Lister les giveaways actifs |
| `/giveaway cancel <id>` | Annuler |

### 2.4 Embed de giveaway

```
╔══════════════════════════════════╗
║  🎉 GIVEAWAY : Nitro Classic    ║
╠══════════════════════════════════╣
║ Réagis avec 🎉 pour participer ! ║
║                                  ║
║ ⏰ Fin : 27 août 2026 18:00     ║
║ 👥 Participants : 142            ║
║ 🏆 Gagnants : 1                  ║
║                                  ║
║ Lancé par : @Staff               ║
╚══════════════════════════════════╝
```

Boutons : `🎉 Participer` | `📊 Participants` (ephemeral)

### 2.5 Tirage

- Tâche cron vérifie toutes les minutes les giveaways `ends_at <= now AND status = 'active'`
- Tirage via `crypto.randomInt` (CSPRNG)
- Si moins de participants que de gagnants : on tire ceux qui sont là
- Éditer le message pour afficher les gagnants
- DM les gagnants (si `dm_winners: true`)
- Enregistrer dans `giveaways.winners` (JSON array)

### 2.6 Architecture

```
src/modules/feature_giveaways/
├── giveaways.module.js
├── config/schema.js
├── db/schema.js
├── services/
│   ├── giveaway.service.js
│   ├── giveaway-draw.service.js
│   └── embed-builder.service.js
├── events/
│   ├── interaction-create.listener.js # bouton "Participer"
│   └── message-delete.listener.js     # cancel si le bot supprime
├── commands/
│   ├── giveaway-start.command.js
│   ├── giveaway-end.command.js
│   ├── giveaway-reroll.command.js
│   ├── giveaway-list.command.js
│   └── giveaway-cancel.command.js
├── cron/
│   └── giveaway-draw.cron.js          # toutes les minutes
├── controllers/
│   └── giveaways.controller.js
└── tests/
    ├── draw.test.js
    └── participation.test.js
```

### 2.7 BDD

Voir [`../architecture/data-model.md`](../architecture/data-model.md#34-giveaways-phase-5) — tables `giveaways` et `giveaway_entries`.

## 3. Sondages (Polls)

### 3.1 Cas d'usage

- Un membre crée un sondage avec `/poll create`
- Les autres votent via boutons
- Résultats affichés en live (mis à jour à chaque vote ou toutes les X secondes)

### 3.2 Configuration

```yaml
features:
  polls:
    enabled: false
    allowed_roles: []
    defaults:
      max_options: 10
      min_options: 2
      max_duration_days: 7
      allow_multi_choice: true
      anonymous: false
      show_results_before_end: false
      color: "#5865F2"
```

### 3.3 Slash commands

| Commande | Description |
|---|---|
| `/poll create` | Modal : question, options (jusqu'à 10), durée |
| `/poll end <id>` | Terminer et figer les résultats |
| `/poll delete <id>` | Supprimer |
| `/poll list` | Lister les sondages actifs |

### 3.4 Embed de sondage

```
╔══════════════════════════════════╗
║  📊 Sondage : Pizza ou Sushi ?   ║
╠══════════════════════════════════╣
║ 🍕 Pizza      ████░░░░░░  45% (45) ║
║ 🍣 Sushi      ██████░░░░  55% (55) ║
║                                  ║
║ Total : 100 votes                ║
║ ⏰ Fin : dans 2j 4h              ║
╚══════════════════════════════════╝
```

### 3.5 Architecture

```
src/modules/feature_polls/
├── polls.module.js
├── config/schema.js
├── db/schema.js
├── services/
│   ├── poll.service.js
│   ├── vote.service.js
│   └── results-builder.service.js
├── events/
│   ├── interaction-create.listener.js
│   └── message-delete.listener.js
├── commands/
│   ├── poll-create.command.js
│   ├── poll-end.command.js
│   ├── poll-delete.command.js
│   └── poll-list.command.js
├── cron/
│   └── poll-close.cron.js
├── controllers/
│   └── polls.controller.js
└── tests/
    ├── vote.test.js
    └── results.test.js
```

### 3.6 BDD

Voir [`../architecture/data-model.md`](../architecture/data-model.md#35-polls-phase-5) — tables `polls` et `poll_votes`.

### 3.7 Mise à jour des résultats

- **Option A (simple)** : éditer le message à chaque clic → peut hit les rate limits
- **Option B (recommandée)** : cache en mémoire pendant 5s, refresh au-delà
- **Option C** : mettre à jour toutes les 10s via un cron si le poll est récent

## 4. UI (Nuxt)

- Page `/features/giveaways` : config, giveaways actifs
- Page `/features/polls` : config, sondages en cours
- Composant `PollLive.vue` (intégrable dans une page) : sondages actifs avec vote

## 5. Critères d'acceptation

### Giveaways
- [ ] Création via modal fonctionnel
- [ ] Tirage automatique à l'heure prévue
- [ ] Tirage aléatoire réellement aléatoire (`crypto.randomInt`)
- [ ] DM des gagnants
- [ ] Reroll fonctionne
- [ ] Contraintes de durée / nombre de gagnants respectées

### Polls
- [ ] Création multi-options
- [ ] Vote unique ou multi selon config
- [ ] Résultats affichés en live
- [ ] Auto-close à la fin
- [ ] Pas de rate limit hit (cache ou throttle)
