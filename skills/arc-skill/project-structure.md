# Project Structure

## Complete Folder Tree

```
src/
├── api/
│   └── http/
│       ├── index.ts                    # Axios instance + interceptors
│       ├── auth.ts                     # Auth endpoints (login, register, refresh, logout)
│       ├── user.ts                     # User/profile endpoints
│       └── [domain].ts                 # Domain-specific endpoints
│
├── components/
│   ├── app-scroll-view/
│   │   ├── index.tsx                   # Themed ScrollView wrapper
│   │   ├── app-scroll-view.styles.ts
│   │   └── app-scroll-view.types.ts
│   │
│   ├── app-text-input/
│   │   ├── index.tsx                   # Custom TextInput with label, error, icon
│   │   ├── app-text-input.styles.ts
│   │   ├── app-text-input.types.ts
│   │   └── app-text-input.constants.ts
│   │
│   ├── button/
│   │   ├── index.tsx                   # Main button component
│   │   ├── button.styles.ts
│   │   ├── button.types.ts
│   │   ├── button.constants.ts         # Sizes, variants
│   │   ├── button.utils.ts             # (optional) Variant resolvers
│   │   └── components/                 # Sub-components
│   │       ├── index.ts
│   │       ├── button-icon/
│   │       │   └── index.tsx
│   │       └── button-label/
│   │           └── index.tsx
│   │
│   ├── pressable/
│   │   ├── index.tsx                   # Themed Pressable wrapper
│   │   ├── pressable.styles.ts
│   │   └── pressable.types.ts
│   │
│   ├── toast/
│   │   ├── index.tsx                   # Toast barrel export
│   │   ├── toast.types.ts
│   │   ├── context/
│   │   │   └── index.tsx               # ToastContext + useToast hook
│   │   └── components/
│   │       ├── index.ts
│   │       ├── toast-provider/
│   │       │   └── index.tsx           # Toast state manager
│   │       ├── toast-container/
│   │       │   ├── index.tsx           # Animated toast display
│   │       │   └── toast-container.styles.ts
│   │       └── toast-content/
│   │           ├── index.tsx           # Toast message UI
│   │           └── toast-content.styles.ts
│   │
│   ├── skeleton/
│   │   ├── index.tsx                   # Loading skeleton
│   │   ├── skeleton.styles.ts
│   │   └── skeleton.types.ts
│   │
│   └── [component-name]/
│       ├── index.tsx
│       ├── [component-name].styles.ts
│       ├── [component-name].types.ts
│       ├── [component-name].constants.ts
│       ├── [component-name].utils.ts   # (optional)
│       ├── [component-name].hooks.ts   # (optional)
│       └── components/                 # (optional) Sub-components
│           ├── index.ts
│           └── [sub-component]/
│               └── index.tsx
│
├── constants/
│   ├── app.ts                          # APP_NAME, USE_MOCK_DATA, feature flags
│   ├── keys.ts                         # Query keys, storage keys
│   └── platform.ts                     # isIOS, isAndroid, platform utils
│
├── helpers/
│   └── format-api-error.ts             # Error message extraction
│
├── hooks/
│   ├── use-app-navigation.ts           # Typed navigation hook
│   ├── use-app-theme.ts                # Theme access hook
│   └── use-styles.ts                   # Responsive styles hook
│
├── locale/                             # (if i18n enabled)
│   ├── i18n.ts                         # Lingui setup, dynamicActivate
│   ├── i18n-provider.tsx               # I18nProvider wrapper
│   ├── constants.ts                    # Locale definitions
│   └── locales/
│       ├── en/
│       │   └── messages.ts             # English translations
│       └── [lang]/
│           └── messages.ts
│
├── mocks/
│   └── [domain].ts                     # Mock data for development
│
├── navigation/
│   └── index.tsx                       # Root navigator (Stack + Tabs)
│
├── providers/
│   └── app-initialization/
│       ├── index.tsx                   # Barrel export
│       ├── app-initialization.tsx      # Theme init, fonts, splash
│       └── app-initialization.types.ts
│
├── screens/
│   ├── [tab-name]/                     # Grouped by navigation tab
│   │   ├── [screen-name]/
│   │   │   ├── index.ts               # Barrel export
│   │   │   ├── [screen-name].tsx       # Screen component
│   │   │   ├── [screen-name].styles.ts # Screen styles
│   │   │   ├── [screen-name].types.ts  # (optional) Screen-specific types
│   │   │   ├── [screen-name].constants.ts # (optional)
│   │   │   └── components/            # Screen-specific components
│   │   │       ├── index.ts
│   │   │       └── [sub-component]/
│   │   │           ├── index.tsx
│   │   │           └── [sub-component].styles.ts
│   │   └── index.ts                   # Barrel export for tab group
│   │
│   └── onboarding/                    # Onboarding/auth screens
│       ├── login/
│       │   ├── index.ts
│       │   ├── login.tsx
│       │   └── login.styles.ts
│       ├── register/
│       │   ├── index.ts
│       │   ├── register.tsx
│       │   └── register.styles.ts
│       └── index.ts
│
├── store/
│   ├── services/
│   │   ├── index.ts                   # QueryClient, clientStorage, persister
│   │   ├── auth.ts                    # useLoginMutation, useRegisterMutation, etc.
│   │   └── [domain].ts               # Domain query/mutation hooks
│   │
│   └── storage/
│       ├── index.ts                   # MMKV instance + clientStorage wrapper
│       ├── auth.ts                    # Token get/set/remove
│       └── theme.ts                   # Color scheme get/set
│
├── theme/
│   ├── index.ts                       # Theme definitions, colorSet(), all schemes
│   ├── interfaces.ts                  # AppTheme, AppColorSchemeName types
│   └── fonts.ts                       # Font family configuration
│
├── types/
│   ├── api.types.ts                   # ApiErrorResponse, PagedResponse<T>
│   ├── auth.types.ts                  # LoginDTO, RegisterDTO, Token, Profile
│   ├── [domain].types.ts              # Domain-specific types
│   └── navigation.types.ts            # RootStackParamList, TabParamList
│
├── utils/
│   ├── trigger-haptic.ts              # Haptic feedback utility
│   └── [utility].ts                   # Other pure utility functions
│
├── app.tsx                            # Root component with provider stack
├── app.styles.ts                      # Root styles
└── index.ts                           # Entry point (registerRootComponent)
```

## Root-Level Config Files

```
/
├── app.json                           # Expo app config
├── babel.config.js                    # Babel + reanimated + lingui plugins
├── tsconfig.json                      # TypeScript with path aliases
├── .prettierrc                        # Prettier config
├── .eslintrc.js                       # ESLint config
├── .env                               # Environment variables (EXPO_PUBLIC_API_URL)
├── .env.example                       # Template for .env
├── .gitignore
├── package.json
└── lingui.config.ts                   # (if i18n) Lingui configuration
```

## Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Folders | kebab-case | `app-text-input/` |
| Component files | kebab-case + suffix | `button.styles.ts` |
| Component exports | PascalCase | `export const Button` |
| Hook files | kebab-case with `use-` prefix | `use-app-theme.ts` |
| Hook exports | camelCase with `use` prefix | `export const useAppTheme` |
| Type files | kebab-case + `.types.ts` | `auth.types.ts` |
| Type exports | PascalCase | `export type LoginDTO` |
| Constant files | kebab-case + `.constants.ts` | `button.constants.ts` |
| Style files | kebab-case + `.styles.ts` | `button.styles.ts` |
| API files | kebab-case domain name | `auth.ts`, `user.ts` |
| Store service files | kebab-case domain name | `auth.ts`, `cafe.ts` |

## Barrel Export Pattern

Every folder with exported content MUST have an `index.ts` (or `index.tsx` for components):

```typescript
// components/button/index.tsx — component barrel
export { Button } from './button';
export type { ButtonProps } from './button.types';

// screens/profile/index.ts — screen group barrel
export { ProfileScreen } from './profile';
export { AppearanceScreen } from './appearance';
export { LanguageScreen } from './language';
```
