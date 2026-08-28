# Changelog de la feature reaction-roles (Phase 10)

> Document d'historique des versions du module `feature_reaction-roles`.

## v1 — Phase 10 initiale (livrée)

- Table `reaction_roles` (PK, message_id, emoji, role_id, mode, etc.)
- Listener `messageReactionAdd` / `messageReactionRemove`
- Slash `/reactionrole-add|remove|list` + REST CRUD
- Dashboard `/modules/reaction-roles/{overview,roles,config}`

## v2 — Phase 10 enrichissement (livrée)

- Table `reaction_roles` étendue : `kind` (reaction|button|select),
  `metadata` (JSON).
- Nouveau composant partagé `src/services/interactive-message-builder.js`
  (analogue à `CardRendererService`) :
  - `validateComponent` (valide les limites Discord)
  - `buildRow` / `buildMessage` (construit les ActionRow)
  - `makeCustomId` / `parseCustomId` (`ir:<id>[:<suffix>]`)
  - `execute` (dispatch vers toggle_role / give_role / take_role /
    open_url pour les buttons, give_role idempotent pour les selects)
- Listener v2 : handler `interactionCreate` pour les custom_id `ir:*`
  (avec garde-fou `self_assignable`).
- 2 nouveaux slash commands (admin) :
  - `/reactionrole-add-button` (label, style, emoji, action, role, url)
  - `/reactionrole-add-select` (placeholder, min/max, options
    au format `Label|value|roleId?` séparées par `;`)
- 2 nouveaux endpoints REST :
  - `POST /api/reaction-roles/button`
  - `POST /api/reaction-roles/select`
- Nouvelle page dashboard `/modules/reaction-roles/components`
  avec 2 onglets internes (Bouton / Select), formulaires de création
  guidés, validation côté client, liste des components existants.

### Limites appliquées (cohérentes avec Discord)

- 5 ActionRows × 5 components max par message (= 25 actions max)
- 25 options max par select
- 80 chars par label, 100 par description / value, 100 par custom_id
- custom_id préfixé `ir:` (le listener filtre sur ce préfixe)

### Tests (35 dans `tests/interactive-message-builder.test.js`)

- Validation button : kind manquant, label manquant/trop long,
  action invalide, roleId manquant, url manquante/non-http,
  button toggle_role/open_url valides
- Validation select : options vide, > 25, label vide, description
  trop longue, 25 options valides
- buildRow / buildMessage : 1 button, 5 buttons, 6 buttons → throw,
  25 buttons en 5 rows, 26 → throw, mix buttons + select groupés
- makeCustomId / parseCustomId : simple, avec suffix, trop long, prefix
- execute button : open_url, toggle add, toggle remove, give_role
  idempotent
- execute select : ajoute manquants, no_selection, ignore sans roleId
