# Feature : Flux RSS, LootScraper & Alertes Multi-Sources

> **Module** : `util_autofeeds` — **Statut** : Implémenté, testé (21 tests vitest dédiés, 102/102 suites au vert) et intégré au Dashboard Nuxt 4.

---

## 1. Objectif & Inspirations

Permettre à un serveur Discord d'agréger, filtrer et diffuser des flux d'actualités, vidéos, streams et bons plans en continu à l'instar des solutions :
- [MonitorRSS](https://monitorss.xyz/)
- [RSS.app Discord Bot](https://rss.app/bots/rssfeeds-discord-bot)
- [ReadyBot.io](https://readybot.io/)
- [FeedSync.net](https://feedsync.net/)

Le module intègre nativement les flux officiels de **[LootScraper](https://eikowagenknecht.com/lootscraper/)** (agrégateur de jeux PC/consoles 100% gratuits), un système d'**abonnements individuels par tags, auteurs/comptes, catégories ou mots-clés**, et une architecture modulaire **multi-sources de 11 connecteurs unifiés**.

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

### 2.2 Souscriptions Membres par Tags, Comptes & Alertes Personnalisées (`autofeeds-subscription.service.js`)
Les membres du serveur Discord peuvent s'abonner individuellement pour être alertés dès qu'une publication correspond à leurs centres d'intérêt :
- **Par Tag** : ex. `#epic`, `#steam`, `#prime`, `#ps5`, `#gratuit`
- **Par Compte / Auteur** : ex. `@PlayStation`, `@Zerator`, `@Dealabs`
- **Par Catégorie** : ex. `gaming`, `news`, `tech`, `deals`
- **Par Mots-clés** : ex. `100% off`, `giveaway`, `rtx 4090`
- **Par Flux spécifique** ou sur tous les flux du serveur
- **Filtres personnels d'abonné** : chaque abonné peut restreindre ses alertes avec ses propres mots-clés requis (`includeKeywords`), mots-clés interdits (`excludeKeywords`) ou regex (`regexFilter`).
- **Modes de notification flexibles** :
  - 📢 **Mention dans le salon** : mentionne l'utilisateur (`<@userId>`) lors de la publication dans le salon Discord.
  - 📩 **Message Privé (DM)** : envoie une copie privée de l'annonce directement en DM par le bot.

### 2.3 Boutons Interactifs d'Abonnement en 1-Clic sous chaque Message Discord
Chaque publication postée dans Discord inclut une rangée d'actions interactives :
- 🔗 **Voir l'article** (bouton lien direct vers la source officielle).
- 🔔 **Suivre #[tag]** (bouton composant Discord pour s'abonner instantanément au tag principal).
- 👤 **Suivre @[auteur]** (bouton composant Discord pour s'abonner aux publications de ce créateur/compte).

En cliquant dessus, le membre est instantanément abonné sans avoir besoin de taper une commande slash !

### 2.4 Moteur de Filtrage Avancé (`base.provider.js`)
Chaque flux peut être restreint par des filtres précis :
- `includeKeywords` (ou `filterKeywords`) : mots-clés requis dans le titre ou le contenu.
- `excludeKeywords` : mots-clés interdits (ex. `dlc`, `beta`, `payant`).
- `titleKeywords` : mots-clés requis spécifiquement dans le titre.
- `excludeTitleKeywords` : mots-clés interdits dans le titre.
- `authorInclude` / `authorExclude` : liste blanche ou liste noire d'auteurs (ex. exclure `AutoModerator`).
- `tagInclude` / `tagExclude` : filtres de tags XML ou de flairs Reddit.
- `requireMedia` : impose la présence d'une image ou d'un média pour publier l'article.
- `regexFilter` : expression régulière personnalisée insensible à la casse.

---

## 3. Architecture Multi-Sources (`services/providers/`)

Le bot s'appuie sur le pattern centralisé **`ProviderRegistry`** qui détecte automatiquement le connecteur adapté selon l'URL ou l'identifiant saisi :

| Fournisseur | Statut | Types de Cibles | Métadonnées extraites |
| :--- | :---: | :--- | :--- |
| **RSS / Atom Universel** | 🟢 Opérationnel | Tous sites, WordPress, blogs, LootScraper, XML | Titre, résumé HTML nettoyé, images/enclosures, tags XML, dates |
| **YouTube Vidéos** | 🟢 Opérationnel | `@handle`, `UC...` Channel ID, Playlists | Titre de la vidéo, miniature HD, lien de lecture direct, auteur |
| **YouTube Live** | 🟢 Opérationnel | `@handle/live`, chaînes /live, WebSub | Détection temps réel des lives, statut en direct, thumbnail, passage offline automatique |
| **Reddit** | 🟢 Opérationnel | `r/subreddit`, URL Reddit | Miniatures Reddit, flairs convertis en tags, auteur |
| **Google News** | 🟢 Opérationnel | Requêtes de recherche, sujets thématiques | Titre, source média, lien canonique |
| **Twitch** | 🟢 Opérationnel | Chaînes Twitch, streamers | Statut live temps réel, jeu/catégorie, nombre de spectateurs, miniature, clôture in-place |
| **Kick** | 🟢 Opérationnel | Chaînes Kick, streamers | Statut live API v2, aperçu de stream, catégorie de jeu, clôture in-place |
| **X / Twitter** | 🟢 Opérationnel | Comptes `@user`, URLs `x.com` / `twitter.com`, Nitter | Tweets, retweets, liens officiels, hashtags convertis en tags |
| **TikTok** | 🟢 Opérationnel | Comptes créateurs `@user`, URLs TikTok, ProxiTok | Vidéos, liens canoniques TikTok, auteur |
| **Instagram** | 🟢 Opérationnel | Profils publics Instagram (passerelle RSSHub) | Photos, carrousels, auteur, tags |
| **Facebook** | 🟢 Opérationnel | Pages publiques Facebook (passerelle RSSHub) | Publications officielles, auteur, liens |
| **LinkedIn** | 🟢 Opérationnel | Entreprises LinkedIn (passerelle RSSHub) | Actualités d'entreprises, offres de recrutement |

---

## 4. Cycle de Vie des Lives & Alertes en Direct

### 4.1 Détection & Notification
- Les flux en direct (`twitch`, `kick`, `youtube_live`) bénéficient d'un intervalle resserré par défaut de **2 minutes** (contre 15 min pour les vidéos et 30 min pour les flux RSS).
- Lorsqu'une diffusion commence, un message Discord riche est publié dans le salon cible avec mentions combinées (rôle ping global + souscripteurs individuels du streamer).
- Le message personnalisé accepte les variables dynamiques :
  - `{streamer}` : nom du diffuseur
  - `{title}` : titre du live
  - `{game}` : jeu ou catégorie en cours
  - `{viewers}` : nombre de spectateurs
  - `{url}` : lien direct vers la diffusion
  - `{mentions}` : pings des abonnés et rôles
- La session active est sauvegardée dans la table `autofeed_live_sessions` avec l'ID du message Discord envoyé.

### 4.2 Clôture In-Place sans Ghost-Ping & Gestion des Threads
- Dès que le streamer coupe son live, le bot détecte l'état hors ligne.
- Le message Discord initial est **mis à jour sur place (edit in-place)** :
  - Le titre devient `⚫ [OFFLINE] [Streamer] n'est plus en direct`.
  - La couleur passe en gris discret (`#747F8D`).
  - La durée totale de la diffusion (ex: `2h 15min`) et le dernier jeu joué sont affichés.
  - Le bouton d'action devient `🎬 Voir la chaîne / Replay`.
  - Le contenu textuel de mentions est **vidé** pour éliminer tout risque de *ghost-ping*.
- **Threads Discord dédiés (`createThread: true`)** :
  - Un fil de discussion Discord peut être ouvert automatiquement sous l'annonce de direct pour centraliser les réactions communautaires.
  - À la clôture du live, le fil reçoit un message annonçant la fin de la diffusion puis est archivé automatiquement (`threadAutoArchiveDuration`).
- **Attribution automatique de rôle (`subscriberRoleId`)** :
  - Un rôle dédié peut être assigné aux membres dès qu'ils cliquent sur le bouton d'abonnement ou via `/feed subscribe`.
- La session est archivée en base de données.

### 4.3 Webhooks Entrants & Fallback Polling
- **Twitch EventSub** (`POST /api/webhooks/twitch` et `/api/autofeeds/webhooks/twitch`) : supporte la vérification du challenge (`webhook_callback_verification`) et les événements `stream.online` / `stream.offline`.
- **YouTube WebSub (PubSubHubbub)** (`GET/POST /api/webhooks/youtube`) : supporte la vérification `hub.challenge` et l'ingestion XML immédiate.
- En cas de coupure ou d'absence de configuration webhook, le cycle de scrutation en arrière-plan (`pollFeeds`) assure le fallback transparent automatique sans nécessiter aucun compte développeur.

---

## 5. Commandes Slash Discord (`/feed` ou `/autofeed`)

| Commande | Permissions | Description |
| :--- | :--- | :--- |
| `/feed list` | Tous | Affiche la liste des flux actifs sur le serveur |
| `/feed streamers` | Tous | Affiche le statut en temps réel (🔴 EN DIRECT ou ⚫ Hors ligne) de tous les streamers configurés |
| `/feed add url:<url> channel:<#salon> [nom] [categorie] [tags] [intervalle]` | Admin | Ajoute un nouveau flux (RSS, YouTube, Reddit, X, Twitch...) |
| `/feed presets` | Admin | Affiche le catalogue LootScraper et permet l'installation en 1 bouton |
| `/feed pause id:<id>` | Admin | Met en pause ou réactive un flux sans le supprimer |
| `/feed delete id:<id>` | Admin | Supprime un flux enregistré |
| `/feed test id:<id>` | Admin | Force la vérification immédiate et prévisualise l'embed dans Discord |
| `/feed subscribe [tag] [compte] [categorie] [mot_cle] [flux_id] [mode]` | Tous | S'abonne aux notifications (mention salon, DM, les deux, ou rôle dédié) |
| `/feed unsubscribe [tag] [compte] [categorie] [mot_cle] [flux_id]` | Tous | Supprime un abonnement actif et retire le rôle attribué |
| `/feed my-subscriptions` | Tous | Affiche la liste de ses alertes et abonnements personnels |

---

## 6. Endpoints REST API (`/api/autofeeds` & `/api/webhooks`)

- `GET /api/autofeeds` : Liste des flux configurés pour la guilde active.
- `POST /api/autofeeds` : Création d'un flux (supporte `createThread`, `subscriberRoleId`, `notificationDelivery`).
- `GET /api/autofeeds/presets` : Catalogue des presets disponibles (LootScraper, Reddit, Google News).
- `POST /api/autofeeds/presets/install` : Installation d'un preset en 1-clic (`{ presetId, channelId }`).
- `GET /api/autofeeds/providers` : Liste des 12 fournisseurs et capacités.
- `GET /api/autofeeds/subscriptions` : Abonnements de la guilde (filtrables par `guild_id` ou `user_id`).
- `POST /api/autofeeds/subscriptions` : Création d'une souscription avec mode de notification (`channel`, `dm`, `both`, `role`).
- `DELETE /api/autofeeds/subscriptions/:id` : Suppression d'une souscription.
- `PATCH /api/autofeeds/:id` : Mise à jour d'un flux (options de stream, threads, rôle dédié, statut, intervalle, filtres, tags, couleur).
- `DELETE /api/autofeeds/:id` : Suppression d'un flux.
- `POST /api/autofeeds/:id/test` : Test d'envoi immédiat du flux sans impacter l'historique anti-doublon.
- `POST /api/webhooks/twitch` : Webhook EventSub Twitch (challenge verification + notifications stream.online / stream.offline).
- `GET /api/webhooks/youtube` : Challenge WebSub Hub YouTube.
- `POST /api/webhooks/youtube` : Notification WebSub YouTube.

---

## 7. Interface Dashboard Nuxt 4

Accessible sur le dashboard via la section **Modules** et **Configuration** :
- **📊 Vue d'ensemble** (`/modules/autofeeds/overview`) : Statistiques dynamiques, héro LootScraper, flux récents et guide des commandes.
- **📰 Flux configurés** (`/modules/autofeeds/list`) : Grille de gestion, filtre rapide `🔴 Directs & Lives`, badges d'état `LIVE`, switch actif/pause, badges `⚠️ En erreur (X/10)`, `🧵 Thread`, `🏷️ Rôle auto`, bouton de test direct ⚡ et modal d'ajout/édition avec gestion des threads et rôles d'abonnés.
- **🎁 Catalogue & LootScraper** (`/modules/autofeeds/presets`) : Grille de cartes prêtes à l'emploi pour LootScraper (Epic, Steam, GOG, Prime, Itch.io) avec installation en 1-clic.
- **🔔 Abonnements & Alertes** (`/modules/autofeeds/subscriptions`) : Tableau complet des souscriptions membres avec badges colorés (Tag, Compte, Catégorie, Mot-clé, Flux), filtres personnels et création/suppression.
- **🌐 Fournisseurs & Architecture** (`/modules/autofeeds/providers`) : Fiches techniques des 12 sources opérationnelles avec formats d'URL supportés et exemples.
- **⚙️ Configuration Globale** (`/config/autofeeds`) : Clés Twitch & YouTube par défaut, cadences d'interrogation (lives 2m, vidéos 15m, RSS 30m), seuils d'erreurs et salon de log Discord global.
- **🛡️ Configuration Serveur** (`/panel/[guild]/config/autofeeds`) : Surcharges spécifiques par serveur (clés d'API dédiées, salon de logs serveur, alertes d'erreurs).

---

## 8. Surveillance de Santé, Circuit Breaker & Résilience

Pour protéger les ressources du bot et du serveur Discord :
1. **Compteur d'échecs (`failCount`)** : Chaque échec d'accès à un flux (timeout, 404, parsing) incrémente un compteur dédié.
2. **Badge visuel d'avertissement** : Dès le premier échec, un badge `⚠️ En erreur (X/10)` s'affiche sur la carte du flux dans le dashboard avec le détail du message d'erreur au survol.
3. **Alerte modérateur / admin log** : Après **3 échecs consécutifs**, le bot poste automatiquement une alerte dans le salon de logs configuré (`log_channel_id`).
4. **Disjoncteur automatique (Circuit Breaker)** : Après **10 échecs consécutifs**, le flux est **automatiquement désactivé** (`isActive: false`) pour éviter le spam réseau et préserver les quotas d'API, et un avertissement final est consigné dans les logs.
5. **Rétablissement transparent** : Dès qu'une vérification réussit (ou lors d'un test manuel ⚡ réussi), le compteur `failCount` est immédiatement remis à zéro et le statut repasse en `ok`.


