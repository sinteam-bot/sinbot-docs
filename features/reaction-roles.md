# Feature : Rôles à Réaction & Messages Interactives (Phase 10 v2)

> **Phase** : 10 — **Priorité** : 🟡 Haute — **Statut** : ✅ v1 livrée + 🟡 enrichissement v2 en cours

## 1. Objectif

Permettre à un admin de configurer un message Discord avec :
- **Réactions** : ajouter/retirer un rôle quand un user clique sur un emoji (déjà implémenté v1)
- **Boutons** : action customisable (toggle_role, give_role, take_role, open_url, custom_id)
- **Select menus** : choix unique ou multiple parmi une liste d'options (chacune liée optionnellement à un rôle)

Le tout est limité à 5 ActionRows × 5 components par message (= 25 actions max), conformément aux limites Discord.

## 2. Cas d'usage

| Cas | Comportement |
|---|---|
| **Self-roles** | "Réagis avec 🎮 pour Gamer, 🎨 pour Artist" |
| **Verify** | Bouton "✅ Verify" → donne le rôle Verified |
| **Tickets** | Bouton "📩 Open ticket" → crée un ticket |
| **Poll-like** | Select menu "Choisis ton rôle" avec options pré-écrites |
| **Multi-select** | "Quels sont tes centres d'intérêt ? (max 3)" |
| **Open URL** | Bouton "📖 Documentation" → lien externe |
| **Custom** | Bouton "🎁 Réclamer" → log custom_id pour futur hook |

## 3. Spécifications

### 3.1 Modèle de données (étend `reaction_roles`)

On **étend** la table `reaction_roles` existante (Phase 10 v1) avec :
- `kind` : enum `'reaction' | 'button' | 'select'` (default `'reaction'`)
- `metadata` : TEXT (JSON sérialisé) — contient les champs spécifiques au type

Backward-compatible : les entrées existantes ont `kind='reaction'` et `metadata=null`. Le listener `messageReactionAdd/Remove` continue de fonctionner normalement.

**Métadata par kind** :

```jsonc
// kind = 'button'
{
  "label": "Verify",                  // requis
  "style": "primary",                 // primary | secondary | success | danger | link
  "emoji": "✅",                       // optionnel
  "action": "toggle_role",             // toggle_role | give_role | take_role | open_url | custom
  "role_id": "1234567890",            // requis sauf pour open_url / custom
  "url": "https://...",                // requis si action = open_url
  "custom_id_suffix": "claim_loot_42" // optionnel, ajouté au custom_id
}

// kind = 'select' (string select uniquement)
{
  "placeholder": "Choisis tes rôles…",   // optionnel
  "min_values": 1,                      // default 1 (obligatoire)
  "max_values": 1,                      // default 1 (single-select), > 1 = multi-select
  "options": [
    { "label": "Gamer",      "value": "gamer",  "role_id": "111", "description": "Jeux vidéo" },
    { "label": "Artist",     "value": "artist", "role_id": "222" },
    { "label": "Developer",  "value": "dev",    "role_id": "333", "description": "Code" }
  ]
}
```

### 3.2 Limites de sécurité

- **Max 25 actions par message** (5 ActionRows × 5 components)
- **Max 25 options par select** (limite Discord)
- **Max 80 chars par label** / **100 par description** / **100 par value** / **100 par custom_id**
- **Custom_id préfixé `ir:`** pour distinguer des autres handlers Discord
- **Cooldown** par user×action (5s) pour éviter les abus

## 4. Architecture : composant partagé `InteractiveMessageBuilder`

Comme pour `CardRendererService` (Phase 6), on crée **un service partagé** réutilisable par d'autres features (welcome, tickets, etc.) :

```
src/services/interactive-message-builder.js   ← composant partagé
```

- `buildMessage(components)` : ActionRow[] depuis une liste de composants
- `validateComponent(comp)` : lève une erreur si limites dépassées
- `makeCustomId(comp)` : `ir:<actionId>` ou `ir:<actionId>:<suffix>`
- `execute(interaction, component, member)` : dispatch vers la bonne action

`feature_reaction-roles/` devient un **consommateur** de ce composant partagé :

```
feature_reaction-roles/
├── services/
│   ├── reaction-roles.repository.js  (étendu avec kind + metadata)
│   ├── reaction-roles.service.js     (parse metadata + validation)
│   └── interactive-listener.js       (events: messageReaction + interactionCreate)
```

## 5. Base de données (ALTER TABLE)

```sql
-- Phase 10 v2: étendre reaction_roles
ALTER TABLE reaction_roles ADD COLUMN IF NOT EXISTS kind TEXT NOT NULL DEFAULT 'reaction';
ALTER TABLE reaction_roles ADD COLUMN IF NOT EXISTS metadata TEXT;
CREATE INDEX IF NOT EXISTS idx_reaction_roles_kind ON reaction_roles(guild_id, kind, message_id);
```

## 6. API publique

### `InteractiveMessageBuilder` (partagé)

- `buildMessage(components)` : ActionRow[] depuis une liste de `components`
- `validateComponent(comp)` : lève une erreur si limites dépassées
- `makeCustomId(comp)` : `ir:<actionId>` ou `ir:<actionId>:<suffix>`
- `execute(interaction, component, member)` : dispatch vers la bonne action

### `ReactionRolesService` (étendu)

- `create({guildId, channelId, messageId, kind, roleId, emoji, metadata, ...})` :
  - si kind='reaction' : `roleId` + `emoji` requis, metadata null
  - si kind='button' : `metadata` = {label, style, action, roleId, url, customIdSuffix}
  - si kind='select' : `metadata` = {placeholder, minValues, maxValues, options}
- `list(messageId)` : retourne tous les composants configurés sur un message
- `findForInteraction(customId, messageId, kind)` : retrouve l'action qui matche

## 7. REST API (étendu)

Routes existantes de Phase 10 v1 + :
- `POST /api/reaction-roles/button` body : `{guildId, channelId, messageId, label, style, emoji?, action, roleId?, url?, customIdSuffix?}`
- `POST /api/reaction-roles/select` body : `{guildId, channelId, messageId, placeholder?, minValues?, maxValues?, options: [{label, value, roleId?, description?}]}`

## 8. Slash commands (étendus)

Existants + nouveaux :
- `/reactionrole-add-button channel message_id label style:Primary action:ToggleRole role:@role [emoji:🎉] [custom_id_suffix:...]`
- `/reactionrole-add-select channel message_id option1:Label1|value1|@role option2:... [min_values:1] [max_values:1] [placeholder:...]`

## 9. Événements Discord écoutés

- `messageReactionAdd` : Phase 10 v1 (kind=reaction)
- `messageReactionRemove` : Phase 10 v1 (kind=reaction)
- `interactionCreate` : Phase 10 v2 (kind=button/select, custom_id `ir:*`)
- `messageDelete` : Phase 10 v1 (cleanup)

## 10. Critères d'acceptation (DoD v2)

- [x] Table `reaction_roles` étendue avec `kind` + `metadata`
- [x] `InteractiveMessageBuilder` partagé créé dans `src/services/`
- [x] Listener `interactionCreate` dispatch vers la bonne action
- [x] Toggle role sur click button / select option
- [x] Support de open_url (button style='link')
- [x] Support de select unique ET multi (max_values > 1)
- [x] Tests unitaires sur InteractiveMessageBuilder (limites, validation)
- [x] Page frontend mise à jour avec formulaires de création
- [x] Aucune régression sur les reactions existantes (kind='reaction')

## 11. Tests

- `InteractiveMessageBuilder.buildMessage` : 5 buttons / 1 select avec 5 options
- `InteractiveMessageBuilder` : valide les limites (cap à 25, labels courts)
- `InteractiveMessageBuilder.execute` : toggle_role add/remove selon l'état actuel
- `InteractiveMessageBuilder.execute` : select multi → applique toutes les options sélectionnées
- Custom_id préfixé correctement
- Aucune régression sur le listener `messageReactionAdd`
