# Backlog — Features DraftBot à implémenter

> **Date** : 2026-08-27
>
> Ce document est un **backlog actionnable** : pour chaque feature, on trouve
> - La priorité (P0 / P1 / P2 / P3)
> - L'effort estimé
> - Le nombre de tables / endpoints / commandes à créer
> - Le lien vers le document d'audit détaillé
>
> **Note** : P0 = bloquant / valeur immédiate, P3 = nice-to-have.

## Backlog priorisé

### P0 — Critique (à planifier ASAP)

| ID | Feature | Phase | Effort | Tables | Endpoints | Commands | Justification |
|---|---|:---:|:---:|---:|---:|---:|---|
| F-01 | Économie (monnaie) | 9 | 🔴 | 2 | 6 | 6 | Fondationnel : débloque inventaire, gifts d'anniv, drops |
| F-02 | Objets & inventaire | 9 | 🔴 | 4 | 14+ | 15+ | Engagement fort, dépendance de F-01 |
| F-03 | Rôles-Réactions | 10 | 🟡 | 1 | 5 | 3 | Très demandé côté communauté |

### P1 — Haute valeur (engagement)

| ID | Feature | Phase | Effort | Tables | Endpoints | Commands | Justification |
|---|---|:---:|:---:|---:|---:|---:|---|
| F-04 | Réactions de mots (auto-réponses) | 11 | 🟡 | 1 | 5 | 4 | Simple à mettre, gros usage fun |
| F-05 | Commandes d'informations | 8 | 🟢 | 0 | 3 | 3 | Effort très faible, gain UX immédiat |
| F-06 | Rappels | 11 | 🟡 | 1 | 5 | 3 | Usage quotidien staff + users |
| F-07 | Commandes personnalisées | 11 | 🟡 | 1 | 5 | 4 | Très demandé, permet de customiser le bot sans dev |

### P2 — Valeur modérée

| ID | Feature | Phase | Effort | Tables | Endpoints | Commands | Justification |
|---|---|:---:|:---:|---:|---:|---:|---|
| F-08 | Starboards | 12 | 🟡 | 1 | 4 | 4 | Engagement communautaire |
| F-09 | Signalements | 12 | 🟡 | 2 | 5 | 3 | Modération communautaire (sécurité) |
| F-10 | Sticky roles | 8 | 🟢 | 1 | 3 | 3 | Effort très faible |
| F-11 | Stats de jeux (dashboard) | 8 | 🟢 | 0 | 3 | 0 | Branche les données existantes au dashboard |
| F-12 | Salons vocaux temporaires | 12 | 🟡 | 1 | 3 | 3 | Engagement vocal |

### P3 — Nice-to-have (long terme)

| ID | Feature | Phase | Effort | Tables | Endpoints | Commands | Justification |
|---|---|:---:|:---:|---:|---:|---:|---|
| F-13 | Calendrier de l'Avent | 13 | 🟡 | 1 | 3 | 3 | Saisonnier, ~1 mois/an |
| F-14 | Bingo | 13 | 🟡 | 2 | 4 | 6 | Fun mais niche |
| F-15 | Commandes fun (`/roll`, `/8ball`, etc.) | 13 | 🟢 | 1 | 1 | 5 | Trivial à implémenter |
| F-16 | Messages channel-bound | 12 | 🟡 | 1 | 4 | 3 | Extension du `DailyMessageModule` |
| F-17 | Messages récurrents | 11 | 🟡 | 1 | 5 | 3 | Planification cron-like |
| F-18 | Sauvegardes serveur | 12 | 🔴 | 0 | 3 | 3 | Sensible (rate limits), effort élevé |
| F-19 | Interserveurs | 14 | 🔴 | 3+ | 10+ | 5+ | Premium-only Draftbot, GDPR, opt-in |

## Estimation globale

| Priorité | Features | Effort cumulé | Semaines estimées |
|---|---:|---:|---:|
| P0 | 3 | 🔴🔴🔴 | ≈ 5-6 |
| P1 | 4 | 🟡🟡🟡🟡 | ≈ 3-4 |
| P2 | 5 | 🟢🟢🟡🟡🟡 | ≈ 3 |
| P3 | 7 | 🟢🟡🟡🟡🟡🔴🔴 | ≈ 4-5 |
| **Total** | **19** | — | **≈ 15-18 semaines** |

## Plan d'attaque recommandé

### Sprint 1 (1 semaine) — Quick wins
- F-05 `/serverinfo`, `/userinfo`, `/avatar` (3 commandes)
- F-10 Sticky roles
- F-11 Stats de jeux dashboard

**Valeur** : UX immédiate, effort minimal, débloque 3 features visibles.

### Sprint 2 (3-4 semaines) — Économie & Inventaire
- F-01 Économie (tables + service + 6 commands)
- F-02 Inventaire (tables + 4 services + 15+ commands)

**Valeur** : moteur économique du serveur. Débloque :
- Cadeau "argent" dans les anniversaires (remplace le placeholder)
- Cadeau "objet inventaire" dans les anniversaires
- Drops d'objets dans les giveaways (extension)
- `/shop` et `/daily` côté user (engagement quotidien)

### Sprint 3 (1 semaine) — Réaction Roles
- F-03 Rôles-Réactions (table + service + 3 commands)

**Valeur** : très demandé, simple à implémenter.

### Sprint 4 (2 semaines) — Engagement avancé
- F-04 Réactions de mots
- F-06 Rappels
- F-07 Commandes personnalisées

**Valeur** : 3 features orientées "fun + custom".

### Sprint 5 (2 semaines) — Modération communautaire
- F-08 Starboards
- F-09 Signalements
- F-12 Salons vocaux temporaires

**Valeur** : engagement communautaire + sécurité.

### Sprint 6 (1 semaine) — Jeux additionnels
- F-15 Commandes fun (`/roll`, `/coinflip`, `/8ball`, etc.)

**Valeur** : fun, effort minimal.

### Sprint 7+ — Le reste
- F-13, F-14, F-16, F-17, F-18, F-19 selon la demande

## Métriques de succès

À chaque fin de feature, mesurer :
- **Adoption** : nombre de serveurs qui activent la feature
- **Usage** : nombre de commands exécutées / jour
- **Satisfaction** : NPS ou feedback qualitatif
- **Performance** : latence P95 des endpoints
- **Stabilité** : taux d'erreur, MTBF

## Décisions à prendre

Avant de commencer Phase 9, on doit clarifier :
1. **Mode multi-guild** : on garde le mode actuel (chacun sa config) ou on supporte le partage d'économie cross-guild ?
2. **Inventaire cross-server** : un user garde-t-il ses objets en changeant de serveur ? (Draftbot = non)
3. **Taxe de transaction** : `/pay` prend-il une commission ? (Draftbot = non)
4. **Modération anti-abuse** : limite-t-on le `/daily` à 1 fois / 24h par user ? (Draftbot = oui)
5. **RGPD** : combien de temps garde-t-on l'historique des transactions ?

## Voir aussi

- [`audit-draftbot.md`](./audit-draftbot.md) — analyse stratégique globale
- [`draftbot-feature-list.md`](./draftbot-feature-list.md) — table exhaustive avec liens
- [`migration-impact.md`](./migration-impact.md) — impact DB / API par feature
