# Diagramme — EventBus

## 1. Pourquoi un EventBus ?

Sans EventBus, chaque module ferait :

```js
client.on('messageCreate', myHandler);
client.on('messageCreate', otherHandler);
client.on('messageCreate', yetAnotherHandler);
```

**Problèmes** :
- 3 listeners Discord.js pour le même event (perf)
- Si l'un throw, les autres ne sont pas garantis d'être appelés
- Pas de filtrage centralisé (chaque handler refait ses checks)
- Difficile de désactiver une feature à chaud

## 2. Avec l'EventBus

```
                            ┌────────────────────┐
                            │   Discord Client   │
                            │   (discord.js)     │
                            └────────────────────┘
                                       │
                                       │ 'messageCreate' (1 seul listener)
                                       ▼
                            ┌────────────────────┐
                            │   EventBus         │
                            │   _dispatch()      │
                            └────────────────────┘
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        │                              │                              │
        ▼                              ▼                              ▼
┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
│ Subscriber #1    │         │ Subscriber #2    │         │ Subscriber #3    │
│ priority: 100    │         │ priority: 50     │         │ priority: 10     │
│ configKey: auto  │         │ configKey: count │         │ ignoreBots: true │
└──────────────────┘         └──────────────────┘         └──────────────────┘
        │                              │                              │
        ▼                              ▼                              ▼
   AutoModEngine                 RoadToInfiniteListener          XPListener
   (vérifie spam,                (vérifie counter)              (gain XP)
    bad-words, etc)
```

## 3. Flux détaillé

```js
// 1. Discord émet l'event
client.emit('messageCreate', message);

// 2. L'EventBus (déjà enregistré via _ensureDiscordListener) intercepte
eventBus.dispatch('messageCreate', message);

// 3. dispatch() itère sur les subscribers (triés par priority DESC)
for (const sub of subscribers) {
  const { handler, options, context } = sub;

  // 3a. Vérification configKey
  if (options.configKey) {
    const modConfig = getConfig()[options.configKey];
    if (modConfig && modConfig.enabled === false) continue;  // skip
  }

  // 3b. Filtrage spécifique messageCreate
  if (eventName === 'messageCreate') {
    if (options.ignoreBots && message.author?.bot) continue;
    if (options.channelId && message.channel?.id !== options.channelId) continue;
  }

  // 3c. Filtre custom
  if (options.filter && !options.filter(...args)) continue;

  // 3d. Exécution isolée
  try {
    if (context) {
      await handler.apply(context, args);
    } else {
      await handler(...args);
    }
  } catch (err) {
    console.error(`❌ [EventBus] Erreur dans le gestionnaire pour "${eventName}":`, err);
    // Continue avec les autres subscribers
  }
}
```

## 4. Déclaration côté module

```js
const { OnEvent } = require('../../core');

@OnEvent('messageCreate', {
  configKey: 'features.automod',   // ← désactivé si la feature est off
  ignoreBots: true,
  priority: 100,                    // ← exécuté en premier
  filter: (message) => !message.system && !message.webhookId
})
class MessageCreateListener {
  static inject = [AutomodEngine];

  constructor(engine) {
    this.engine = engine;
  }

  async handle(message) {
    await this.engine.onMessageCreate(message);
  }
}
```

## 5. Options disponibles

| Option | Type | Défaut | Description |
|---|---|---|---|
| `configKey` | string | `null` | Chemin dans la config YAML (ex: `'features.automod'`). Si la valeur est `enabled: false`, le handler est skippé. |
| `ignoreBots` | boolean | `true` | Ignore les messages de bots (uniquement pour `messageCreate`). |
| `channelId` | string | `null` | Filtre par salon Discord (uniquement pour `messageCreate`). |
| `priority` | number | `0` | Plus la valeur est haute, plus le handler est exécuté tôt. |
| `filter` | function | `null` | Prédicat custom : `(args) => boolean`. Retourner `false` pour skippé. |

## 6. Désactivation à chaud

Quand un admin toggle une feature off dans le dashboard :

```
Dashboard toggle "automod" → PATCH /api/features/automod
   │
   ▼
featureRegistry.set(guildId, 'automod', { enabled: false, ... })
   │
   ├─► UPDATE feature_flags SET enabled = 0
   │
   └─► eventBus.emit('feature.updated', { name, enabled: false })
          │
          └─► (futur) Invalide le cache configKey
```

> **Aujourd'hui** : la lecture `getConfig()[options.configKey]` est faite à chaque event → la désactivation est **immédiate** sans cache à invalider.

## 7. Isolation des erreurs

```js
try {
  await handler.apply(context, args);
} catch (err) {
  console.error(`❌ [EventBus] Erreur dans le gestionnaire pour "${eventName}":`, err);
  // Continue avec le subscriber suivant, ne propage pas
}
```

Conséquence : si l'automod crash sur un message bizarre, le counter et l'XP continuent de tourner.

## 8. Souscriptions multiples (legacy cache)

L'EventBus attache aussi en dur (dans `init()`) les listeners de **synchronisation du cache BDD** :

```js
client.on('guildMemberAdd', member => DiscordCacheService.cacheSingleMember(member));
client.on('guildMemberRemove', member => DiscordCacheService.softDeleteMember(member.id));
// ... etc pour roles, channels, emojis, messages
```

Ce sont des listeners **directement** sur le client, pas via `dispatch()`. Ils doivent rester pour la cohérence du cache (utilisés par le dump Discord).

## 9. Limites & évolutions futures

- **Limite actuelle** : 100 listeners max (`setMaxListeners(100)`)
- **Évolution** : `priority` est en mémoire seulement, pas persisté
- **Évolution** : pas de retry automatique sur erreur (géré par le handler)
- **Évolution** : pas de métriques (combien d'events/sec par type) — à ajouter via Prometheus ou simple compteur
