# TypeScript Conventions

## TSConfig

```json
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@screens/*": ["src/screens/*"],
      "@hooks/*": ["src/hooks/*"],
      "@theme/*": ["src/theme/*"],
      "@api/*": ["src/api/*"],
      "@store/*": ["src/store/*"],
      "@types/*": ["src/types/*"],
      "@constants/*": ["src/constants/*"],
      "@utils/*": ["src/utils/*"],
      "@providers/*": ["src/providers/*"],
      "@locale/*": ["src/locale/*"],
      "@navigation/*": ["src/navigation/*"],
      "@helpers/*": ["src/helpers/*"],
      "@mocks/*": ["src/mocks/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx"]
}
```

## Babel Path Aliases

```javascript
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      [
        'module-resolver',
        {
          alias: {
            '@': './src',
            '@components': './src/components',
            '@screens': './src/screens',
            '@hooks': './src/hooks',
            '@theme': './src/theme',
            '@api': './src/api',
            '@store': './src/store',
            '@types': './src/types',
            '@constants': './src/constants',
            '@utils': './src/utils',
            '@providers': './src/providers',
            '@locale': './src/locale',
            '@navigation': './src/navigation',
            '@helpers': './src/helpers',
            '@mocks': './src/mocks',
          },
        },
      ],
      'react-native-reanimated/plugin', // Must be LAST
    ],
  };
};
```

Install: `npm install -D babel-plugin-module-resolver`

## React Compiler

Enabled via `app.json` — Expo handles the babel plugin automatically:

```json
{
  "expo": {
    "experiments": {
      "reactCompiler": true
    }
  }
}
```

Verify project compatibility: `npx react-compiler-healthcheck@latest`

## Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Types/Interfaces | PascalCase | `LoginDTO`, `AppTheme` |
| Type files | kebab-case + `.types.ts` | `auth.types.ts` |
| Enums | PascalCase | `ButtonVariant` |
| Constants | SCREAMING_SNAKE_CASE | `STORAGE_KEYS`, `QUERY_KEYS` |
| Functions | camelCase | `formatApiError()` |
| Hooks | camelCase with `use` | `useAppTheme()` |
| Components | PascalCase | `Button`, `AppTextInput` |
| Props interfaces | PascalCase + `Props` | `ButtonProps` |
| Context values | PascalCase + `ContextValue` | `AppInitializationContextValue` |

## Type Patterns

### Props Interface
```typescript
// Always export props from .types.ts
export interface ComponentNameProps {
  title: string;
  subtitle?: string;         // Optional props use ?
  onPress: () => void;
  variant?: 'primary' | 'secondary';  // Union types for variants
  children?: React.ReactNode;
}
```

### API Response Types
```typescript
// Generic paginated response
export interface PagedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    size: number;
  };
}

// Error type
export type ApiServiceErr = AxiosError<ApiErrorResponse>;
```

### Navigation Types
```typescript
// Param list type
export type RootStackParamList = {
  Home: undefined;                    // No params
  ItemDetail: { itemId: string };     // Required params
  Search: { query?: string };         // Optional params
};
```

### Style Callback Type
```typescript
// Style callback — always this signature
type StyleCallback<T extends StyleSheet.NamedStyles<T>> = (
  theme: AppTheme,
  insets: EdgeInsets,
) => T;
```

### Generic Storage
```typescript
// Typed get/set
clientStorage.getItem<Token>(STORAGE_KEYS.TOKENS);
clientStorage.setItem<Token>(STORAGE_KEYS.TOKENS, token);
```

### React Query Typed Hooks
```typescript
// Always specify all generics
useQuery<ResponseType, ErrorType>({...});
useMutation<ResponseType, ErrorType, VariablesType>({...});
```

## Rules

1. **Strict mode always** — `strict: true` in tsconfig
2. **No `any`** — use `unknown` for truly unknown types, then narrow
3. **No type assertions** (`as Type`) unless absolutely necessary (prefer type guards)
4. **Export types from `.types.ts` files** — not from component files
5. **Use `const assertions`** for query keys: `['key'] as const`
6. **Prefer `interface` over `type`** for object shapes (better error messages, extendable)
7. **Use path aliases** — `@/hooks/use-app-theme` not `../../hooks/use-app-theme`
8. **Function return types** — let TypeScript infer them (explicit only when complex/public API)
9. **Discriminated unions** for state machines:
   ```typescript
   type Status =
     | { type: 'idle' }
     | { type: 'loading' }
     | { type: 'success'; data: Item[] }
     | { type: 'error'; message: string };
   ```
