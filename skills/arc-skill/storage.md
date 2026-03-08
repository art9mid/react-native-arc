# Storage Architecture

## Stack

- **react-native-mmkv** — High-performance native key-value storage (10x faster than AsyncStorage)
- Used for: auth tokens, user preferences (theme, language), onboarding flags
- Also used as React Query persistence backend

## MMKV Instance

```typescript
// src/store/storage/index.ts

import { MMKV } from 'react-native-mmkv';

// Single MMKV instance for the entire app
export const storage = new MMKV();

// --- Typed Storage Wrapper ---
// Provides type-safe get/set for JSON-serializable data

export const clientStorage = {
  getItem: <T>(key: string): T | null => {
    const value = storage.getString(key);
    if (!value) return null;
    try {
      return JSON.parse(value) as T;
    } catch {
      return value as unknown as T;
    }
  },

  setItem: <T>(key: string, value: T): void => {
    storage.set(key, JSON.stringify(value));
  },

  removeItem: (key: string): void => {
    storage.delete(key);
  },

  getString: (key: string): string | undefined => {
    return storage.getString(key);
  },

  setString: (key: string, value: string): void => {
    storage.set(key, value);
  },

  getBoolean: (key: string): boolean => {
    return storage.getBoolean(key) ?? false;
  },

  setBoolean: (key: string, value: boolean): void => {
    storage.set(key, value);
  },

  clearAll: (): void => {
    storage.clearAll();
  },
};
```

## Storage Keys

```typescript
// src/constants/keys.ts

export const STORAGE_KEYS = {
  // Auth
  TOKENS: 'settings.tokens',

  // Preferences
  COLOR_SCHEME: 'settings.colorScheme',
  LANGUAGE: 'settings.language',

  // Flags
  ONBOARDING_COMPLETED: 'settings.onboardingCompleted',
} as const;

export const QUERY_KEYS = {
  // Auth & Profile
  PROFILE: ['profile'] as const,
  USER: ['user'] as const,

  // Domain-specific (add per feature)
  // ITEMS: ['items'] as const,
  // ITEM_DETAIL: (id: string) => ['items', id] as const,
} as const;
```

## Domain Storage Modules

### Auth Token Storage

```typescript
// src/store/storage/auth.ts

import { clientStorage } from './index';
import { STORAGE_KEYS } from '@/constants/keys';
import type { Token } from '@/types/auth.types';

export const getToken = (): Token | null =>
  clientStorage.getItem<Token>(STORAGE_KEYS.TOKENS);

export const setToken = (token: Token): void =>
  clientStorage.setItem(STORAGE_KEYS.TOKENS, token);

export const removeToken = (): void =>
  clientStorage.removeItem(STORAGE_KEYS.TOKENS);

export const isAuthenticated = (): boolean => {
  const token = getToken();
  return !!token?.accessToken;
};
```

### Theme Storage

```typescript
// src/store/storage/theme.ts

import { storage } from './index';
import { STORAGE_KEYS } from '@/constants/keys';
import type { AppColorSchemeName } from '@/theme/interfaces';

export const getStoredColorScheme = (): AppColorSchemeName | null =>
  (storage.getString(STORAGE_KEYS.COLOR_SCHEME) as AppColorSchemeName) ?? null;

export const setStoredColorScheme = (scheme: AppColorSchemeName): void =>
  storage.set(STORAGE_KEYS.COLOR_SCHEME, scheme);
```

### Onboarding Storage

```typescript
// src/store/storage/onboarding.ts

import { clientStorage } from './index';
import { STORAGE_KEYS } from '@/constants/keys';

export const hasCompletedOnboarding = (): boolean =>
  clientStorage.getBoolean(STORAGE_KEYS.ONBOARDING_COMPLETED);

export const setOnboardingCompleted = (): void =>
  clientStorage.setBoolean(STORAGE_KEYS.ONBOARDING_COMPLETED, true);
```

## React Query Persistence

MMKV is used as the persistence layer for React Query cache, enabling offline-first experience:

```typescript
// src/store/services/index.ts

import { QueryClient } from '@tanstack/react-query';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';
import { clientStorage } from '@/store/storage';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
      refetchOnMount: false,
      retry: 2,
      staleTime: 1000 * 60 * 5, // 5 minutes
    },
  },
});

// Sync persister backed by MMKV
export const appPersister = createSyncStoragePersister({
  storage: {
    getItem: (key: string) => clientStorage.getString(key) ?? null,
    setItem: (key: string, value: string) => clientStorage.setString(key, value),
    removeItem: (key: string) => clientStorage.removeItem(key),
  },
});
```

## Usage Rules

1. **Never access MMKV directly in components** — always use domain storage modules
2. **Never store sensitive data unencrypted** — MMKV supports encryption if needed:
   ```typescript
   const storage = new MMKV({ encryptionKey: 'your-key' });
   ```
3. **Use STORAGE_KEYS constants** — never hardcode storage key strings
4. **Typed wrappers for complex data** — always use `clientStorage.getItem<Type>()`
5. **Clear on logout** — call `storage.clearAll()` or selectively remove auth-related keys
