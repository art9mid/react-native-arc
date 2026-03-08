# State Management

## Architecture

| State Type | Solution | Location |
|------------|----------|----------|
| Server state (API data) | TanStack React Query | `src/store/services/` |
| Auth tokens | MMKV storage | `src/store/storage/auth.ts` |
| User preferences | MMKV storage | `src/store/storage/theme.ts` |
| Theme / Init state | React Context | `src/providers/app-initialization/` |
| Component UI state | React `useState` | Component-local |
| Form state | Formik | Component-local |

**No Redux, Zustand, or Jotai.** React Query + MMKV + Context covers all needs.

## React Query Setup

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
      gcTime: 1000 * 60 * 60 * 24, // 24 hours (garbage collection)
    },
    mutations: {
      retry: 0,
    },
  },
});

export const appPersister = createSyncStoragePersister({
  storage: {
    getItem: (key: string) => clientStorage.getString(key) ?? null,
    setItem: (key: string, value: string) => clientStorage.setString(key, value),
    removeItem: (key: string) => clientStorage.removeItem(key),
  },
});
```

## Query Hook Pattern

```typescript
// src/store/services/auth.ts

import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import { loginApi, registerApi, logoutApi, getProfileApi } from '@/api/http/auth';
import { setToken, removeToken } from '@/store/storage/auth';
import { QUERY_KEYS } from '@/constants/keys';
import type { LoginDTO, RegisterDTO, Token, Profile } from '@/types/auth.types';
import type { ApiServiceErr } from '@/types/api.types';

// --- Queries ---

export const useProfileQuery = () =>
  useQuery<Profile, ApiServiceErr>({
    queryKey: QUERY_KEYS.PROFILE,
    queryFn: getProfileApi,
  });

// --- Mutations ---

export const useLoginMutation = () => {
  const queryClient = useQueryClient();

  return useMutation<Token, ApiServiceErr, LoginDTO>({
    mutationFn: loginApi,
    onSuccess: (token) => {
      setToken(token);
      // Optionally pre-fetch profile
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.PROFILE });
    },
  });
};

export const useRegisterMutation = () => {
  const queryClient = useQueryClient();

  return useMutation<Token, ApiServiceErr, RegisterDTO>({
    mutationFn: registerApi,
    onSuccess: (token) => {
      setToken(token);
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.PROFILE });
    },
  });
};

export const useLogoutMutation = () => {
  const queryClient = useQueryClient();

  return useMutation<void, ApiServiceErr>({
    mutationFn: logoutApi,
    onSuccess: () => {
      removeToken();
      queryClient.clear(); // Wipe entire cache
    },
    onError: () => {
      // Even if server logout fails, clear local state
      removeToken();
      queryClient.clear();
    },
  });
};
```

## Domain Query Hook Pattern

```typescript
// src/store/services/[domain].ts

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { getItems, getItemById, createItem } from '@/api/http/[domain]';
import { QUERY_KEYS } from '@/constants/keys';
import type { Item, CreateItemDTO } from '@/types/[domain].types';
import type { ApiServiceErr, PagedResponse, PaginationParams } from '@/types/api.types';

// List query with pagination
export const useItemsQuery = (params?: PaginationParams) =>
  useQuery<PagedResponse<Item>, ApiServiceErr>({
    queryKey: [...QUERY_KEYS.ITEMS, params],
    queryFn: () => getItems(params),
    enabled: true,
  });

// Detail query
export const useItemQuery = (id: string) =>
  useQuery<Item, ApiServiceErr>({
    queryKey: QUERY_KEYS.ITEM_DETAIL(id),
    queryFn: () => getItemById(id),
    enabled: !!id,
  });

// Create mutation with cache invalidation
export const useCreateItemMutation = () => {
  const queryClient = useQueryClient();

  return useMutation<Item, ApiServiceErr, CreateItemDTO>({
    mutationFn: createItem,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.ITEMS });
    },
  });
};
```

## Usage in Components

```typescript
// In a screen component:

const ItemListScreen = () => {
  const { styles } = useStyles(createStyles);
  const { data, isLoading, error, refetch } = useItemsQuery();
  const createMutation = useCreateItemMutation();

  const handleCreate = (item: CreateItemDTO) => {
    createMutation.mutate(item, {
      onSuccess: () => {
        // Navigate or show toast
      },
      onError: (error) => {
        toast.show({ type: 'error', message: formatApiError(error) });
      },
    });
  };

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorView onRetry={refetch} />;

  return (
    <FlatList
      data={data?.data}
      renderItem={({ item }) => <ItemCard item={item} />}
      keyExtractor={(item) => item.id}
    />
  );
};
```

## Query Key Convention

```typescript
// src/constants/keys.ts

export const QUERY_KEYS = {
  // Simple keys
  PROFILE: ['profile'] as const,

  // Parameterized keys (for detail views)
  ITEM_DETAIL: (id: string) => ['items', id] as const,

  // Nested keys (for list + filters)
  ITEMS: ['items'] as const,
  // React Query auto-matches: ['items'] matches ['items', { page: 1 }]
} as const;
```

## Provider Stack Integration

```typescript
// In src/app.tsx:

import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client';
import { queryClient, appPersister } from '@/store/services';

const App = () => (
  <PersistQueryClientProvider
    client={queryClient}
    persistOptions={{ persister: appPersister }}
  >
    {/* ...rest of providers */}
  </PersistQueryClientProvider>
);
```

## Best Practices

1. **One file per domain** in `src/store/services/` — keeps hooks organized
2. **Always type query/mutation generics** — `useQuery<Data, Error>()`, `useMutation<Data, Error, Variables>()`
3. **Use `enabled` flag** for conditional queries — don't call hooks conditionally
4. **Invalidate related queries** on mutation success — ensures UI stays fresh
5. **Use `queryClient.clear()`** on logout — prevents data leaks between accounts
6. **Set reasonable `staleTime`** — 5 minutes default, adjust per query
7. **Prefer `select`** for data transformation over transforming in components
8. **Handle loading/error states** in every component that uses queries
