# Bots spécialisées — Features complémentaires

> **Date** : 2026-08-29
> **Sources** : Confessions Bot, Needle, Tickets Bot, Ticket Tool, ModMail, Statbot, Welcomer, MsgPlanner
>
> Ce document liste les features uniques des bots spécialisées et les compare avec l'existant du Bot.

---

## 1. Confessions (confessions.bot / confessy.xyz)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Confessions anonymes (`/confess`) | Messages anonymes dans un canal dédié | ❌ | `community_confessions/` |
| Réponses anonymes (`/reply`) | Répondre anonymement à une confession | ❌ | `community_confessions/` |
| Sondages anonymes | Créer des sondages anonymes | ❌ | `community_confessions/` |
| Mode revue (review) | Staff approuve/rejette avant publication | 🟡 | `community_confessions/` (partial: reports existants) |
| Bannissement (`/confessban`) | Interdire un user de confesser | ❌ | `community_confessions/` |
| Filtres de mots | Blocker mots inappropriés dans confessions | ❌ | `community_confessions/` |
| Signalements (`/report`) | Signaler une confession abusive | 🟡 | `community_reports/` (existe mais pas pour confessions) |
| Canaux multiples | Plusieurs canaux de confession (premium) | ❌ | `community_confessions/` |
| Couleurs embed custom | Personnaliser couleur des embeds | 🟡 | `welcome_cards/` (existe partiellement) |

**Valeur ajoutée** : Espace d'expression anonyme sécurisé pour les membres. Feature communautaire très populaire.

---

## 2. Needle (needle.gg)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Auto-thread automatique | Crée un thread par message dans canaux configurés | ❌ | `automation_autothread/` |
| Titre configurable | Variables de message, regex pour titre | ❌ | `automation_autothread/` |
| Message custom | Message personnalisé dans le thread | ❌ | `automation_autothread/` |
| Changement de titre (`/title`) | Mod titre sans permissions spéciales | ❌ | `automation_autothread/` |
| Fermeture (`/close`) | Archiver le thread | ❌ | `automation_autothread/` |
| Slowmode | Rate limit dans les threads créés | ❌ | `automation_autothread/` |
| Pin message | Pin auto du premier message | ❌ | `automation_autothread/` |
| Emoji thread sans reply | Indicateur 🆕 pour threads sans réponse | ❌ | `automation_autothread/` |

**Valeur ajoutée** : Réduit le bruit dans les salons d'images/discussions. Organise automatiquement les conversations.

---

## 3. Tickets Bot (tickets.bot)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Multi-panels | Plusieurs panneaux de ticket (3 gratuit, unlimited premium) | 🟡 | `community_tickets/` (1 seul panel) |
| Forms/Modals | Questions avant création ticket | ✅ | `community_tickets/` (formulaire existant) |
| Staff Teams | Équipes avec rôles séparés | ❌ | `community_tickets/` |
| Tags/Canned responses | Réponses prédéfinies rapides | ❌ | `community_tickets/` |
| Transcripts HTML | Sauvegarde HTML chiffrée | ✅ | `community_tickets/` (transcript existant) |
| SLA Monitoring | Suivi temps de réponse, résolution | ❌ | `community_tickets/` |
| Auto-close | Fermeture auto des tickets inactifs | ❌ | `community_tickets/` |
| User Ratings | Notes 1-5 étoiles par les users | ❌ | `community_tickets/` |
| Live messaging | Répondre depuis le dashboard | ❌ | `community_tickets/` |
| Analytics | Stats volume, temps réponse, activité staff | ❌ | `community_tickets/` |
| Support hours | Horaires d'ouverture configurables | ❌ | `community_tickets/` |
| AI Summaries | Résumés IA des tickets (premium) | 🔶 | Hors périmètre |
| Whitelabel | Nom/avatar custom (premium) | 🔶 | Hors périmètre |

**Valeur ajoutée** : Système de support professionnel complet avec analytics et SLA.

---

## 4. Ticket Tool (tickettool.xyz)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Panneaux personnalisables | Boutons, catégories, rôles | 🟡 | `community_tickets/` (basique) |
| Forms avant ticket | Questions avant création | ✅ | `community_tickets/` |
| Custom tags | Réponses rapides prédéfinies | ❌ | `community_tickets/` |
| Storage categories | Recyclage des canaux supprimés | ❌ | `community_tickets/` |
| Google Drive transcripts | Sauvegarde transcripts sur Drive | ❌ | `community_tickets/` |
| Config backup/restore | Export/import config entre serveurs | ❌ | `community_tickets/` |
| Blacklist roles | Rôles interdits d'utiliser tickets | ❌ | `community_tickets/` |
| Global ticket limit | Limite tickets ouverts par user | ❌ | `community_tickets/` |

**Valeur ajoutée** : Gestion avancée des tickets avec backup, storage, et intégrations cloud.

---

## 5. ModMail (modmail.xyz)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Shared inbox (DM → Ticket) | DM au bot crée un canal ticket | 🟡 | `community_tickets/` (partiel: panel bouton) |
| Anonymous replies | Staff répond anonymement | ❌ | `community_tickets/` |
| Greeting/closing messages | Messages automatiques ouverture/fermeture | ✅ | `community_tickets/` |
| Snippets | Sauvegarde de messages réutilisables | ❌ | `community_tickets/` |
| User blacklisting | Interdire un user de contacter | ❌ | `community_tickets/` |
| Log viewer | Interface web de logs tickets | ❌ | `community_tickets/` |
| Custom settings | Catégorie, rôles, logs configurables | 🟡 | `community_tickets/` (partiel) |

**Valeur ajoutée** : Communication staff-membre simplifiée via DM. Alternative au système de tickets classique.

---

## 6. Statbot (statbot.net)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Server statistics | Messages, voix, member flow, activité | 🟡 | `security_logs/` (logs bruts) |
| Channel counters | Salons vocaux avec stats (membres, rôles, horloges) | ❌ | `util_statcounters/` |
| Statroles | Rôles basés sur l'activité (pas juste XP) | ❌ | `engagement_statroles/` |
| Graphs | Graphiques messages/voix/jours | ❌ | `frontend/` (pas de graphs) |
| User privacy | Anonymisation des stats user | ❌ | `security_logs/` |
| Dashboard analytics | Dashboard web avec KPIs | 🟡 | `frontend/` (basique) |
| Top members | Top membres actifs text/voix | ✅ | `engagement_xp-level/` (leaderboard) |
| Member flow | Graphique entrées/sorties | ❌ | `security_logs/` |

**Valeur ajoutée** : Analytics avancés pour comprendre l'activité du serveur.

---

## 7. Welcomer (welcomer.gg)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Welcome images custom | Images avec background, polices, couleurs | 🟡 | `welcome_cards/` (SVG basique) |
| Background upload | Upload d'image de fond custom | ❌ | `welcome_cards/` |
| Welcome DM | Message DM de bienvenue | ✅ | `welcome_welcome/` |
| Borderwall (anti-bot) | Challenge à l'arrivée (anti-spam) | ❌ | `security_captcha/` (captcha existe) |
| AutoRoles | Rôles auto à l'arrivée | ✅ | `welcome_welcome/` |
| FreeRoles | Commande pour s'assigner des rôles | ❌ | `community_reaction-roles/` (via réactions) |
| Giveaways | Concours | ✅ | `game_engagement/` |
| Polls | Sondages | ✅ | `game_engagement/` |
| Leaver messages | Message de départ | ✅ | `welcome_welcome/` |
| Rules display | Affichage des règles | ❌ | `community_rules/` |
| TempChannels | Salons vocaux temporaires | ✅ | `util_temp-voice/` |
| TimeRoles | Rôles basés sur l'ancienneté | ❌ | `community_timed-roles/` |

**Valeur ajoutée** : Welcomes images avancées et système anti-bot (Borderwall).

---

## 8. Message Planner Bot (discordmessageplannerbot.com)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| One-time scheduled messages | Programmer un message à date/heure | ❌ | `automation_scheduler/` |
| Recurring messages | Messages répétés (toutes les N minutes/heures/jours) | ❌ | `automation_scheduler/` |
| Calendar scheduling | Règles calendrier (fin de mois, etc.) | ❌ | `automation_scheduler/` |
| Visual editor | Éditeur visuel avec preview live | ❌ | `automation_scheduler/` |
| Embed builder | Jusqu'à 10 embeds, 25 champs chacun | ❌ | `automation_scheduler/` |
| Attachments | Jusqu'à 10 images par message | ❌ | `automation_scheduler/` |
| Forum/Thread delivery | Envoi dans threads et posts forum | ❌ | `automation_scheduler/` |
| Auto-publish | Publication auto dans annonces | ❌ | `automation_scheduler/` |
| Auto-clean | Auto-suppression après délai ou garder dernier | ❌ | `automation_scheduler/` |
| Templates | 100 templates réutilisables par serveur | ❌ | `automation_scheduler/` |
| Template rotation | Rotation de templates (tips du jour) | ❌ | `automation_scheduler/` |
| Export/Import | Sauvegarde complète des configurations | ❌ | `automation_scheduler/` |
| Timezone support | Fuseaux horaires IANA par schedule | ❌ | `automation_scheduler/` |
| Multi-langue | 8 langues | ❌ | `core/i18n/` |

**Valeur ajoutée** : Planification complète de messages avec builder d'embeds intégré.

---

## 9. Confessy (confessy.xyz)

| Feature | Description | Status Bot | Module cible |
|---|---|:---:|---|
| Confessions anonymes | Même feature que Confessions Bot | ❌ | `community_confessions/` |
| Anonymous threads | Threads anonymes | ❌ | `community_confessions/` |
| Anonymous identity | Identité anonyme persistante | ❌ | `community_confessions/` |
| Free logging | Logs gratuits (pas premium) | ❌ | `community_confessions/` |

---

## 10. Synthèse — Features uniques à implémenter

### 10.1 Gaps uniques par effort

#### Effort 🟢 (1-2 jours)

| Feature | Source | Module | Description courte |
|---|---|---|---|
| Confessions anonymes | Confessions Bot | `community_confessions/` | `/confess` + canal dédié |
| Anonymous replies | Confessions Bot | `community_confessions/` | `/reply` anonyme |
| Confess ban | Confessions Bot | `community_confessions/` | `/confessban` |
| Tags (canned responses) | Tickets Bot | `community_tickets/` | Réponses prédéfinies |
| Snippets | ModMail | `community_tickets/` | Messages sauvegardés |
| Auto-close tickets | Tickets Bot | `community_tickets/` | Fermeture auto inactifs |
| User ratings | Tickets Bot | `community_tickets/` | Notes 1-5 étoiles |
| One-time scheduled messages | MsgPlanner | `automation_scheduler/` | Message à date/heure |
| Staff teams | Tickets Bot | `community_tickets/` | Équipes de support |
| Config backup/restore | Ticket Tool | `community_tickets/` | Export/import config |

#### Effort 🟡 (3-5 jours)

| Feature | Source | Module | Description courte |
|---|---|---|---|
| Auto-thread | Needle | `automation_autothread/` | Thread auto par message |
| Thread title regex | Needle | `automation_autothread/` | Titre via regex/variables |
| Recurring messages | MsgPlanner | `automation_scheduler/` | Messages récurrents |
| Forms avancés | Tickets Bot | `community_tickets/` | Multi-champs, validation |
| SLA Monitoring | Tickets Bot | `community_tickets/` | Suivi temps réponse |
| Stats graphs | Statbot | `frontend/` | Graphiques d'activité |
| Channel counters | Statbot | `util_statcounters/` | Compteurs vocaux |
| TimeRoles | Welcomer | `community_timed-roles/` | Rôles par ancienneté |
| Borderwall | Welcomer | `security_captcha/` | Challenge anti-bot |
| Statroles | Statbot | `engagement_statroles/` | Rôles par activité |
| Storage categories | Ticket Tool | `community_tickets/` | Recyclage canaux |

#### Effort 🔴 (5-10 jours)

| Feature | Source | Module | Description courte |
|---|---|---|---|
| Review mode (confessions) | Confessions Bot | `community_confessions/` | Approbation avant publication |
| Analytics dashboard | Tickets Bot | `frontend/` | KPIs support + graphs |
| Visual embed builder | MsgPlanner | `automation_scheduler/` | Éditeur visuel avec preview |
| Template rotation | MsgPlanner | `automation_scheduler/` | Rotation templates |
| Multi-timezone | MsgPlanner | `automation_scheduler/` | Fuseaux IANA |
| Auto-publish | MsgPlanner | `automation_scheduler/` | Publication auto annonces |
| Full welcome image builder | Welcomer | `welcome_cards/` | Upload BG, polices, couleurs |

---

## 11. Comparaison couverture features spécialisées

| Bot source | Features totales | Déjà implémentées | À implémenter | % couvert |
|---|---:|---:|---:|---:|
| Confessions | 8 | 0 | 8 | 0% |
| Needle | 8 | 0 | 8 | 0% |
| Tickets Bot | 12 | 4 | 8 | 33% |
| Ticket Tool | 8 | 1 | 7 | 13% |
| ModMail | 7 | 2 | 5 | 29% |
| Statbot | 8 | 1 | 7 | 13% |
| Welcomer | 12 | 7 | 5 | 58% |
| MsgPlanner | 14 | 0 | 14 | 0% |
| **Total** | **77** | **15** | **62** | **19%** |

---

## 12. Recommandations prioritaires

### Tier 1 — Impact communautaire fort

| Feature | Source | Raison | Effort |
|---|---|---|---:---:|
| Confessions anonymes | Confessions | Feature communautaire très demandée | 🟢 |
| Auto-thread | Needle | Organisation salons (images, discussions) | 🟡 |
| Recurring messages | MsgPlanner | Automation essentielle (annonces, rappels) | 🟡 |
| Channel counters | Statbot | Utilitaire simple et visible | 🟡 |

### Tier 2 — Amélioration support

| Feature | Source | Raison | Effort |
|---|---|---|---:---:|
| Tags (canned responses) | Tickets Bot | UX support | 🟢 |
| Snippets | ModMail | UX support | 🟢 |
| Auto-close tickets | Tickets Bot | Maintenance automatique | 🟢 |
| SLA Monitoring | Tickets Bot | Qualité support | 🟡 |
| Staff teams | Tickets Bot | Organisation | 🟢 |

### Tier 3 — Analytics & Engagement

| Feature | Source | Raison | Effort |
|---|---|---|---:---:|
| Stats graphs | Statbot | Visibilité activité | 🟡 |
| Statroles | Statbot | Récompense activité | 🟡 |
| TimeRoles | Welcomer | Récompense fidélité | 🟡 |
| Analytics dashboard | Tickets Bot | Pilotage support | 🔴 |

### Tier 4 — Features avancées

| Feature | Source | Raison | Effort |
|---|---|---|---:---:|
| Visual embed builder | MsgPlanner | Création messages riches | 🔴 |
| Full welcome image builder | Welcomer | Personnalisation accueils | 🔴 |
| Multi-timezone | MsgPlanner | International | 🟡 |
| Auto-publish | MsgPlanner | Canaux annonces | 🟢 |

---

## 13. Voir aussi

- [`draftbot-feature-list.md`](./draftbot-feature-list.md) — extraction Draftbot
- [`mee6-feature-list.md`](./mee6-feature-list.md) — extraction MEE6
- [`dyno-feature-list.md`](./dyno-feature-list.md) — extraction Dyno
- [`carlbot-feature-list.md`](./carlbot-feature-list.md) — extraction Carl-bot
- [`roadmap-complete-features.md`](../plan/roadmap-complete-features.md) — roadmap unifiée

## 14. source 

The bots to research:
- https://confessions.bot/ and https://confessy.xyz/ - Confessions feature
- https://needle.gg/ - Auto-thread feature
- https://tickets.bot/ - Improved ticket system
- https://docs.tickettool.xyz/ - Ticket tool
- https://modmail.xyz/ - Modmail system
- https://statbot.net/ - Statistics bot
- https://welcomer.gg/ - Welcome bot
- https://www.discordmessageplannerbot.com/ - Message planning/scheduling