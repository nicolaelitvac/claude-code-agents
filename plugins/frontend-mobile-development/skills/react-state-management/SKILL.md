---
name: react-state-management
description: Master modern React state management with Redux Toolkit, Zustand, Jotai, and React Query. Use when setting up global state, managing server state, or choosing between state management solutions.
---

# React State Management

Comprehensive guide to modern React state management patterns, from local component state to global stores and server state synchronization.

## When to Use This Skill

- Setting up global state management in a React app
- Choosing between Redux Toolkit, Zustand, or Jotai
- Managing server state with React Query or SWR
- Implementing optimistic updates
- Debugging state-related issues
- Migrating from legacy Redux to modern patterns

## Core Concepts

### 1. State Categories

| Type             | Description                  | Solutions                     |
| ---------------- | ---------------------------- | ----------------------------- |
| **Local State**  | Component-specific, UI state | useState, useReducer          |
| **Global State** | Shared across components     | Redux Toolkit, Zustand, Jotai |
| **Server State** | Remote data, caching         | React Query, SWR, RTK Query   |
| **URL State**    | Route parameters, search     | React Router, nuqs            |
| **Form State**   | Input values, validation     | React Hook Form, Formik       |

### 2. Selection Criteria

- Small app, simple state → Zustand or Jotai
- Large app, complex state → Redux Toolkit
- Heavy server interaction → React Query + light client state
- Atomic/granular updates → Jotai

## Quick Start: Zustand

```typescript
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'

interface AppState {
  user: User | null
  theme: 'light' | 'dark'
  setUser: (user: User | null) => void
  toggleTheme: () => void
}

export const useStore = create<AppState>()(
  devtools(
    persist(
      (set) => ({
        user: null,
        theme: 'light',
        setUser: (user) => set({ user }),
        toggleTheme: () => set((state) => ({
          theme: state.theme === 'light' ? 'dark' : 'light'
        })),
      }),
      { name: 'app-storage' }
    )
  )
)
```

## Patterns

### Redux Toolkit with TypeScript

Use configureStore, createSlice, createAsyncThunk. Export useAppDispatch and useAppSelector with typed RootState and AppDispatch. Use Immer in reducers.

### Zustand with Slices

Use StateCreator for slice functions; combine slices in one create() call. Use selective subscriptions (useStore(state => state.user)) to avoid re-renders.

### Jotai for Atomic State

Use atom(), atomWithStorage(), derived atoms, async atoms. Use useAtom for read/write, write-only atoms for actions. Works with Suspense for async atoms.

### React Query for Server State

Use queryKey factory (userKeys.list(filters)), useQuery with queryFn and staleTime/gcTime. Use useMutation with onMutate (optimistic update), onError (rollback), onSettled (invalidateQueries).

### Combining Client + Server State

Use Zustand (or Jotai) for UI/client state; React Query for server data. Keep them separate; don't duplicate server data in global state.

## Best Practices

### Do's

- Colocate state; keep it as close to where it's used as possible.
- Use selectors to prevent unnecessary re-renders.
- Separate server state (React Query) from client state (Zustand/Jotai).
- Type everything with TypeScript.

### Don'ts

- Don't over-globalize; not everything needs global state.
- Don't duplicate server state in global state.
- Don't mutate directly; use immutable updates.
- Don't store derived data; compute it instead.

## Resources

- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [Zustand GitHub](https://github.com/pmndrs/zustand)
- [Jotai Documentation](https://jotai.org/)
- [TanStack Query](https://tanstack.com/query)
