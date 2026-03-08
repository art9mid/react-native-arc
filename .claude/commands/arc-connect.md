---
name: arc-connect
description: "Connect API layer, storage, state management, authentication, and i18n to an existing React Native Expo project. Use after arc-scaffold to add backend integration."
---

# Arc Connect — API, Storage & State

You are adding the data layer to an existing React Native (Expo) project that was scaffolded with `/arc-scaffold`. This agent handles API client, storage, state management, auth flow, and optionally i18n — NOT UI components or visual design.

## Step 1: Gather Requirements

Ask the user for:
- **API base URL** (or leave as env variable placeholder)
- **Auth flow needed?** — email/password, social login (Google/Apple), or none
- **Key data domains** — e.g. "users, products, orders" (determines API modules + types)
- **i18n needed?** — if yes, which languages (default: English)
- **Offline support?** — basic cache (default) or full offline-first

## Step 2: Read Reference Files

Read these from `skills/arc-skill/`:
1. `api-services.md` — Axios setup, interceptors, API modules
2. `storage.md` — MMKV wrappers, typed access
3. `state-management.md` — React Query client, query/mutation hooks
4. `providers.md` — provider stack with PersistQueryClientProvider
5. `i18n.md` — Lingui setup (if needed)
6. `mobile-design/mobile-backend.md` — offline sync, auth patterns, cursor pagination

Read templates:
7. `templates/api-service.md` — API module + React Query hooks template
8. `templates/hook.md` — hook patterns

## Step 3: Install Additional Dependencies (if needed)

```bash
# Auth forms (if auth flow)
npx expo install formik yup

# i18n (if enabled)
npm install @lingui/react @lingui/core
npm install -D @lingui/cli @lingui/macro @lingui/babel-plugin-lingui-macro
npx expo install expo-localization
```

Update `babel.config.js` to add `@lingui/babel-plugin-lingui-macro` if i18n enabled.

## Step 4: Generate Storage Layer

1. **`src/store/storage/index.ts`** — MMKV instance + `clientStorage` typed wrapper
2. **`src/store/storage/auth.ts`** — `getToken()`, `setToken()`, `removeToken()`, `isAuthenticated()`
3. **`src/store/storage/theme.ts`** — `getStoredColorScheme()`, `setStoredColorScheme()`
4. **`src/store/storage/onboarding.ts`** — `hasCompletedOnboarding()`, `setOnboardingCompleted()`

## Step 5: Generate Type Definitions

1. **`src/types/api.types.ts`** — `ApiErrorResponse`, `ApiServiceErr`, `PagedResponse<T>`, `CursorPagedResponse<T>`, `PaginationParams`
2. **`src/types/auth.types.ts`** — `LoginDTO`, `RegisterDTO`, `Token`, `Profile`, `UpdateProfileDTO`
3. **`src/types/[domain].types.ts`** — For each domain the user listed (e.g. `product.types.ts`)

## Step 6: Generate API Client

1. **`src/api/http/index.ts`** — Axios instance with:
   - Request interceptor (attach Bearer token, mobile headers)
   - Response interceptor (401 → refresh token → retry, logging in __DEV__)
2. **`src/api/http/auth.ts`** — `loginApi`, `registerApi`, `refreshTokenApi`, `logoutApi`, `getProfileApi`
3. **`src/api/http/[domain].ts`** — For each domain: `getItems`, `getItemById`, `createItem`, `updateItem`, `deleteItem`

## Step 7: Generate State Management

1. **`src/store/services/index.ts`** — `QueryClient` (with defaults) + `appPersister` (MMKV-backed)
2. **`src/store/services/auth.ts`** — `useProfileQuery`, `useLoginMutation`, `useRegisterMutation`, `useLogoutMutation`
3. **`src/store/services/[domain].ts`** — For each domain: list query, detail query, create/update/delete mutations
4. **Update `src/constants/keys.ts`** — Add QUERY_KEYS for each domain

## Step 8: Generate Helpers

1. **`src/helpers/format-api-error.ts`** — extract user-friendly error message from AxiosError
2. **`src/utils/trigger-haptic.ts`** — haptic feedback utility

## Step 9: Generate Mock Data (for development)

1. **`src/mocks/auth.ts`** — MOCK_TOKEN, MOCK_PROFILE
2. **`src/mocks/[domain].ts`** — Mock data for each domain

## Step 10: Generate i18n (if enabled)

1. **`lingui.config.ts`** — Lingui configuration
2. **`src/locale/constants.ts`** — supported locales, default locale
3. **`src/locale/i18n.ts`** — setup, `dynamicActivate`, `initI18n`
4. **`src/locale/i18n-provider.tsx`** — React provider wrapper
5. **`src/locale/locales/en/messages.ts`** — empty English catalog
6. Add `intl:extract`, `intl:extract:all`, `intl:compile` scripts to `package.json`

## Step 11: Update Provider Stack

Update `src/app.tsx` to include full provider chain:

```
GestureHandlerRootView
  └─ KeyboardProvider
     └─ PersistQueryClientProvider (queryClient + appPersister)
        └─ AppInitializationProvider (will be created by /arc-ui)
           └─ SafeAreaProvider
              └─ I18nProvider (if enabled)
                 └─ NavigationContainer
                    └─ ToastProvider (will be created by /arc-ui)
                       └─ RootNavigator
```

For now, skip AppInitializationProvider and ToastProvider if `/arc-ui` hasn't been run yet — wrap with comments marking where they go.

## Step 12: Update Navigation (if auth flow)

Update `src/navigation/index.tsx` to add conditional auth screens:
- If not authenticated → show Login, Register, ForgotPassword
- If authenticated → show MainTabs + stack screens

## What This Agent Does NOT Do

- Project initialization, folder structure → use `/arc-scaffold`
- Theme, components, screen UI → use `/arc-ui`
- Adding new features after initial setup → use `/arc-feature`

## Output

Tell the user:
1. What was created (API modules, storage keys, query hooks)
2. How to test: set `USE_MOCK_DATA = true` in `src/constants/app.ts`
3. Next steps: run `/arc-ui` to add theme + components + screen content
