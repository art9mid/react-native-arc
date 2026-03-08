# React Native Arc — GitHub Copilot Instructions

You are working with a React Native (Expo) architecture reference project.

## Architecture Reference

All architecture documentation is in `skills/react-native-arc/`. Read these files BEFORE writing code:

- `project-structure.md` — Folder tree, naming conventions, barrel exports
- `navigation.md` — React Navigation (native-stack + bottom-tabs)
- `theme.md` — Theme system, color schemes, useStyles hook
- `components.md` — Component file structure (types, styles, constants in separate files)
- `api-services.md` — Axios HTTP client with auth interceptors
- `storage.md` — MMKV storage wrappers
- `state-management.md` — TanStack React Query + persistence
- `performance.md` — React Native performance best practices
- `providers.md` — Provider stack
- `typescript.md` — TSConfig, path aliases, type conventions
- `i18n.md` — Lingui.js internationalization

## Mobile Design System

Read `skills/react-native-arc/mobile-design/` for touch psychology, platform conventions, performance, typography, colors, navigation, and backend patterns.

## Code Templates

Use templates from `skills/react-native-arc/templates/` for components, screens, hooks, and API services.

## Critical Rules

1. Component structure: `index.tsx` + `name.styles.ts` + `name.types.ts` + `name.constants.ts`
2. All styles via `useStyles()` hook — never inline `StyleSheet.create`
3. All sizing via `react-native-size-matters` `moderateScale` — never hardcode pixels
4. All API data via React Query hooks — never call API directly in components
5. All storage via typed MMKV wrappers — never access MMKV directly
6. Navigation typed via `useAppNavigation()` hook
7. Barrel exports (`index.ts`) in every folder
8. kebab-case folders, PascalCase exports
9. Touch targets minimum 44pt (iOS) / 48dp (Android)
10. `FlatList` with `React.memo` items, `useCallback` renderItem — never `ScrollView` for lists
11. `react-native-reanimated` for animations (UI thread)
12. `expo-image` instead of React Native `Image`
13. Platform-respectful: iOS follows HIG, Android follows Material Design 3
