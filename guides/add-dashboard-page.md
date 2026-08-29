# Guide — Ajouter une page dashboard pour un feature

> Ce guide explique comment ajouter une page d'interface pour un feature existant
> (par exemple : suivre les invites, leaderboard, configuration).
>
> Pré-requis : avoir lu [`create-a-feature.md`](./create-a-feature.md) et compris
> comment fonctionnent les routes et composables Nuxt du projet.

## 1. Pré-requis

- Le feature **backend** est implémenté (service + repository + controller)
- Les **endpoints REST** sont définis dans `src/modules/<feature>/controllers/`
- Vous avez compris la structure des pages `frontend/pages/modules/<feature>.vue`

## 2. Étape 1 — Cartographier l'existant

Avant de coder, répondre à ces questions :

- Quel est le **nom** du feature (kebab-case) ? Ex: `feature_invites` → page `invites`
- Quels **endpoints REST** existent ? (lister les routes du controller)
- Quelles **données** afficher ? (stats globales, listes paginées, formulaires de config)
- Quelles **actions** l'utilisateur peut faire ? (refresh, add, remove, etc.)

Exemple : pour `feature_invites` :
- `GET /api/invites/:guildId/:userId` (stats user)
- `GET /api/invites/:guildId/leaderboard` (top inviters)
- `GET /api/invites/:guildId/blacklist` (blacklist)
- `GET /api/config` (config générale, contient `features.invites`)

## 3. Étape 2 — Créer le composable

Le composable encapsule les appels API. Il centralise la logique de fetch, le typage, et la sérialisation des query params.

`frontend/composables/useInvites.ts` :

```ts
import { useDiscordApi } from './useDiscordApi';

export interface InviteStats {
  real: number;
  bonus: number;
  leaves: number;
  fake: number;
  total: number;
}

export interface InviteLeaderboardEntry {
  inviterId: string;
  inviterUsername: string;
  real: number;
  bonus: number;
  total: number;
}

export interface BlacklistEntry {
  guildId: string;
  targetId: string;
  targetType: 'user' | 'role';
  reason: string | null;
  moderatorId: string | null;
  createdAt: number;
}

export const useInvites = () => {
  const api = useDiscordApi();

  async function getUserStats(guildId: string, userId: string): Promise<InviteStats> {
    const res = await api.apiFetch<{ success: boolean; data: InviteStats }>(
      `/api/invites/${encodeURIComponent(guildId)}/${encodeURIComponent(userId)}`
    );
    return res.data;
  }

  async function getLeaderboard(guildId: string, limit = 25): Promise<InviteLeaderboardEntry[]> {
    const qs = new URLSearchParams({ limit: String(limit) });
    const res = await api.apiFetch<{ success: boolean; data: InviteLeaderboardEntry[] }>(
      `/api/invites/${encodeURIComponent(guildId)}/leaderboard?${qs.toString()}`
    );
    return res.data;
  }

  async function getBlacklist(guildId: string): Promise<BlacklistEntry[]> {
    const res = await api.apiFetch<{ success: boolean; data: BlacklistEntry[] }>(
      `/api/invites/${encodeURIComponent(guildId)}/blacklist`
    );
    return res.data;
  }

  return { getUserStats, getLeaderboard, getBlacklist };
};
```

**Conventions** :
- Un composable par feature (cohérence : `useTickets`, `useEconomy`, etc.)
- Les **types** sont exportés en même temps que le composable
- Les **erreurs** sont propagées (le composable `useToast` les attrape dans la page)
- L'**authentification** est gérée par `useDiscordApi`

## 4. Étape 3 — Créer la page parent

La page parent contient :
- L'en-tête du module (titre, description, icône)
- La sous-navigation (onglets entre les sous-pages)
- Le `<NuxtPage />` qui injecte la sous-page active

`frontend/pages/modules/invites.vue` :

```vue
<template>
  <div class="view-panel">
    <div class="module-view-scroller">
      <div class="module-header" style="margin-bottom: 20px;">
        <div style="display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 12px;">
          <div>
            <h2 style="font-size: 22px; font-weight: 700; color: var(--header-primary); margin: 0 0 6px 0; display: flex; align-items: center; gap: 10px;">
              <span>🎟️</span> Suivi des Invitations
            </h2>
            <p class="module-desc" style="margin: 0; color: var(--text-muted); font-size: 13px;">
              Tracker qui invite qui, leaderboard des inviteurs, détection de fake invites.
            </p>
          </div>
        </div>

        <!-- Sous-navigation -->
        <div class="module-tab-nav" style="margin-top: 16px; display: flex; gap: 8px; border-bottom: 1px solid var(--border-subtle); padding-bottom: 8px; flex-wrap: wrap;">
          <NuxtLink to="/modules/invites/leaderboard" class="module-tab-btn" :class="{ active: isTabActive('/modules/invites/leaderboard') }">
            <span>🏆</span> Classement
          </NuxtLink>
          <NuxtLink to="/modules/invites/blacklist" class="module-tab-btn" :class="{ active: isTabActive('/modules/invites/blacklist') }">
            <span>⛔</span> Blacklist
          </NuxtLink>
          <NuxtLink to="/modules/invites/config" class="module-tab-btn" :class="{ active: isTabActive('/modules/invites/config') }">
            <span>⚙️</span> Configuration
          </NuxtLink>
        </div>
      </div>

      <NuxtPage />
    </div>
  </div>
</template>

<script setup lang="ts">
import { useRoute } from 'vue-router';

definePageMeta({
  title: 'Suivi des Invitations',
  icon: '🎟️',
  description: 'Tracker qui invite qui, leaderboard, fake detection',
  section: 'modules',
  order: 20
});

useSeoMeta({
  title: 'Suivi des Invitations - Bot',
  description: 'Système de tracking des invitations avec détection de fake'
});

const route = useRoute();

function isTabActive(path: string): boolean {
  if (path === '/modules/invites/leaderboard' && (route.path === '/modules/invites' || route.path === '/modules/invites/')) {
    return true;
  }
  return route.path.startsWith(path);
}
</script>

<style scoped>
.module-tab-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 14px;
  border-radius: 6px;
  font-size: 13px;
  color: var(--text-muted);
  text-decoration: none;
  transition: background 0.15s, color 0.15s;
}
.module-tab-btn:hover {
  background: var(--background-modifier-hover);
  color: var(--text-normal);
}
.module-tab-btn.active {
  background: var(--brand-experiment, #5865f2);
  color: white;
  font-weight: 600;
}
</style>
```

**Conventions `definePageMeta`** :
- `title` : affiché dans la navigation
- `icon` : emoji
- `description` : tooltip + meta
- `section` : `'modules'`
- `order` : tri dans la nav (plus petit = plus haut)

## 5. Étape 4 — Créer les sous-pages

Chaque onglet a son fichier dans `frontend/pages/modules/<feature>/<onglet>.vue`.

`frontend/pages/modules/invites/leaderboard.vue` :

```vue
<template>
  <div style="display: flex; flex-direction: column; gap: 20px;">
    <!-- Bannière stats -->
    <div class="module-stats-banner">
      <div class="module-stat-card">
        <div class="module-stat-icon">🏆</div>
        <div class="module-stat-info">
          <span class="module-stat-label">Top inviter</span>
          <span class="module-stat-value">{{ topTotal }} 📨</span>
          <span v-if="topEntry" class="module-stat-sub">
            par {{ topEntry.inviterUsername }}
          </span>
        </div>
      </div>
      <div class="module-stat-card">
        <div class="module-stat-icon">📊</div>
        <div class="module-stat-info">
          <span class="module-stat-label">Inviteurs classés</span>
          <span class="module-stat-value">{{ leaderboard.length }}</span>
        </div>
      </div>
    </div>

    <!-- Tableau leaderboard -->
    <div class="config-card">
      <div class="card-subtitle" style="display: flex; align-items: center; justify-content: space-between;">
        <span>🏆 Classement des Inviteurs</span>
        <button class="module-btn" @click="load" :disabled="loading">
          {{ loading ? '⏳' : '🔄' }} Rafraîchir
        </button>
      </div>
      <div v-if="error" style="color: var(--red);">❌ {{ error }}</div>
      <div v-else-if="loading && leaderboard.length === 0" class="config-card">
        ⏳ Chargement…
      </div>
      <div v-else-if="leaderboard.length === 0" style="color: var(--text-muted); text-align: center; padding: 40px;">
        Aucun inviteur classé pour le moment.
      </div>
      <div v-else class="module-table-wrapper">
        <table class="module-table">
          <thead>
            <tr>
              <th style="width: 80px; text-align: center;">Rang</th>
              <th>Inviteur</th>
              <th style="text-align: right;">Réelles</th>
              <th style="text-align: right;">Bonus</th>
              <th style="text-align: right;">Total</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(e, idx) in paginated" :key="e.inviterId">
              <td style="text-align: center;">
                <strong :class="medalClass(idx)">
                  {{ medalLabel(idx) }}
                </strong>
              </td>
              <td>
                <DiscordUser :user-id="e.inviterId" :show-id="true" />
                <span style="color: var(--text-muted); font-size: 12px; margin-left: 8px;">
                  @{{ e.inviterUsername }}
                </span>
              </td>
              <td style="text-align: right;">{{ e.real.toLocaleString('fr-FR') }}</td>
              <td style="text-align: right;">{{ e.bonus.toLocaleString('fr-FR') }}</td>
              <td style="text-align: right;">
                <strong style="color: var(--green);">{{ e.total.toLocaleString('fr-FR') }}</strong>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <DiscordPagination v-model="page" v-model:page-size="pageSize" :total-items="leaderboard.length" :page-size-options="[10, 25, 50, 100]" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useInvites } from '~/composables/useInvites';
import { useFeatures } from '~/composables/useFeatures';
import { useToast } from '~/composables/useToast';
import DiscordPagination from '~/components/common/DiscordPagination.vue';
import DiscordUser from '~/components/common/DiscordUser.vue';

definePageMeta({
  title: 'Classement des Invites',
  icon: '🏆',
  description: 'Top des inviters du serveur',
  section: 'modules',
  hidden: true
});

useSeoMeta({ title: 'Classement des Invites - Bot' });

const features = useFeatures();
const invites = useInvites();
const { showToast } = useToast();

const config = ref<any>(null);
const guildId = ref<string>('');
const leaderboard = ref<any[]>([]);
const loading = ref(false);
const error = ref<string | null>(null);
const page = ref(1);
const pageSize = ref(25);

const topEntry = computed(() => leaderboard.value[0] || null);
const topTotal = computed(() => topEntry.value?.total || 0);
const paginated = computed(() => {
  const start = (page.value - 1) * pageSize.value;
  return leaderboard.value.slice(start, start + pageSize.value);
});

function medalLabel(idx: number): string {
  const rank = (page.value - 1) * pageSize.value + idx + 1;
  if (rank === 1) return '🥇 #1';
  if (rank === 2) return '🥈 #2';
  if (rank === 3) return '🥉 #3';
  return `#${rank}`;
}
function medalClass(idx: number): string {
  const rank = (page.value - 1) * pageSize.value + idx + 1;
  if (rank === 1) return 'medal-gold';
  if (rank === 2) return 'medal-silver';
  if (rank === 3) return 'medal-bronze';
  return '';
}

async function load() {
  loading.value = true;
  error.value = null;
  try {
    // Récupère le guild_id depuis la config
    const featureState = await features.get('invites');
    guildId.value = featureState?.guild_id || '';
    if (!guildId.value) {
      error.value = 'Feature non configurée pour ce serveur.';
      leaderboard.value = [];
      return;
    }
    leaderboard.value = await invites.getLeaderboard(guildId.value, 1000);
  } catch (e: any) {
    error.value = e.message || 'Erreur inconnue';
    showToast({ type: 'error', message: error.value });
  } finally {
    loading.value = false;
  }
}

onMounted(load);
</script>

<style scoped>
.medal-gold { color: #f1c40f; }
.medal-silver { color: #bdc3c7; }
.medal-bronze { color: #e67e22; }
.module-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border-radius: 6px;
  background: var(--background-modifier-hover);
  color: var(--text-normal);
  font-size: 12px;
  border: 1px solid var(--border-subtle);
  cursor: pointer;
  font-family: inherit;
}
.module-btn:hover:not(:disabled) {
  background: var(--brand-experiment, #5865f2);
  color: white;
}
</style>
```

**Composants communs réutilisables** :
- `DiscordUser` : `<DiscordUser :user-id="..." :show-id="true" />`
- `DiscordChannel` : `<DiscordChannel :channel-id="..." />`
- `DiscordTime` : `<DiscordTime :value="timestamp" mode="relative" />`
- `DiscordPagination` : `<DiscordPagination v-model="page" v-model:page-size="size" :total-items="total" :page-size-options="[10,25,50,100]" />`

## 6. Étape 5 — Routing

Nuxt 3 fait du **file-based routing** automatique :
- `frontend/pages/modules/invites.vue` → `/modules/invites`
- `frontend/pages/modules/invites/leaderboard.vue` → `/modules/invites/leaderboard`

Le `<NuxtPage />` dans `invites.vue` injecte la sous-page active selon l'URL.

## 7. Étape 6 — Tester

```bash
cd frontend
npm run build   # vérifie qu'il n'y a pas d'erreur TypeScript
npm run dev     # test manuel
```

## 8. Étape 7 — Build prod

```bash
cd frontend
npm run generate   # build statique
# ou
npm run build
```

Le `nuxt.config.ts` est en mode SPA statique (`ssr: false`). Les pages sont prérendues.

## 9. Checklist finale

- [ ] Composable créé dans `frontend/composables/`
- [ ] Page parent créée dans `frontend/pages/modules/`
- [ ] Sous-pages créées dans `frontend/pages/modules/<feature>/`
- [ ] `definePageMeta` configuré (title, icon, description, section, order)
- [ ] `useSeoMeta` configuré
- [ ] `DiscordUser`, `DiscordChannel`, `DiscordTime`, `DiscordPagination` utilisés si pertinent
- [ ] Gestion d'erreurs avec `useToast().showToast`
- [ ] `onMounted(load)` pour charger les données
- [ ] Pas de secrets en dur dans le code

## 10. Anti-patterns à éviter

- ❌ **Appeler `apiFetch` directement** dans la page → toujours passer par un composable
- ❌ **Hardcoder `guildId`** → le lire depuis `features.get('<feature>')` ou l'URL
- ❌ **Oublier `useSeoMeta`** → nécessaire pour le SEO et le breadcrumb dynamique
- ❌ **Mettre le CSS dans le global** → utiliser `<style scoped>` dans chaque page
- ❌ **Oublier le `loading` / `error` state** → toujours gérer les 3 états (idle, loading, error)
- ❌ **Faire du SQL/DB côté frontend** → toujours via les endpoints REST du backend

## 11. Exemple complet : feature `feature_invites`

Voir l'implémentation réelle :
- `frontend/composables/useInvites.ts`
- `frontend/pages/modules/invites.vue`
- `frontend/pages/modules/invites/leaderboard.vue`
- `frontend/pages/modules/invites/blacklist.vue`
- `frontend/pages/modules/invites/config.vue`
