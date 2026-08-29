# Feature : Tickets / Support

> **Phase** : 3 — **Priorité** : 🟡 Moyenne — **Statut** : à implémenter

## 1. Objectif

Permettre aux membres d'ouvrir un ticket via un bouton persistant, et aux modérateurs de gérer ces tickets (claim, réponse, clôture) avec transcript automatique.

## 2. Cas d'usage

- Un utilisateur clique sur "📩 Ouvrir un ticket" dans un salon dédié
- Un thread (ou salon privé) est créé automatiquement
- Seuls l'utilisateur, le staff autorisé et le bot y ont accès
- Un staff peut "claim" le ticket
- Le staff peut fermer le ticket → transcript généré en BDD + log dans un salon

## 3. Spécifications

### 3.1 Configuration

```yaml
features:
  tickets:
    enabled: false
    allowed_roles: ["STAFF_ROLE_ID"]
    panel:
      channel_id: "TICKET_PANEL_CHANNEL_ID"
      title: "📩 Support"
      description: "Clique sur le bouton ci-dessous pour ouvrir un ticket."
      button_label: "Ouvrir un ticket"
      button_emoji: "📩"
      color: "#5865F2"
    categories:
      - id: "support"
        label: "Support"
        emoji: "❓"
        description: "Question ou problème général"
        staff_roles: ["STAFF_ROLE_ID"]
      - id: "report"
        label: "Signalement"
        emoji: "🚨"
        description: "Signaler un membre ou un message"
        staff_roles: ["MOD_ROLE_ID", "ADMIN_ROLE_ID"]
      - id: "partner"
        label: "Partenariat"
        emoji: "🤝"
        description: "Proposition de partenariat"
        staff_roles: ["ADMIN_ROLE_ID"]
    settings:
      max_open_per_user: 3
      auto_close_after_days: 7
      transcript_channel_id: "TRANSCRIPT_LOG_CHANNEL_ID"
      ping_staff_on_open: true
      staff_role_to_ping: "STAFF_ROLE_ID"
      modal_fields:
        - { id: "subject", label: "Sujet", style: "short", required: true }
        - { id: "description", label: "Description", style: "paragraph", required: true, min_length: 20, max_length: 1500 }
```

### 3.2 Cycle de vie

```
[Closed ◄── Close ──── Open ──claim──► Claimed ──close──► Closed]
                          ▲                                 │
                          └──────── Reopen (admin) ──────────┘
```

| Statut | Description | Permissions |
|---|---|---|
| `open` | Créé, pas encore pris en charge | staff peut claim |
| `claimed` | Pris en charge par un staff | seul le claimer + admin peuvent répondre |
| `closed` | Fermé, transcript enregistré | read-only |

### 3.3 Slash commands

| Commande | Description | Permission |
|---|---|---|
| `/ticket close [reason]` | Fermer le ticket courant | staff / claimer |
| `/ticket claim` | Prendre en charge | staff |
| `/ticket unclaim` | Lâcher le ticket | claimer / admin |
| `/ticket add <user>` | Ajouter un utilisateur au ticket | staff |
| `/ticket remove <user>` | Retirer un utilisateur | staff |
| `/ticket rename <name>` | Renommer le salon | staff |
| `/ticket transcript` | Forcer la génération du transcript | staff |
| `/ticket reopen` | Rouvrir un ticket fermé | admin |

## 4. Architecture

```
src/modules/community_tickets/
├── tickets.module.js
├── config/schema.js
├── db/schema.js                       # tickets, ticket_messages
├── events/
│   ├── interaction-create.listener.js # bouton + modals
│   └── message-create.listener.js     # capture des messages
├── services/
│   ├── ticket.service.js              # CRUD + state machine
│   ├── panel.service.js               # envoi du panel
│   ├── modal.service.js               # construction des modals
│   ├── transcript.service.js          # génération du transcript
│   └── ticket-permissions.service.js
├── commands/
│   ├── ticket-close.command.js
│   ├── ticket-claim.command.js
│   ├── ticket-unclaim.command.js
│   ├── ticket-add.command.js
│   ├── ticket-remove.command.js
│   ├── ticket-rename.command.js
│   ├── ticket-transcript.command.js
│   └── ticket-reopen.command.js
├── ui/panel.embed.js                  # builder d'embed
├── controllers/
│   └── tickets.controller.js
└── tests/
    ├── ticket.test.js
    ├── transcript.test.js
    └── permissions.test.js
```

## 5. Base de données

Voir [`../architecture/data-model.md`](../architecture/data-model.md#33-tickets-phase-3) — tables `tickets` et `ticket_messages`.

## 6. Flux d'ouverture

```
User clique "Ouvrir un ticket"
  │
  ▼
Discord envoie interactionCreate (customId: "ticket:open")
  │
  ▼
PanelService affiche un modal avec les champs configurés
  │
  ▼
User soumet le modal (customId: "ticket:modal")
  │
  ▼
TicketService.create({
  userId, category, subject, description
})
  │
  ├─► Crée un thread parent du salon panel (preferred) OU
  │   un salon privé avec permissions overwrites
  │
  ├─► Insère en BDD (table tickets)
  │
  ├─► Envoie l'embed d'ouverture dans le thread/salon
  │
  ├─► Ping le staff_role si ping_staff_on_open = true
  │
  └─► Répond à l'utilisateur en ephemeral : "✅ Ticket créé : <#CHANNEL_ID>"
```

## 7. Transcript

À la fermeture d'un ticket :

1. Lire tous les messages du salon/thread
2. Formater en HTML (embed avec `description`, `fields`, `footer`)
3. Insérer dans `ticket_messages` (snapshot complet)
4. Optionnel : poster dans `transcript_channel_id`
5. Optionnel : DM le transcript à l'utilisateur

> Note : avec un thread, on perd l'historique au-delà de 90 jours via l'API Discord. Le transcript BDD est donc **indispensable**.

## 8. UI (Nuxt)

- Page `/tickets` : liste filtrable (open / claimed / closed)
- Détail d'un ticket : messages, participants, claimer
- Bouton "Reprendre dans Discord" (lien vers le channel)
- Page de config : catégories, panel, channels

## 9. Critères d'acceptation

- [ ] Panel cliquable ouvre un modal
- [ ] Ticket créé avec les bonnes permissions
- [ ] Claim / unclaim fonctionne
- [ ] Close génère un transcript en BDD
- [ ] Max N tickets ouverts par utilisateur respecté
- [ ] Auto-close après X jours d'inactivité
- [ ] Slash commands testées
- [ ] Page `/tickets` opérationnelle
- [ ] Aucune fuite de permissions (vérifier `permissionOverwrites`)
