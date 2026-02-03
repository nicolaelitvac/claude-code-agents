---
name: vue-state-management
description: Master Vue state management with Pinia, provide/inject, and server state (useFetch, TanStack Query). Use when setting up global state, stores, or choosing state management in Vue/Nuxt apps.
---

# Vue State Management

Comprehensive guide to Vue 3 state management: Pinia stores, provide/inject, composables, and server state with useFetch or TanStack Query (Vue Query).

## When to Use This Skill

- Setting up global state with Pinia in a Vue or Nuxt app
- Designing stores (option vs setup style) and composables
- Managing server state with useFetch, useAsyncData, or Vue Query
- Implementing optimistic updates and caching
- Migrating from Vuex to Pinia
- Sharing state between Vue web and Capacitor mobile

## Core Concepts

### 1. State Categories

| Type             | Description                  | Solutions                          |
| ---------------- | ---------------------------- | ---------------------------------- |
| **Local State**  | Component-specific, UI state | ref, reactive, computed             |
| **Global State** | Shared across the app        | Pinia                               |
| **Server State** | Remote data, caching         | useFetch, useAsyncData, Vue Query   |
| **URL State**    | Route params, query          | useRoute, useRouter                 |
| **Form State**   | Inputs, validation           | v-model, VeeValidate, FormKit      |

### 2. Selection

```
Simple shared state → Pinia (one or a few stores)
Heavy server data   → useFetch/useAsyncData (Nuxt) or Vue Query
Scoped tree state   → provide/inject
```

## Quick Start: Pinia

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null as User | null,
    theme: 'light' as 'light' | 'dark',
  }),
  getters: {
    isAuthenticated: (state) => !!state.user,
  },
  actions: {
    setUser(user: User | null) {
      this.user = user
    },
    toggleTheme() {
      this.theme = this.theme === 'light' ? 'dark' : 'light'
    },
  },
})
```

```vue
<script setup lang="ts">
const userStore = useUserStore()
</script>

<template>
  <header :class="userStore.theme">
    {{ userStore.user?.name }}
    <button @click="userStore.toggleTheme">Toggle Theme</button>
  </header>
</template>
```

## Patterns

### Pattern 1: Pinia Setup Stores (Composition-style)

```typescript
// stores/cart.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCartStore = defineStore('cart', () => {
  const items = ref<CartItem[]>([])

  const total = computed(() =>
    items.value.reduce((sum, i) => sum + i.price * i.quantity, 0)
  )

  function add(item: CartItem) {
    const existing = items.value.find((i) => i.id === item.id)
    if (existing) existing.quantity += item.quantity
    else items.value.push({ ...item, quantity: item.quantity ?? 1 })
  }

  function remove(id: string) {
    items.value = items.value.filter((i) => i.id !== id)
  }

  return { items, total, add, remove }
})
```

### Pattern 2: Pinia with Persistence (Nuxt)

```typescript
// stores/settings.ts
import { defineStore } from 'pinia'

export const useSettingsStore = defineStore('settings', {
  state: () => ({
    theme: 'light',
    sidebarOpen: true,
  }),
  persist: true, // requires nuxt-ui or pinia-plugin-persistedstate
})
```

With `pinia-plugin-persistedstate`:

```typescript
// plugins/pinia.ts
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.$pinia.use(piniaPluginPersistedstate)
})
```

### Pattern 3: provide/inject for Scoped State

```vue
<!-- ParentLayout.vue -->
<script setup lang="ts">
const theme = ref<'light' | 'dark'>('light')
provide('theme', theme)
provide('setTheme', (v: 'light' | 'dark') => { theme.value = v })
</script>

<template>
  <div>
    <slot />
  </div>
</template>
```

```vue
<!-- Child.vue -->
<script setup lang="ts">
const theme = inject<Ref<'light' | 'dark'>>('theme')
const setTheme = inject<(v: 'light' | 'dark') => void>('setTheme')
</script>
```

### Pattern 4: Server State with useFetch (Nuxt)

```vue
<script setup lang="ts">
const { data: users, pending, refresh } = await useFetch('/api/users', {
  key: 'users',
  default: () => [],
})
</script>
```

### Pattern 5: TanStack Query (Vue Query) for Complex Server State

```typescript
// composables/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/vue-query'

export const userKeys = {
  all: ['users'] as const,
  list: (filters: UserFilters) => [...userKeys.all, 'list', filters] as const,
  detail: (id: string) => [...userKeys.all, 'detail', id] as const,
}

export function useUsers(filters: UserFilters) {
  return useQuery({
    queryKey: userKeys.list(filters),
    queryFn: () => $fetch('/api/users', { params: filters }),
    staleTime: 5 * 60 * 1000,
  })
}

export function useUpdateUser() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: (user: User) => $fetch(`/api/users/${user.id}`, { method: 'PUT', body: user }),
    onSuccess: (_, variables) => {
      queryClient.invalidateQueries({ queryKey: userKeys.detail(variables.id) })
    },
  })
}
```

### Pattern 6: Combining Pinia + Server State

```vue
<script setup lang="ts">
const uiStore = useUIStore()
const { data: dashboard } = await useFetch('/api/dashboard', { key: 'dashboard' })
</script>

<template>
  <div :class="{ 'sidebar-open': uiStore.sidebarOpen }">
    <Sidebar />
    <main>
      <DashboardContent v-if="dashboard" :data="dashboard" />
    </main>
  </div>
</template>
```

## Best Practices

### Do's

- Prefer **Pinia** for global client state in new Vue/Nuxt apps.
- Use **useFetch** / **useAsyncData** in Nuxt for server state when possible.
- Keep state **colocated**; only lift to Pinia when shared.
- Use **getters** for derived state to avoid duplication.
- Type stores and composables with **TypeScript**.

### Don'ts

- Don't store server data in Pinia when useFetch/useAsyncData can hold it (avoids duplication).
- Don't mutate state outside actions in option stores; in setup stores, mutations are explicit.
- Don't over-use global state; prefer provide/inject for feature-scoped state.

## Resources

- [Pinia Documentation](https://pinia.vuejs.org/)
- [Nuxt Data Fetching](https://nuxt.com/docs/getting-started/data-fetching)
- [TanStack Query Vue](https://tanstack.com/query/latest/docs/framework/vue/overview)
