# Feature : Flux RSS, LootScraper & Alertes Multi-Sources

> **Module** : `util_autofeeds` — **Statut** : Implémenté, testé (15 tests vitest dédiés) et intégré au Dashboard Nuxt 3.

---

## 1. Objectif & Inspirations

Permettre à un serveur Discord d'agréger, filtrer et diffuser des flux d'actualités et bons plans en continu à l'instar des solutions :
- [MonitorRSS](https://monitorss.xyz/)
- [RSS.app Discord Bot](https://rss.app/bots/rssfeeds-discord-bot)
- [ReadyBot.io](https://readybot.io/)
- [FeedSync.net](https://feedsync.net/)

Le module intègre nativement les flux officiels de **[LootScraper](https://eikowagenknecht.com/lootscraper/)** (agrégateur de jeux PC/consoles 100% gratuits), un système d'**abonnements individuels par tags, catégories ou mots-clés**, et une architecture modulaire **multi-sources extensible**.

---

## 2. Fonctionnalités Clés

### 2.1 Presets LootScraper en 1-Clic (`config/presets.js`)
Catalogue de flux pré-paramétrés prêts à être installés dans n'importe quel salon Discord :
- 🎁 **LootScraper - Tous les jeux gratuits** : agrège Epic Games, Steam, GOG, Prime Gaming et Itch.io.
- ⚡ **LootScraper - Epic Games Store** : alertes hebdomadaires des jeux offerts sur Epic.
- 🎮 **LootScraper - Steam Freebies** : jeux passés temporairement à 100% de réduction sur Steam.
- 🕹️ **LootScraper - GOG.com** : offres DRM-free.
- 👑 **LootScraper - Prime Gaming** : butins et jeux Amazon Prime.
- 👾 **LootScraper - Itch.io** : jeux indépendants offerts.
- 🤖 **Reddit r/FreeGameFindings** & **r/GameDeals**
- 📰 **Google News Tech & Gaming**

### 2.2 Souscriptions Membres par Tags & Alertes Personnalisées (`autofeeds-subscription.service.js`)
Les membres du serveur Discord peuvent s'abonner individuellement pour être alertés dès qu'une publication correspond à leurs centres d'intérêt :
- **Par Tag** : ex. `#epic`, `#steam`, `#prime`, `#ps5`, `#gratuit`
- **Par Catégorie** : ex. `gaming`, `news`, `tech`, `deals`
- **Par Mots-clés** : ex. `100% off`, `giveaway`, `rtx 4090`
- **Par Flux spécifique** ou sur tous les flux du serveur
- **Modes de notification flexibles** :
  - 📢 **Mention dans le salon** : mentionne l'utilisateur (`<@userId>`) lors de la publication dans le salon Discord.
  - 📩 **Message Privé (DM)** : envoie une copie privée de l'annonce directement en DM par le bot.

### 2.3 Bouton Interactif d'Abonnement en 1-Clic sous chaque Message
Chaque publication postée dans Discord inclut un bouton d'action interactif :
- 🔗 **Voir l'offre / l'article** (bouton lien vers la source).
- 🔔 **M'alerter pour #[tag]** (bouton composant Discord).
En cliquant dessus, le membre est instantanément abonné au tag sans avoir besoin de taper une commande !

### 2.4 Moteur de Filtrage Avancé (`base.provider.js`)
Chaque flux peut être restreint par des filtres précis :
- `filterKeywords` : liste de mots-clés requis (la publication doit contenir au moins un des mots).
- `excludeKeywords` : liste de mots-clés interdits (la publication est ignorée si l'un des mots est présent, ex. `dlc`, `beta`).
- `regexFilter` : expression régulière personnalisée pour les utilisateurs avancés.

---

## 3. Architecture Multi-Sources (`services/providers/`)

Le bot utilise le pattern **`ProviderRegistry`** qui détecte automatiquement le connecteur adapté selon l'URL :

| Fournisseur | Statut | Types de Cibles | Métadonnées extraites |
| :--- | :---: | :--- | :--- |
| **RSS / Atom Universel** | 🟢 Opérationnel | Tous sites, WordPress, blogs, LootScraper | Titre, résumé HTML nettoyé, images/enclosures, tags |
| **YouTube** | 🟢 Opérationnel | `@handle`, `UC...` Channel ID, Playlists | Titre de la vidéo, miniature HD, lien de lecture |
| **Reddit** | 🟢 Opérationnel | `r/subreddit`, URL Reddit | Miniatures Reddit, flairs convertis en tags, auteur |
| **Google News** | 🟢 Opérationnel | Requêtes de recherche, sujets thématiques | Titre, source média, lien canonique |
| **Twitch** | 🟡 En Roadmap | Streamers, chaînes, jeux | Statut live, aperçu de stream, catégorie de jeu |
| **Kick** | 🟡 En Roadmap | Streamers Kick | Alertes de live instantanées |
| **X / Twitter** | 🟡 En Roadmap | Comptes `@user`, hashtags, passerelle Nitter | Tweets, retweets, médias attachés |
| **TikTok** | 🟡 En Roadmap | Comptes créateurs, sons | Nouvelles vidéos publiées |
| **Instagram** | 🟡 En Roadmap | Profils publics | Photos, carrousels, descriptions |
| **Facebook** | 🟡 En Roadmap | Pages publiques | Publications officielles |
| **LinkedIn** | 🟡 En Roadmap | Entreprises, recrutements | Offres d'emploi, posts d'actualités |

---

## 4. Commandes Slash Discord (`/feed` ou `/autofeed`)

| Commande | Permissions | Description |
| :--- | :--- | :--- |
| `/feed list` | Tous | Affiche la liste des flux actifs sur le serveur |
| `/feed add url:<url> channel:<#salon> [name] [category] [tags]` | Admin | Ajoute un nouveau flux (RSS, YouTube, Reddit...) |
| `/feed presets` | Admin | Affiche le catalogue LootScraper et permet l'installation en 1 bouton |
| `/feed delete id:<id>` | Admin | Supprime un flux enregistré |
| `/feed test id:<id>` | Admin | Force la vérification immédiate et envoie un exemple dans Discord |
| `/feed subscribe [tag] [category] [keyword] [mode:mention\|dm]` | Tous | S'abonne aux notifications pour un tag ou une catégorie |
| `/feed unsubscribe id:<subId>` | Tous | Supprime un abonnement actif |
| `/feed my-subscriptions` | Tous | Affiche ses alertes et abonnements personnels |

---

## 5. Endpoints REST API (`/api/autofeeds`)

- `GET /api/autofeeds` : Liste des flux configurés pour la guilde active.
- `POST /api/autofeeds` : Création d'un flux.
- `GET /api/autofeeds/presets` : Catalogue des presets disponibles (LootScraper, Reddit...).
- `POST /api/autofeeds/presets/install` : Installation d'un preset en 1-clic (`{ presetId, channelId }`).
- `GET /api/autofeeds/providers` : Liste des fournisseurs et capacités.
- `GET /api/autofeeds/subscriptions` : Abonnements de la guilde (filtrables par `userId` ou `feedId`).
- `POST /api/autofeeds/subscriptions` : Création d'une souscription.
- `DELETE /api/autofeeds/subscriptions/:id` : Suppression d'une souscription.
- `PATCH /api/autofeeds/:id` : Mise à jour d'un flux (statut, intervalle, filtres, tags).
- `DELETE /api/autofeeds/:id` : Suppression d'un flux.
- `POST /api/autofeeds/:id/test` : Test d'envoi immédiat du flux.

---

## 6. Interface Dashboard Nuxt 3

Accessible sur le dashboard via la section **Modules** :
- **📊 Vue d'ensemble** (`/modules/autofeeds/overview`) : Statistiques, raccourci LootScraper, flux récents et guide des commandes.
- **📰 Flux configurés** (`/modules/autofeeds/list`) : Tableau de gestion, filtres par catégorie, switch actif/pause, bouton de test direct et modal d'ajout complet avec sélecteur de salon Discord.
- **🎁 Catalogue & LootScraper** (`/modules/autofeeds/presets`) : Grille de cartes prêtes à l'emploi pour LootScraper (Epic, Steam, GOG, Prime, Itch.io) avec installation en 1-clic.
- **🔔 Abonnements & Alertes** (`/modules/autofeeds/subscriptions`) : Tableau des souscriptions utilisateurs, badges de mode (Mention / DM) et création/suppression.
- **🌐 Fournisseurs & Roadmap** (`/modules/autofeeds/providers`) : Fiches techniques des 11 sources (opérationnelles et prévues).
