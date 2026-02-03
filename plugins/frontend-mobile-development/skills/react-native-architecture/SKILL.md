---
name: react-native-architecture
description: Build production React Native apps with Expo, navigation, native modules, offline sync, and cross-platform patterns. Use when developing mobile apps, implementing native integrations, or architecting React Native projects.
---

# React Native Architecture

Production-ready patterns for React Native development with Expo, including navigation, state management, native modules, and offline-first architecture.

## When to Use This Skill

- Starting a new React Native or Expo project
- Implementing complex navigation patterns
- Integrating native modules and platform APIs
- Building offline-first mobile applications
- Optimizing React Native performance
- Setting up CI/CD for mobile releases

## Core Concepts

### 1. Project Structure

```
src/
├── app/                    # Expo Router screens
│   ├── (auth)/            # Auth group
│   ├── (tabs)/            # Tab navigation
│   └── _layout.tsx        # Root layout
├── components/
│   ├── ui/                # Reusable UI components
│   └── features/          # Feature-specific components
├── hooks/                 # Custom hooks
├── services/              # API and native services
├── stores/                # State management
├── utils/                 # Utilities
└── types/                 # TypeScript types
```

### 2. Expo vs Bare React Native

| Feature            | Expo           | Bare RN        |
| ------------------ | -------------- | -------------- |
| Setup complexity   | Low            | High           |
| Native modules     | EAS Build      | Manual linking |
| OTA updates        | Built-in       | Manual setup   |
| Build service      | EAS            | Custom CI      |
| Custom native code | Config plugins | Direct access  |

## Quick Start

```bash
npx create-expo-app@latest my-app -t expo-template-blank-typescript
npx expo install expo-router expo-status-bar react-native-safe-area-context
npx expo install @react-native-async-storage/async-storage
npx expo install expo-secure-store expo-haptics
```

Root layout: Stack from expo-router, wrap with ThemeProvider and QueryProvider.

## Patterns

### Expo Router Navigation

Use Tabs and Stack; file-based routes (e.g. (tabs)/profile/[id].tsx). useLocalSearchParams for params; router.push/replace/back for programmatic navigation.

### Authentication Flow

AuthProvider with createContext, useRouter, useSegments. Store token in SecureStore; on mount check auth; effect to redirect unauthenticated users to /login and authenticated users from (auth) to /(tabs).

### Offline-First with React Query

Use PersistQueryClientProvider with createAsyncStoragePersister(AsyncStorage). Set networkMode: 'offlineFirst'. Sync online status with NetInfo and onlineManager.setEventListener. Use placeholderData to keep previous data while revalidating.

### Native Module Integration

Wrap expo-haptics, expo-local-authentication, expo-notifications in services; check Platform.OS !== 'web' where needed. Request permissions and handle channels on Android.

### Platform-Specific Code

Use Platform.select in StyleSheet, or .ios.tsx / .android.tsx / .web.tsx file variants. Use Reanimated for 60fps animations; Haptics on press for native feel.

### Performance Optimization

Use FlashList instead of FlatList; memo list items; useCallback for renderItem and keyExtractor. removeClippedSubviews, maxToRenderPerBatch, windowSize.

## EAS Build & Submit

eas.json: development (simulator), preview (internal), production. submit with appleId/ascAppId (iOS) and serviceAccountKeyPath (Android). eas update for OTA.

## Best Practices

### Do's

- Use Expo for faster development and OTA updates.
- Use FlashList over FlatList for long lists.
- Memoize components and callbacks.
- Use Reanimated for animations.
- Test on real devices.

### Don'ts

- Don't inline styles; use StyleSheet.create.
- Don't fetch in render; use useEffect or React Query.
- Don't ignore platform differences.
- Don't store secrets in code.
- Don't skip error boundaries.

## Resources

- [Expo Documentation](https://docs.expo.dev/)
- [Expo Router](https://docs.expo.dev/router/introduction/)
- [React Native Performance](https://reactnative.dev/docs/performance)
- [FlashList](https://shopify.github.io/flash-list/)
