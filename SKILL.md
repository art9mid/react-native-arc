---
name: arc-skill
description: Scaffold production-ready React Native Expo projects with battle-tested architecture including navigation, theme system, API layer, storage, state management, i18n, linting, and mobile UX design system. Use when creating a new React Native app, setting up Expo project architecture, or when the user mentions React Native project scaffolding, mobile app boilerplate, or Expo starter template.
metadata:
  author: art9mid
  version: "1.0"
---

# React Native Arc

Scaffold a complete React Native (Expo) project with production-ready architecture.

## When to Use

- User wants to create a new React Native / Expo project
- User asks for mobile app boilerplate or starter template
- User needs project architecture guidance for React Native
- User mentions scaffolding, project setup, or mobile app structure

## Pre-Flight

Ask the user for:
1. **Project name** (used in `app.json`, package name)
2. **Brief description** of the app
3. **Target platforms**: iOS only, Android only, or both
4. **i18n needed?** (default: yes)
5. **Auth flow needed?** (default: yes)
6. **Navigation pattern**: `tabs + stack` (default), `stack only`, or `drawer + stack`

## Mobile Design Checkpoint (mandatory)

Before writing any code, complete:
```
Platform:   [iOS / Android / Both]
Framework:  React Native (Expo)

3 Principles:
1. Touch-first (44pt+ targets, thumb zone aware)
2. Platform-respectful (iOS HIG / Material Design 3)
3. Performance-first (FlatList, Reanimated, memo)

Anti-Patterns to Avoid:
1. ScrollView for lists
2. Hardcoded pixel values
3. Inline styles / anonymous functions in render
```

## Architecture Reference Files

Read from `skills/arc-skill/` before generating code:

**Core Architecture:**
1. `project-structure.md` — Folder tree, naming conventions, barrel exports
2. `navigation.md` — React Navigation, typed hooks, deep linking, tab UX
3. `theme.md` — Theme system, color schemes, OLED dark mode, useStyles hook
4. `components.md` — Component file structure, touch targets, haptics, expo-image
5. `api-services.md` — Axios HTTP client, interceptors, cursor pagination
6. `storage.md` — MMKV wrappers, typed access, React Query persistence
7. `state-management.md` — React Query setup, query/mutation patterns
8. `performance.md` — FlatList, images, animations, startup optimization
9. `providers.md` — Provider stack order, AppInitialization
10. `typescript.md` — TSConfig, path aliases, type conventions
11. `linting.md` — ESLint, Prettier, Husky, lint-staged, import order
12. `i18n.md` — Lingui.js internationalization

**Mobile Design System** (`skills/arc-skill/mobile-design/`):
- `GUIDE.md` — Master checklist and anti-patterns
- `touch-psychology.md` — Fitts' Law, thumb zones, haptics
- `mobile-performance.md` — Deep optimization (16ms frame budget)
- `mobile-navigation.md` — Tab bar UX, state preservation, back handling
- `mobile-typography.md` — SF Pro / Roboto, Dynamic Type, type scales
- `mobile-color-system.md` — OLED, dark mode, WCAG contrast
- `platform-ios.md` — iOS Human Interface Guidelines
- `platform-android.md` — Material Design 3
- `mobile-backend.md` — Offline sync, push notifications, auth patterns
- `decision-trees.md` — Framework, state, storage decisions
- `mobile-testing.md` — Testing pyramid (Jest, RNTL, Detox, Maestro)
- `mobile-debugging.md` — Reactotron, Flipper, profiling

**Code Templates** (`skills/arc-skill/templates/`):
- `component.md` — Component with types, styles, constants, sub-components
- `screen.md` — Screen with data fetching, forms, navigation
- `hook.md` — Custom hooks (useStyles, useAppTheme, useDebounce)
- `api-service.md` — API module + React Query hooks

## Execution Steps

### 1. Initialize
```bash
npx create-expo-app@latest <project-name> --template blank-typescript
cd <project-name>
```

### 2. Install Dependencies
```bash
# Navigation
npx expo install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs react-native-screens react-native-safe-area-context

# State & Storage
npx expo install @tanstack/react-query @tanstack/react-query-persist-client react-native-mmkv

# HTTP
npx expo install axios

# UI & Animations
npx expo install react-native-reanimated react-native-gesture-handler react-native-size-matters expo-blur expo-haptics expo-status-bar expo-image expo-splash-screen expo-font @expo/vector-icons react-native-keyboard-controller

# Forms (if auth)
npx expo install formik yup

# i18n (if enabled)
npm install @lingui/react @lingui/core
npm install -D @lingui/cli @lingui/macro @lingui/babel-plugin-lingui-macro
npx expo install expo-localization

# Dev tools
npm install -D typescript @types/react eslint prettier eslint-config-expo eslint-plugin-react-hooks eslint-plugin-react-native eslint-config-prettier husky lint-staged babel-plugin-module-resolver
```

### 3. Generate `src/` Structure
Follow `project-structure.md` for the complete folder tree.

### 4. Generate Core Files (in dependency order)
1. Types → 2. Constants → 3. Theme → 4. Storage → 5. API Client → 6. API Modules → 7. Store Services → 8. Hooks → 9. Providers → 10. Components → 11. Screens → 12. Navigation → 13. App Entry → 14. i18n

### 5. Configure Tooling
- `tsconfig.json` with path aliases (see `typescript.md`)
- `babel.config.js` with reanimated + lingui + module-resolver
- ESLint + Prettier + Husky + lint-staged (see `linting.md`)
- `.env.example`, `app.json`

### 6. Verify
```bash
npx expo start
```

## Component Convention

```
component-name/
├── index.tsx                    # Component + barrel export
├── component-name.styles.ts     # Styles via useStyles callback
├── component-name.types.ts      # Props interface
├── component-name.constants.ts  # (optional) Variants, sizes
├── component-name.utils.ts      # (optional) Pure helpers
├── component-name.hooks.ts      # (optional) Component hooks
└── components/                  # (optional) Sub-components
    ├── index.ts
    └── sub-component/
        └── index.tsx
```

## Critical Rules

- Styles via `useStyles()` — never inline `StyleSheet.create`
- Sizing via `moderateScale` — never hardcode pixels
- API data via React Query — never call API directly in components
- Storage via typed MMKV wrappers — never raw access
- Navigation via typed `useAppNavigation()` hook
- Touch targets min 44pt (iOS) / 48dp (Android)
- `FlatList` with `React.memo` items — never `ScrollView` for lists
- `expo-image` — never React Native `Image`
- Reanimated for animations — never `Animated` API for complex motion
- Barrel exports (`index.ts`) in every folder
- kebab-case folders, PascalCase exports
- Platform-respectful: iOS feels iOS, Android feels Android
- Dark mode: `#121212` surfaces, `#E8E8E8` text, WCAG AA contrast
- No `console.log` in production — blocks JS thread
