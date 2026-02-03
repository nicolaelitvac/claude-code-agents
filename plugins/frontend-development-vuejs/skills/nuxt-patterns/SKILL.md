---
name: nuxt-patterns
description: Master Nuxt 3 with file-based routing, useAsyncData, useFetch, server routes, and middleware. Use when building Nuxt applications, implementing SSR/SSG, or optimizing Vue server-side rendering.
---

# Nuxt 3 Patterns

Comprehensive patterns for Nuxt 3 architecture, data fetching, server routes, and full-stack Vue development.

## When to Use This Skill

- Building new Nuxt 3 applications
- Implementing SSR, SSG, or hybrid rendering
- Setting up data fetching with useFetch and useAsyncData
- Creating server API routes and middleware
- Optimizing SEO with useHead and useSeoMeta
- Deploying with Nitro to various targets

## Core Concepts

### 1. Rendering and Data

| Approach        | When to Use                          |
| --------------- | ------------------------------------ |
| **useFetch**    | Simple GET from one URL, SSR-safe    |
| **useAsyncData** | Complex logic, transforms, keyed cache |
| **$fetch**      | Client-only or event-driven requests |
| **Lazy**        | useLazyAsyncData / useLazyFetch for non-blocking load |

### 2. Directory Structure

```
nuxt-app/
├── app.vue              # Root (optional; else default)
├── nuxt.config.ts
├── pages/                # File-based routing
│   ├── index.vue
│   ├── [id].vue
│   └── admin/
│       └── users.vue
├── layouts/
│   ├── default.vue
│   └── auth.vue
├── components/
├── composables/
├── server/
│   ├── api/              # /api/* routes
│   │   └── products.get.ts
│   └── routes/           # Custom server routes
├── middleware/
│   └── auth.ts
├── plugins/
└── public/
```

## Quick Start

```vue
<!-- app.vue or layouts/default.vue -->
<template>
  <div>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </div>
</template>
```

```vue
<!-- pages/index.vue -->
<script setup lang="ts">
const { data: products, pending, error, refresh } = await useFetch('/api/products', {
  key: 'products',
  transform: (data) => data.items,
})
</script>

<template>
  <div>
    <div v-if="pending">Loading...</div>
    <div v-else-if="error">{{ error.message }}</div>
    <ul v-else>
      <li v-for="p in products" :key="p.id">{{ p.name }}</li>
    </ul>
  </div>
</template>
```

## Patterns

### Pattern 1: useAsyncData with Key and Watch

```typescript
// pages/products/[[category]].vue
<script setup lang="ts">
const route = useRoute()
const category = computed(() => route.params.category as string)

const { data: products, pending } = await useAsyncData(
  () => `products-${category.value}`,
  () => $fetch('/api/products', { params: { category: category.value || undefined } }),
  { watch: [category] }
)
</script>
```

### Pattern 2: Server API Routes

```typescript
// server/api/products.get.ts
export default defineEventHandler(async (event) => {
  const query = getQuery(event)
  const products = await db.product.findMany({
    where: query.category ? { category: query.category } : undefined,
    take: 20,
  })
  return products
})

// server/api/products.post.ts
export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  const product = await db.product.create({ data: body })
  return product
})
```

### Pattern 3: Middleware (Auth)

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const user = useUser()
  if (!user.value && to.path.startsWith('/dashboard')) {
    return navigateTo('/login')
  }
})
```

```vue
<!-- pages/dashboard.vue -->
<script setup lang="ts">
definePageMeta({ middleware: 'auth' })
</script>
```

### Pattern 4: SEO and Head

```vue
<script setup lang="ts">
const route = useRoute()
const { data: post } = await useFetch(() => `/api/posts/${route.params.id}`)

useHead({
  title: () => post.value?.title ?? 'Post',
  meta: [
    { name: 'description', content: () => post.value?.excerpt ?? '' },
  ],
})

useSeoMeta({
  ogTitle: () => post.value?.title,
  ogDescription: () => post.value?.excerpt,
})
</script>
```

### Pattern 5: Lazy Data and Suspense

```vue
<script setup lang="ts">
const { data: slowData } = await useLazyAsyncData('slow', () => $fetch('/api/slow'))
</script>

<template>
  <Suspense>
    <template #default>
      <Content :data="slowData" />
    </template>
    <template #fallback>
      <div>Loading...</div>
    </template>
  </Suspense>
</template>
```

### Pattern 6: Error Handling

```vue
<!-- error.vue -->
<template>
  <div>
    <h1>{{ error?.statusCode ?? 500 }}</h1>
    <p>{{ error?.message }}</p>
    <button @click="clearError">Go home</button>
  </div>
</template>

<script setup lang="ts">
const error = useError()
const clearError = () => clearError({ redirect: '/' })
</script>
```

## Best Practices

### Do's

- Use **useFetch** or **useAsyncData** for initial page data so it runs on the server and is serialized to the client.
- Use a **key** for useAsyncData to control cache and refetch behavior.
- Put server-only logic in **server/** (api or routes).
- Use **middleware** for auth and route guards.
- Set **useHead** / **useSeoMeta** for SEO.

### Don'ts

- Don't use **$fetch** alone for initial page data if you need SSR (no automatic serialization).
- Don't use Node-only APIs in client-side code; use server routes or Nitro server utils.
- Don't forget to handle **pending** and **error** in the UI.

## Resources

- [Nuxt 3 Documentation](https://nuxt.com/docs)
- [Data Fetching](https://nuxt.com/docs/getting-started/data-fetching)
- [Server Directory](https://nuxt.com/docs/guide/directory-structure/server)
- [Nitro Deployment](https://nitro.unjs.io/deploy)
