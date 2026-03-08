# API Service Templates

## API Module Template

Each domain gets one file in `src/api/http/`:

```typescript
// src/api/http/[domain].ts

import { httpClient } from './index';
import type { [DomainItem], Create[DomainItem]DTO, Update[DomainItem]DTO } from '@/types/[domain].types';
import type { PagedResponse, PaginationParams } from '@/types/api.types';

// --- List (paginated) ---
export const get[DomainItems] = async (
  params?: PaginationParams,
): Promise<PagedResponse<[DomainItem]>> => {
  const response = await httpClient.get<PagedResponse<[DomainItem]>>(
    '/[domain-path]',
    { params },
  );
  return response.data;
};

// --- Detail ---
export const get[DomainItem]ById = async (id: string): Promise<[DomainItem]> => {
  const response = await httpClient.get<[DomainItem]>(`/[domain-path]/${id}`);
  return response.data;
};

// --- Create ---
export const create[DomainItem] = async (
  data: Create[DomainItem]DTO,
): Promise<[DomainItem]> => {
  const response = await httpClient.post<[DomainItem]>('/[domain-path]', data);
  return response.data;
};

// --- Update ---
export const update[DomainItem] = async (
  id: string,
  data: Update[DomainItem]DTO,
): Promise<[DomainItem]> => {
  const response = await httpClient.put<[DomainItem]>(
    `/[domain-path]/${id}`,
    data,
  );
  return response.data;
};

// --- Delete ---
export const delete[DomainItem] = async (id: string): Promise<void> => {
  await httpClient.delete(`/[domain-path]/${id}`);
};
```

## Domain Types Template

```typescript
// src/types/[domain].types.ts

export interface [DomainItem] {
  id: string;
  // domain fields
  createdAt: string;
  updatedAt: string;
}

export interface Create[DomainItem]DTO {
  // fields for creating
}

export interface Update[DomainItem]DTO {
  // partial fields for updating
}
```

## Store Service Template (React Query Hooks)

```typescript
// src/store/services/[domain].ts

import {
  useQuery,
  useMutation,
  useQueryClient,
  useInfiniteQuery,
} from '@tanstack/react-query';
import {
  get[DomainItems],
  get[DomainItem]ById,
  create[DomainItem],
  update[DomainItem],
  delete[DomainItem],
} from '@/api/http/[domain]';
import { QUERY_KEYS } from '@/constants/keys';
import type {
  [DomainItem],
  Create[DomainItem]DTO,
  Update[DomainItem]DTO,
} from '@/types/[domain].types';
import type { ApiServiceErr, PagedResponse, PaginationParams } from '@/types/api.types';

// --- List Query ---
export const use[DomainItems]Query = (params?: PaginationParams) =>
  useQuery<PagedResponse<[DomainItem]>, ApiServiceErr>({
    queryKey: [...QUERY_KEYS.[DOMAIN_ITEMS], params],
    queryFn: () => get[DomainItems](params),
  });

// --- Detail Query ---
export const use[DomainItem]Query = (id: string) =>
  useQuery<[DomainItem], ApiServiceErr>({
    queryKey: QUERY_KEYS.[DOMAIN_ITEM]_DETAIL(id),
    queryFn: () => get[DomainItem]ById(id),
    enabled: !!id,
  });

// --- Infinite List Query (for infinite scroll) ---
export const use[DomainItems]InfiniteQuery = () =>
  useInfiniteQuery<PagedResponse<[DomainItem]>, ApiServiceErr>({
    queryKey: QUERY_KEYS.[DOMAIN_ITEMS],
    queryFn: ({ pageParam = 1 }) =>
      get[DomainItems]({ page: pageParam as number }),
    getNextPageParam: (lastPage) => {
      const { page, totalPages } = lastPage.meta;
      return page < totalPages ? page + 1 : undefined;
    },
    initialPageParam: 1,
  });

// --- Create Mutation ---
export const useCreate[DomainItem]Mutation = () => {
  const queryClient = useQueryClient();

  return useMutation<[DomainItem], ApiServiceErr, Create[DomainItem]DTO>({
    mutationFn: create[DomainItem],
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.[DOMAIN_ITEMS] });
    },
  });
};

// --- Update Mutation ---
export const useUpdate[DomainItem]Mutation = () => {
  const queryClient = useQueryClient();

  return useMutation<
    [DomainItem],
    ApiServiceErr,
    { id: string; data: Update[DomainItem]DTO }
  >({
    mutationFn: ({ id, data }) => update[DomainItem](id, data),
    onSuccess: (updated) => {
      // Update list cache
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.[DOMAIN_ITEMS] });
      // Update detail cache directly
      queryClient.setQueryData(
        QUERY_KEYS.[DOMAIN_ITEM]_DETAIL(updated.id),
        updated,
      );
    },
  });
};

// --- Delete Mutation ---
export const useDelete[DomainItem]Mutation = () => {
  const queryClient = useQueryClient();

  return useMutation<void, ApiServiceErr, string>({
    mutationFn: delete[DomainItem],
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.[DOMAIN_ITEMS] });
    },
  });
};
```

## Query Keys Template

```typescript
// Add to src/constants/keys.ts

export const QUERY_KEYS = {
  // ... existing keys

  [DOMAIN_ITEMS]: ['[domain-items]'] as const,
  [DOMAIN_ITEM]_DETAIL: (id: string) => ['[domain-items]', id] as const,
} as const;
```

## Mock Data Template

```typescript
// src/mocks/[domain].ts

import type { [DomainItem] } from '@/types/[domain].types';

export const MOCK_[DOMAIN_ITEMS]: [DomainItem][] = [
  {
    id: '1',
    // ... mock fields
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
  },
  // ... more items
];
```
