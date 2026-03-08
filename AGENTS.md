# React Native Arc — Agent Instructions

You are working with a React Native (Expo) architecture reference. Before generating any code, read the relevant skill files.

## Agent Commands

The workflow is split into focused agents that can be run independently:

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/arc-scaffold` | Project init, folder structure, deps, config, linting, navigation | Starting a new project from scratch |
| `/arc-connect` | API client, storage, state management, auth, i18n | After scaffold — adding the data layer |
| `/arc-ui` | Theme system, components, screen content, mobile UX | After scaffold — building the visual layer |
| `/arc-feature` | Add a complete new domain (types + API + hooks + screens) | Adding a new entity like products or orders |
| `/arc-audit` | Check for UX, performance, and architecture violations | Reviewing existing code quality |
| `/arc-skill` | Full scaffolding (all-in-one) | When you want everything in a single pass |

**Typical workflow:**
```
/arc-scaffold → /arc-connect → /arc-ui → /arc-feature (repeat per domain) → /arc-audit
```

## Architecture Reference

All documentation lives in `skills/arc-skill/`. Read before writing code:

| File | What It Covers |
|------|---------------|
| `project-structure.md` | Folder tree, naming conventions, barrel exports |
| `navigation.md` | React Navigation, typed hooks, deep linking, tab UX |
| `theme.md` | Color schemes, useStyles, useAppTheme, OLED dark mode |
| `components.md` | Component file structure, sub-components, touch targets, haptics |
| `api-services.md` | Axios interceptors, API modules, cursor pagination, error handling |
| `storage.md` | MMKV wrappers, typed access, React Query persistence |
| `state-management.md` | React Query setup, query/mutation hook patterns |
| `performance.md` | FlatList, images, animations, navigation, startup optimization |
| `providers.md` | Provider stack order, AppInitialization, root component |
| `typescript.md` | TSConfig, path aliases, type conventions |
| `linting.md` | ESLint, Prettier, Husky, lint-staged, import order |
| `i18n.md` | Lingui.js setup, language switching, translations |

## Mobile Design System

Read `skills/arc-skill/mobile-design/` for platform-specific guidelines:

| File | What It Covers |
|------|---------------|
| `GUIDE.md` | Master checklist, anti-patterns, mandatory pre-coding checkpoint |
| `touch-psychology.md` | Fitts' Law, 44pt/48dp touch targets, thumb zones, haptics |
| `mobile-performance.md` | FlatList tuning, animation budget (16ms), memory leak prevention |
| `mobile-navigation.md` | Tab bar limits, state preservation, back handling, deep linking |
| `mobile-typography.md` | SF Pro / Roboto, Dynamic Type, type scales, min sizes |
| `mobile-color-system.md` | OLED optimization, dark mode strategy, WCAG contrast |
| `platform-ios.md` | iOS HIG, SF Symbols, sheets, system colors |
| `platform-android.md` | Material Design 3, Material Symbols, elevation, FAB |
| `mobile-backend.md` | Push notifications, offline sync, cursor pagination, auth flow |
| `decision-trees.md` | Framework, state management, storage, auth selection |
| `mobile-testing.md` | Testing pyramid, Jest, RNTL, Detox, Maestro |
| `mobile-debugging.md` | Reactotron, Flipper, profiling, crash analysis |

## Code Templates

Use `skills/arc-skill/templates/` when generating new code:

| Template | Generates |
|----------|-----------|
| `component.md` | Component with types, styles, constants, sub-components |
| `screen.md` | Screen with data fetching, forms, navigation |
| `hook.md` | Custom hooks (useStyles, useAppTheme, useDebounce, etc.) |
| `api-service.md` | API module + React Query hooks + query keys |

## Critical Rules

1. **Component structure**: `index.tsx` + `name.styles.ts` + `name.types.ts` + `name.constants.ts` (separate files)
2. **Styles**: Always via `useStyles()` hook — never inline `StyleSheet.create`
3. **Sizing**: Always via `moderateScale` from `react-native-size-matters` — never hardcode pixels
4. **Data fetching**: Always via React Query hooks — never call API functions directly in components
5. **Storage**: Always via typed MMKV wrappers — never access MMKV directly
6. **Navigation**: Always typed via `useAppNavigation()` hook
7. **Exports**: Barrel exports (`index.ts`) in every folder
8. **Naming**: kebab-case folders, PascalCase exports
9. **Touch targets**: Minimum 44pt (iOS) / 48dp (Android), 8px spacing between targets
10. **Lists**: `FlatList` — never `ScrollView` for lists
11. **Animations**: `react-native-reanimated` (UI thread) — never `Animated` API for complex animations
12. **Images**: `expo-image` with blurhash + disk cache — never React Native `Image`
13. **Platform**: iOS follows HIG, Android follows Material Design 3 — respect both
14. **Dark mode**: `#121212` surfaces (not pure black), `#E8E8E8` text (not pure white), WCAG AA contrast
15. **Cleanup**: Always clean up timers, listeners, subscriptions in `useEffect` return
16. **React Compiler**: Enabled via `app.json` (`expo.experiments.reactCompiler: true`) — do NOT manually add `React.memo`, `useCallback`, or `useMemo`
17. **Dependencies**: Always install latest versions via `npx expo install` (not `npm install` for Expo-compatible packages), then run `npx expo doctor` to verify compatibility
18. **Expo Skills**: Install official Expo skills for AI-assisted development — `expo-app-design` (native UI, data fetching, Tailwind, SwiftUI, Jetpack Compose), `expo-deployment` (app stores, CI/CD), `upgrading-expo` (SDK upgrades)
