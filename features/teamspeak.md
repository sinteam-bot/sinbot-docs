# Feature : Widget & Logs TeamSpeak 3

> **Module** : `util_teamspeak` — **Statut** : Implémenté et testé (54 tests unitaires & d'intégration)

## 1. Objectif

Intégrer un module de **suivi et monitoring d'un serveur TeamSpeak 3** directement dans Discord :
- **Widget arborescence en direct** : visualiser la liste des salons (avec sous-salons, spacers) et les utilisateurs connectés (avec statuts : muet, casque coupé, absent).
- **Logs d'actions en temps réel** : logger les connexions, déconnexions, déplacements et modifications de salons dans un salon Discord dédié (à l'instar des logs d'événements Discord) et dans la table d'audit `event_log`.

> ⚠️ **Important** : Ce module agit au sein du bot Discord via le protocole ServerQuery (bibliothèque [`ts3-nodejs-library`](https://github.com/Multivit4min/TS3-NodeJS-Library)). Il ne s'agit pas d'un bot vocal TeamSpeak, mais d'un widget de monitoring et de logging Discord.

---

## 2. Fonctionnalités clés

### 2.1 Arborescence des salons et utilisateurs (`TeamSpeakTreeService`)
- Construction de la hiérarchie parent-enfant (`cid` / `pid`) respectant l'ordre TS3.
- Nettoyage et mise en forme des **spacers** TS3 (`[cspacer]`, `[*spacer]`, `[spacer]`).
- Formatage des statuts utilisateurs :
  - 🔇 Casque muet (`outputMuted`)
  - 🎙️❌ Micro muet (`inputMuted`)
  - 💤 Absent avec message (`away` / `awayMessage`)
- Rendu en **arbre ASCII / Unicode élégant** intégré dans un embed Discord.
- Options de personnalisation : masquage des salons vides, masquage des clients ServerQuery, etc.

### 2.2 Widget interactif Discord en direct (`TeamSpeakWidgetService`)
- Déploiement d'un message widget persistant dans le salon Discord de votre choix via `/teamspeak widget setup #salon`.
- Mise à jour automatique en direct (avec debounce de 2 secondes) à chaque événement (connexion, déconnexion, déplacement).
- Bouton interactif **`🔄 Actualiser`** permettant à n'importe quel membre de rafraîchir l'arborescence instantanément.
- Bouton optionnel **`🎧 Rejoindre TeamSpeak`** si une URL de connexion web est configurée.

### 2.3 Logs d'actions TeamSpeak vers Discord (`TeamSpeakLogsService`)
- Détection des événements ServerQuery en temps réel :
  - 🟢 **Connexion** (`clientconnect`)
  - 🔴 **Déconnexion** (`clientdisconnect`) avec raison
  - 🔄 **Déplacement** (`clientmoved`) avec ancien et nouveau salon
  - 📁 **Gestion de salons** (`channelcreate`, `channeldelete`)
- Publication d'embeds colorés dans le salon Discord configuré (`logs.channel_id`).
- Archivage en base de données dans la table universelle `event_log` (`event_type LIKE 'ts3_%'`).
- Émission sur l'EventBus interne (`teamspeak:log` et `log.published` pour le WebSocket du dashboard).

---

## 3. Configuration YAML

Exemple de configuration (`data/example/teamspeak.config.yml` ou `data/{guildId}/teamspeak.config.yml`) :

```yaml
enabled: true
allowed_roles: []

server:
  host: "ts.monserveur.fr"
  queryport: 10011
  serverport: 9987
  protocol: "raw" # "raw" ou "ssh"
  username: "serveradmin"
  password: "mon_mot_de_passe_query"
  nickname: "DiscordTS3Widget"
  readyTimeout: 10000
  keepAlive: true

widget:
  enabled: true
  channel_id: "123456789012345678" # Salon où le message widget est posté
  message_id: null                 # ID du message édité (auto-renseigné)
  refresh_interval_seconds: 30
  title: "🔊 Serveur TeamSpeak 3"
  color: "#2580EB"
  hide_empty_channels: false
  show_query_clients: false
  show_channel_ids: true

logs:
  enabled: true
  channel_id: "987654321098765432" # Salon Discord recevant les logs
  color: "#2580EB"
  events:
    client_connect: true
    client_disconnect: true
    client_moved: true
    channel_create: true
    channel_delete: true
    server_edit: false
```

Variables d'environnement de secours supportées :
- `TS3_HOST`, `TS3_QUERY_PORT`, `TS3_SERVER_PORT`
- `TS3_USERNAME`, `TS3_PASSWORD`, `TS3_NICKNAME`, `TS3_PROTOCOL`

---

## 4. Commandes Slash

| Commande | Description | Permissions |
|---|---|---|
| `/teamspeak tree [ephemere] [masquer_vides]` | Affiche l'arborescence actuelle des salons et clients | Tous |
| `/teamspeak status` | Affiche l'état technique et les statistiques du serveur TS3 | Tous |
| `/teamspeak widget action:setup salon:#salon` | Déploie le widget interactif en direct dans le salon | Gérer le serveur |
| `/teamspeak widget action:refresh` | Force l'actualisation immédiate du widget | Gérer le serveur |
| `/teamspeak widget action:delete` | Désactive le widget pour le serveur | Gérer le serveur |
| `/teamspeak logs action:setup salon:#salon` | Configure le salon de réception des logs d'actions | Gérer le serveur |
| `/teamspeak logs action:disable` | Désactive les logs d'actions sur Discord | Gérer le serveur |

---

## 5. API REST (`/api/teamspeak`)

| Méthode | Route | Description |
|---|---|---|
| `GET` | `/api/teamspeak/status` | Statut de connexion et compteurs |
| `GET` | `/api/teamspeak/tree` | Arborescence complète au format JSON |
| `GET` | `/api/teamspeak/config` | Configuration de la guilde (mot de passe masqué) |
| `PATCH` | `/api/teamspeak/config` | Mise à jour de la configuration |
| `POST` | `/api/teamspeak/refresh` | Forcer l'actualisation du cache et du widget |
| `GET` | `/api/teamspeak/logs` | Historique paginé des logs TeamSpeak (`event_log`) |
