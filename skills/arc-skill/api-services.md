# API Services Architecture

## Layer Overview

```
Component → React Query Hook → API Function → Axios Instance → Server
                                                    ↓
                                          Interceptors (auth, refresh, logging)
```

Components NEVER call API functions directly. They use React Query hooks from `src/store/services/`.

## Axios Instance

```typescript
// src/api/http/index.ts

import axios, { AxiosError, InternalAxiosRequestConfig } from 'axios';
import { getToken, setToken, removeToken } from '@/store/storage/auth';
import { refreshTokenApi } from './auth';

const BASE_URL = process.env.EXPO_PUBLIC_API_URL;

export const httpClient = axios.create({
  baseURL: BASE_URL,
  timeout: 15000,
  headers: {
    'Content-Type': 'application/json',
    // Mobile-specific headers (from mobile-design/mobile-backend.md)
    'X-Platform': Platform.OS,                    // 'ios' | 'android'
    'X-App-Version': Application.nativeApplicationVersion ?? '1.0.0',
  },
});

// Note: import { Platform } from 'react-native';
// Note: import * as Application from 'expo-application';

// --- Request Interceptor: Attach Bearer Token ---
httpClient.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const token = getToken();
    if (token?.accessToken) {
      config.headers.Authorization = `Bearer ${token.accessToken}`;
    }

    if (__DEV__) {
      console.log(`→ ${config.method?.toUpperCase()} ${config.url}`);
    }

    return config;
  },
  (error) => Promise.reject(error),
);

// --- Response Interceptor: Token Refresh on 401 ---
httpClient.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & {
      __retry?: boolean;
    };

    if (__DEV__) {
      console.error(`✗ ${error.response?.status} ${originalRequest?.url}`);
    }

    // Skip refresh for auth endpoints to avoid infinite loops
    const skipRefreshUrls = ['/auth/refresh', '/auth/logout', '/auth/login'];
    const shouldSkip = skipRefreshUrls.some((url) =>
      originalRequest?.url?.includes(url),
    );

    if (
      error.response?.status === 401 &&
      !originalRequest.__retry &&
      !shouldSkip
    ) {
      originalRequest.__retry = true;

      try {
        const token = getToken();
        if (!token?.refreshToken) {
          throw new Error('No refresh token');
        }

        const newToken = await refreshTokenApi(token.refreshToken);
        setToken(newToken);

        originalRequest.headers.Authorization = `Bearer ${newToken.accessToken}`;
        return httpClient(originalRequest);
      } catch (refreshError) {
        // Refresh failed — clear auth and reject
        removeToken();
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  },
);
```

## API Module Pattern

Each domain gets its own file exporting plain async functions:

```typescript
// src/api/http/auth.ts

import { httpClient } from './index';
import type {
  LoginDTO,
  RegisterDTO,
  Token,
  Profile,
} from '@/types/auth.types';

export const loginApi = async (data: LoginDTO): Promise<Token> => {
  const response = await httpClient.post<Token>('/auth/login', data);
  return response.data;
};

export const registerApi = async (data: RegisterDTO): Promise<Token> => {
  const response = await httpClient.post<Token>('/auth/register', data);
  return response.data;
};

export const refreshTokenApi = async (refreshToken: string): Promise<Token> => {
  const response = await httpClient.post<Token>('/auth/refresh', {
    refreshToken,
  });
  return response.data;
};

export const logoutApi = async (): Promise<void> => {
  await httpClient.post('/auth/logout');
};

export const getProfileApi = async (): Promise<Profile> => {
  const response = await httpClient.get<Profile>('/users/me');
  return response.data;
};
```

```typescript
// src/api/http/user.ts

import { httpClient } from './index';
import type { Profile, UpdateProfileDTO } from '@/types/auth.types';

export const getUserProfile = async (): Promise<Profile> => {
  const response = await httpClient.get<Profile>('/users/me');
  return response.data;
};

export const updateUserProfile = async (
  data: UpdateProfileDTO,
): Promise<Profile> => {
  const response = await httpClient.put<Profile>('/users/me', data);
  return response.data;
};
```

## Type Definitions

```typescript
// src/types/api.types.ts

import type { AxiosError } from 'axios';

export interface ApiErrorResponse {
  message: string;
  statusCode: number;
  error?: string;
}

export type ApiServiceErr = AxiosError<ApiErrorResponse>;

export interface PagedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    size: number;
    totalPages: number;
  };
}

// Offset-based (simpler, but duplicates on data changes)
export interface PaginationParams {
  page?: number;
  size?: number;
  search?: string;
}

// Cursor-based (preferred for mobile — no duplicates on infinite scroll)
export interface CursorPaginationParams {
  limit?: number;
  after?: string;  // cursor from previous response
  search?: string;
}

export interface CursorPagedResponse<T> {
  data: T[];
  meta: {
    hasMore: boolean;
    nextCursor: string | null;
    total?: number;
  };
}
```

```typescript
// src/types/auth.types.ts

export interface LoginDTO {
  email: string;
  password: string;
}

export interface RegisterDTO {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
}

export interface Token {
  accessToken: string;
  refreshToken: string;
}

export interface Profile {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  avatarUrl?: string;
  createdAt: string;
}

export interface UpdateProfileDTO {
  firstName?: string;
  lastName?: string;
}
```

## Error Handling Utility

```typescript
// src/helpers/format-api-error.ts

import type { ApiServiceErr } from '@/types/api.types';

export const formatApiError = (error: unknown): string => {
  const axiosError = error as ApiServiceErr;

  if (axiosError?.response?.data?.message) {
    return axiosError.response.data.message;
  }

  if (axiosError?.message) {
    return axiosError.message;
  }

  return 'An unexpected error occurred';
};
```

## Mock Data Support

```typescript
// src/constants/app.ts
export const USE_MOCK_DATA = __DEV__ && false; // Toggle for development

// src/mocks/auth.ts
import type { Token, Profile } from '@/types/auth.types';

export const MOCK_TOKEN: Token = {
  accessToken: 'mock-access-token',
  refreshToken: 'mock-refresh-token',
};

export const MOCK_PROFILE: Profile = {
  id: '1',
  email: 'test@example.com',
  firstName: 'John',
  lastName: 'Doe',
  createdAt: new Date().toISOString(),
};
```

Usage in store services:
```typescript
const loginMutationFn = async (data: LoginDTO) => {
  if (USE_MOCK_DATA) return MOCK_TOKEN;
  return loginApi(data);
};
```

## Environment Variables

```bash
# .env
EXPO_PUBLIC_API_URL=https://api.example.com/v1

# .env.example (committed to git)
EXPO_PUBLIC_API_URL=
```

Access via `process.env.EXPO_PUBLIC_API_URL` — Expo automatically handles the `EXPO_PUBLIC_` prefix.
