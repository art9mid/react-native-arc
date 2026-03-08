# Linting & Formatting

## Stack

- **ESLint** — Code quality and error catching
- **Prettier** — Code formatting (single source of truth for style)
- **Husky** — Git hooks (run checks before commit)
- **lint-staged** — Run linters only on staged files (fast)

## Install

```bash
npm install -D eslint prettier eslint-config-expo eslint-plugin-react-hooks eslint-plugin-react-native husky lint-staged eslint-config-prettier
```

## ESLint Config

```javascript
// eslint.config.mjs

import { defineConfig } from 'eslint/config';
import expoConfig from 'eslint-config-expo/flat';
import reactHooksPlugin from 'eslint-plugin-react-hooks';
import prettierConfig from 'eslint-config-prettier';

export default defineConfig([
  ...expoConfig,
  prettierConfig,
  {
    plugins: {
      'react-hooks': reactHooksPlugin,
    },
    rules: {
      // --- Errors ---
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'no-unused-vars': 'off', // handled by TS
      '@typescript-eslint/no-unused-vars': [
        'error',
        {
          argsIgnorePattern: '^_',
          varsIgnorePattern: '^_',
        },
      ],
      '@typescript-eslint/no-explicit-any': 'warn',

      // --- React Native specific ---
      'react-native/no-inline-styles': 'warn',
      'react-native/no-unused-styles': 'warn',
      'react-native/no-color-literals': 'warn',
      'react-native/no-raw-text': 'off', // too strict for most projects

      // --- Import order ---
      'import/order': [
        'warn',
        {
          groups: [
            'builtin',
            'external',
            'internal',
            ['parent', 'sibling'],
            'index',
            'type',
          ],
          pathGroups: [
            { pattern: 'react', group: 'builtin', position: 'before' },
            { pattern: 'react-native', group: 'builtin', position: 'before' },
            { pattern: '@/**', group: 'internal' },
          ],
          pathGroupsExcludedImportTypes: ['react', 'react-native'],
          'newlines-between': 'always',
          alphabetize: { order: 'asc', caseInsensitive: true },
        },
      ],
    },
  },
  {
    ignores: [
      'node_modules/',
      '.expo/',
      'dist/',
      'android/',
      'ios/',
      'babel.config.js',
      'metro.config.js',
    ],
  },
]);
```

### Legacy Format (if flat config not supported)

```javascript
// .eslintrc.js

module.exports = {
  extends: [
    'expo',
    'prettier',
  ],
  plugins: ['react-hooks', 'react-native'],
  rules: {
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    '@typescript-eslint/no-unused-vars': [
      'error',
      { argsIgnorePattern: '^_', varsIgnorePattern: '^_' },
    ],
    '@typescript-eslint/no-explicit-any': 'warn',
    'react-native/no-inline-styles': 'warn',
    'react-native/no-unused-styles': 'warn',
    'react-native/no-color-literals': 'warn',
  },
  ignorePatterns: ['node_modules/', '.expo/', 'dist/', 'android/', 'ios/'],
};
```

## Prettier Config

```json
// .prettierrc

{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 90,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "bracketSameLine": false,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

```
// .prettierignore

node_modules/
.expo/
dist/
android/
ios/
coverage/
*.lock
```

## Husky + lint-staged

### Setup

```bash
npx husky init
```

This creates `.husky/` directory with a `pre-commit` hook.

### Pre-commit Hook

```bash
# .husky/pre-commit

npx lint-staged
```

### lint-staged Config

```json
// package.json (add to root)

{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

## Package.json Scripts

```json
{
  "scripts": {
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix",
    "format": "prettier --write 'src/**/*.{ts,tsx,json}'",
    "format:check": "prettier --check 'src/**/*.{ts,tsx,json}'",
    "typecheck": "tsc --noEmit",
    "check": "npm run typecheck && npm run lint && npm run format:check"
  }
}
```

## Import Order Convention

Imports should follow this order (enforced by ESLint `import/order`):

```typescript
// 1. React / React Native (builtins)
import { useState, useCallback } from 'react';
import { View, Text, FlatList } from 'react-native';

// 2. External libraries
import { useQuery } from '@tanstack/react-query';
import { moderateScale } from 'react-native-size-matters';

// 3. Internal aliases (@/)
import { useStyles } from '@/hooks/use-styles';
import { useAppNavigation } from '@/hooks/use-app-navigation';
import { Button } from '@/components/button';

// 4. Parent / sibling
import { createStyles } from './screen-name.styles';
import { SOME_CONSTANT } from './screen-name.constants';

// 5. Types (always last)
import type { ScreenNameProps } from './screen-name.types';
import type { AppTheme } from '@/theme/interfaces';
```

## Rules Rationale

| Rule | Why |
|------|-----|
| `no-console: warn` | `console.log` blocks JS thread in production — remove before shipping |
| `no-inline-styles: warn` | Inline styles create new objects every render — use `useStyles()` |
| `no-color-literals: warn` | Colors should come from theme — not hardcoded |
| `no-unused-styles: warn` | Dead styles add bundle size |
| `exhaustive-deps: warn` | Missing deps cause stale closures and bugs |
| `no-explicit-any: warn` | Use `unknown` and narrow — `any` disables type safety |
| `argsIgnorePattern: ^_` | Allows unused params prefixed with `_` (common in callbacks) |

## CI Integration

Add to your CI pipeline (GitHub Actions example):

```yaml
# .github/workflows/lint.yml

name: Lint & Typecheck
on: [push, pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run typecheck
      - run: npm run lint
      - run: npm run format:check
```
