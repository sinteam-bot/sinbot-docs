# Feature : Bienvenue avancée

> **Phase** : 6 — **Priorité** : 🟢 Basse — **Statut** : partiellement implémenté (welcome basique), à enrichir

## 1. Objectif

Remplacer le message d'accueil texte basique par :

- Une **carte de bienvenue** générée dynamiquement (image)
- Un **système d'autoroles** configurable (plusieurs rôles attribués d'un coup)
- Un **compteur de boosts** affiché en temps réel
- Un **compteur de membres** avec messages clés (1er, 100e, 1000e…)

## 2. État actuel

`src/modules/welcome_welcome/` existe avec :
- Message embed de bienvenue (config `welcome_message` + `dm_message`)
- AUTO_ROLES supportés (mais simple)

## 3. Spécifications

### 3.1 Configuration

```yaml
features:
  welcome:
    enabled: true
    welcome_channel_id: "WELCOME_CHANNEL_ID"
    auto_roles: ["MEMBER_ROLE_ID", "NOTIF_ROLE_ID"]
    card:
      enabled: true
      template: "default"      # 'default' | 'minimal' | 'custom'
      background: "url_or_path"
      avatar_position: "center"
      text_color: "#FFFFFF"
      accent_color: "#F2C7CE"
      font: "Inter"
      show_join_position: true
      show_account_age: true
      show_server_stats: true
    embed:
      enabled: true
      color: "#F2C7CE"
      title: "🎉 Bienvenue sur le serveur !"
      description: "Bienvenue {user} !"
      footer: "Membre #{memberCount}"
      show_member_count: true
      show_boost_count: true
    dm:
      enabled: true
      title: "👋 Bienvenue !"
      description: "..."
    milestones:
      enabled: true
      channel_id: "MILESTONES_CHANNEL_ID"
      thresholds: [1, 10, 50, 100, 500, 1000, 5000]
      template: "🎯 Le serveur passe à {count} membres !"
```

### 3.2 Carte de bienvenue (image)

Génération via `canvas` (paquet `canvas` ou `@napi-rs/canvas` pour perf) :

```
┌─────────────────────────────────┐
│  Background gradient            │
│                                 │
│         ┌──────────┐            │
│         │  AVATAR  │            │
│         └──────────┘            │
│                                 │
│      Welcome, {username}        │
│      Membre #{position}         │
│                                 │
│   Compte créé le {date}        │
│   Serveur : {memberCount}       │
└─────────────────────────────────┘
```

- **Cache** : 1 image par `userId` pendant 7 jours (réutilisation multi-arrivées)
- **Format** : PNG 1024x512 (ratio idéal Discord)
- **Stockage** : en mémoire + fichier `./data/welcome-cards/{userId}.png` (gitignored)

### 3.3 Compteur de boosts

Affichage dans l'embed + carte :

```js
const boostCount = message.guild.premiumSubscriptionCount;
const boostTier = message.guild.premiumTier; // 0, 1, 2, 3
```

Mise à jour à chaque `guildMemberUpdate` (changement de boost).

### 3.4 Milestones

Quand le compteur de membres croise un seuil configuré :

```
🎯 Le serveur passe à 1000 membres !
Merci à tous pour votre soutien 💜
```

- Vérifier toutes les X minutes via cron (ou sur `guildMemberAdd`)
- Éviter le spam : un message par seuil
- Stockage `welcome_milestones(guild_id, threshold, reached_at)`

### 3.5 Architecture

```
src/modules/welcome_welcome/
├── welcome.module.js
├── config/schema.js
├── db/schema.js                  # welcome_milestones, welcome_settings
├── services/
│   ├── welcome.service.js        # orchestrateur
│   ├── welcome-card.service.js   # génération d'image
│   ├── welcome-embed.service.js  # embed builder
│   ├── welcome-dm.service.js
│   ├── auto-roles.service.js
│   ├── boost-counter.service.js
│   └── milestone.service.js
├── events/
│   ├── guild-member-add.listener.js
│   ├── guild-member-update.listener.js   # pour les boosts
│   └── guild-update.listener.js          # pour les changements de vanity
├── commands/
│   ├── welcome-test.command.js          # DM + salon avec mention
│   ├── welcome-config.command.js
│   └── welcome-card.command.js
├── controllers/
│   └── welcome.controller.js
└── tests/
    ├── card.test.js
    ├── milestones.test.js
    └── auto-roles.test.js
```

## 4. Dépendances

```bash
npm install canvas              # implémentation native (nécessite Cairo)
# ou
npm install @napi-rs/canvas      # alternative pure JS via N-API (recommandé)
```

## 5. UI (Nuxt)

- Page `/features/welcome` : config embed, DM, auto-rôles, milestones
- Prévisualisation de la carte (rendu côté client via canvas)
- Aperçu du message embed
- Bouton "Test" : envoie un message test avec l'utilisateur courant

## 6. Critères d'acceptation

- [ ] Carte d'image générée correctement
- [ ] Cache des cartes fonctionnel
- [ ] Auto-rôles attribués au join
- [ ] Compteur de boosts à jour
- [ ] Milestones postés une seule fois par seuil
- [ ] Page de config fonctionnelle avec prévisualisation
- [ ] Aucune erreur si Cairo n'est pas installé (fallback embed seul)
