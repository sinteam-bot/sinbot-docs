# Architecture du Dashboard Web (Nuxt 3)

> Le dashboard permet de piloter le bot depuis un navigateur : configuration des features, visualisation des logs et stats, gestion de la modération.

## 1. Stack

| Couche | Techno |
|---|---|
| Framework | Nuxt 3 (Vue 3, Vite, Nitro) |
| State | Pinia |
| HTTP | `ofetch` (intégré Nuxt) |
| UI | Tailwind (recommandé) + composants custom |
| Auth | API key (header `x-api-key` ou `?api_key=`) + IP allowlist |
| Mode | SSR pour le SEO / statique pour le build (`npm run generate`) |

## 2. Arborescence

```
frontend/
├── app.vue
├── nuxt.config.ts
├── pages/
│   ├── index.vue
│   ├── features/
│   │   ├── index.vue             # Liste + toggles
│   │   ├── automod.vue           # Formulaire de config automod
│   │   ├── tickets.vue
│   │   └── logs.vue
│   ├── moderation/
│   │   ├── index.vue             # Dashboard mod
│   │   ├── users.vue
│   │   └── logs.vue
│   ├── leaderboard.vue
│   └── settings.vue
├── components/
│   ├── FeatureCard.vue
│   ├── FeatureToggle.vue
│   ├── BadWordsEditor.vue
│   ├── SanctionRulesEditor.vue
│   └── LogLiveFeed.vue
├── composables/
│   ├── useFeatures.ts
│   ├── useApi.ts
│   └── useWebSocket.ts
├── stores/
│   ├── features.ts
│   ├── auth.ts
│   └── logs.ts
└── assets/
    └── styles/
```

## 3. Communication avec le backend

### 3.1 REST (via `useApi`)

```ts
// composables/useApi.ts
export const useApi = () => {
  const config = useRuntimeConfig();
  const apiKey = useState('apiKey', () => localStorage.getItem('api_key') || '');

  return $fetch.create({
    baseURL: config.public.apiBase,
    headers: { 'x-api-key': apiKey.value },
    onRequest({ options }) {
      if (apiKey.value) options.headers['x-api-key'] = apiKey.value;
    }
  });
};
```

```ts
// composables/useFeatures.ts
export const useFeatures = () => {
  const api = useApi();
  const store = useFeaturesStore();

  const list = () => api('/api/features');
  const update = (name: string, payload: FeatureUpdate) =>
    api(`/api/features/${name}`, { method: 'PATCH', body: payload });

  return { list, update, store };
};
```

### 3.2 WebSocket (live logs)

```ts
// composables/useWebSocket.ts
export const useWebSocket = (path: string) => {
  const config = useRuntimeConfig();
  const url = `${config.public.wsBase}${path}?api_key=${useApiKey()}`;
  const ws = new WebSocket(url);

  return ws;
};
```

## 4. Pinia stores

### 4.1 `stores/features.ts`

```ts
export const useFeaturesStore = defineStore('features', () => {
  const features = ref<Record<string, FeatureState>>({});
  const loading = ref(false);

  const load = async () => {
    loading.value = true;
    const { list } = useFeatures();
    features.value = await list();
    loading.value = false;
  };

  const toggle = async (name: string) => {
    const current = features.value[name];
    const { update } = useFeatures();
    const updated = await update(name, { enabled: !current.enabled });
    features.value[name] = updated;
  };

  return { features, loading, load, toggle };
});
```

## 5. Pages clés

### 5.1 `/features` — Liste des features

- Cards par feature (nom, description, toggle on/off)
- Indicateur "actif sur N serveurs"
- Lien vers la page de config dédiée

### 5.2 `/features/automod` — Config

- Éditeur de liste de bad-words (textarea + import/export)
- Sliders pour les seuils (anti-spam, anti-raid)
- Configuration de l'escalade de sanctions
- Rôles autorisés (multi-select depuis la liste des rôles cachés)
- Bouton "Test" pour simuler une sanction

### 5.3 `/moderation` — Dashboard

- KPIs : warns 24h, bans 7j, top contrevenants
- Tableau paginé des `mod_logs` avec filtres
- Actions rapides : warn, mute, kick, ban (modale)

### 5.4 `/logs` — Live feed

- WebSocket connecté sur `/ws/logs`
- Filtre par type (message_delete, member_join…)
- Auto-scroll + pause au survol

## 6. Authentification

- L'utilisateur saisit la clé API une fois, stockée en `localStorage`
- `useApi` l'envoie automatiquement dans `x-api-key`
- Backend : middleware Express `requireApiKey` (cf. `src/web/middlewares/auth.js`)
- L'IP allowlist est vérifiée **côté backend** uniquement

## 7. Build

```bash
# Dev
npm run dev:ui

# Build statique (recommandé pour hébergement simple)
npm run build:ui
# → frontend/.output/public/  (à servir par Express ou Nginx)

# Build SSR (si besoin d'auth dynamique)
cd frontend && npm run build
# → frontend/.output/server/index.mjs
```

Le mode `generate` (statique) est le mode par défaut (`npm run build:ui`). Le backend sert ensuite le dossier `.output/public/` via `express.static`.

## 8. Variables d'environnement (frontend)

```bash
# frontend/.env
NUXT_PUBLIC_API_BASE=https://bot.example.com
NUXT_PUBLIC_WS_BASE=wss://bot.example.com
```

## 9. Roadmap UI

- [x] Auth API key
- [x] Liste des features
- [ ] Config par feature (bad-words, seuils…)
- [ ] Dashboard modération
- [ ] Live logs
- [ ] Visualisation XP / leaderboard
- [ ] Gestion des tickets
- [ ] Sondages en live
