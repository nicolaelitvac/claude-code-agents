---
name: nextjs-app-router-patterns
description: Master Next.js 14+ App Router with Server Components, streaming, parallel routes, and advanced data fetching. Use when building Next.js applications, implementing SSR/SSG, or optimizing React Server Components.
---

# Next.js App Router Patterns

Comprehensive patterns for Next.js 14+ App Router architecture, Server Components, and modern full-stack React development.

## When to Use This Skill

- Building new Next.js applications with App Router
- Migrating from Pages Router to App Router
- Implementing Server Components and streaming
- Setting up parallel and intercepting routes
- Optimizing data fetching and caching
- Building full-stack features with Server Actions

## Core Concepts

### 1. Rendering Modes

| Mode                  | Where        | When to Use                               |
| --------------------- | ------------ | ----------------------------------------- |
| **Server Components** | Server only  | Data fetching, heavy computation, secrets |
| **Client Components** | Browser      | Interactivity, hooks, browser APIs        |
| **Static**            | Build time   | Content that rarely changes               |
| **Dynamic**           | Request time | Personalized or real-time data            |
| **Streaming**         | Progressive  | Large pages, slow data sources            |

### 2. File Conventions

```
app/
├── layout.tsx       # Shared UI wrapper
├── page.tsx         # Route UI
├── loading.tsx      # Loading UI (Suspense)
├── error.tsx        # Error boundary
├── not-found.tsx    # 404 UI
├── route.ts         # API endpoint
├── template.tsx     # Re-mounted layout
├── default.tsx      # Parallel route fallback
└── opengraph-image.tsx  # OG image generation
```

## Quick Start

```typescript
// app/layout.tsx
import { Inter } from 'next/font/google'
import { Providers } from './providers'

const inter = Inter({ subsets: ['latin'] })

export const metadata = {
  title: { default: 'My App', template: '%s | My App' },
  description: 'Built with Next.js App Router',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={inter.className}>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}

// app/page.tsx - Server Component by default
async function getProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 }, // ISR: revalidate every hour
  })
  return res.json()
}

export default async function HomePage() {
  const products = await getProducts()

  return (
    <main>
      <h1>Products</h1>
      <ProductGrid products={products} />
    </main>
  )
}
```

## Patterns

### Pattern 1: Server Components with Data Fetching

Use Suspense, async Server Components, and fetch with `next: { tags }` for cache invalidation. Colocate data fetching in the component that uses it.

### Pattern 2: Client Components with 'use client'

Use 'use client' for interactivity, useState, useTransition, and Server Actions (addToCart, etc.). Handle pending and error state in the UI.

### Pattern 3: Server Actions

Use "use server" for mutations; revalidateTag/revalidatePath for cache; cookies() and redirect() from next/headers and next/navigation.

### Pattern 4: Parallel Routes

Use slots in layout (e.g. children, analytics, team) and @folder for parallel route segments. Provide default.tsx for fallback.

### Pattern 5: Intercepting Routes

Use (.)segment to intercept navigation (e.g. modal overlay). Same segment can render full page when hard-refreshed or direct URL.

### Pattern 6: Streaming with Suspense

Fetch critical data first, then wrap slow components in Suspense with fallback. Stream in reviews and recommendations independently.

### Pattern 7: Route Handlers (API Routes)

Export GET, POST, etc. from app/api/route.ts. Use NextRequest, NextResponse, getQuery, readBody. Params are Promise in App Router.

### Pattern 8: Metadata and SEO

Use generateMetadata({ params }), generateStaticParams(), and notFound(). Set title, description, openGraph, twitter.

## Caching Strategies

- fetch(url, { cache: 'no-store' }) — always fresh
- fetch(url, { next: { revalidate: 60 } }) — ISR
- fetch(url, { next: { tags: ['products'] } }) — tag-based; revalidateTag('products') in Server Action
- revalidatePath('/products') for path invalidation

## Best Practices

### Do's

- Start with Server Components; add 'use client' only when needed.
- Colocate data fetching where it's used.
- Use Suspense boundaries for streaming.
- Use Server Actions for mutations with progressive enhancement.

### Don'ts

- Don't use hooks in Server Components.
- Don't fetch in Client Components for initial data; use Server Components or React Query.
- Don't ignore loading states; provide loading.tsx or Suspense.

## Resources

- [Next.js App Router Documentation](https://nextjs.org/docs/app)
- [Server Components RFC](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md)
- [Vercel Templates](https://vercel.com/templates/next.js)
