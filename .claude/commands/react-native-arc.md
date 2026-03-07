# React Native Arc — Project Scaffolding Skill

You are scaffolding a new React Native (Expo) project using the **react-native-arc** architecture.

## Pre-Flight

1. **Ask the user** for:
   - Project name (used in `app.json`, package name, display name)
   - Brief description of the app (to inform screen/feature choices)
   - Target platforms: iOS only, Android only, or both
   - Whether they need i18n (internationalization) — default: yes
   - Whether they need authentication flow — default: yes
   - Which navigation pattern: `tabs + stack` (default) or `stack only` or `drawer + stack`

2. **Context7 (optional):** If the user has Context7 MCP configured, use it to fetch the latest docs for:
   - `react-native` — core APIs and New Architecture
   - `expo` — SDK setup and latest modules
   - `@react-navigation/native` — navigation
   - `@tanstack/react-query` — server state
   - `react-native-mmkv` — storage
   - `react-native-reanimated` — animations

   If Context7 is not available, proceed with the architecture reference files.

3. **Mobile Design Checkpoint (mandatory):**
   Before writing any code, read `skills/react-native-arc/mobile-design/GUIDE.md` and complete:
   ```
   Platform:   [iOS / Android / Both]
   Framework:  React Native (Expo)

   3 Principles I Will Apply:
   1. Touch-first (44pt+ targets, thumb zone aware)
   2. Platform-respectful (iOS HIG / Material Design 3)
   3. Performance-first (FlatList, Reanimated, memo)

   Anti-Patterns I Will Avoid:
   1. ScrollView for lists
   2. Hardcoded pixel values
   3. Inline styles / anonymous functions in render
   ```

## Architecture Reference Files

Read the following skill files from `skills/react-native-arc/` to understand the full architecture before generating code:

**Core Architecture:**
1. **`project-structure.md`** — Complete folder structure
2. **`navigation.md`** — React Navigation patterns
3. **`theme.md`** — Theme system with color schemes
4. **`components.md`** — Component file structure & patterns
5. **`api-services.md`** — Axios HTTP client & API layer
6. **`storage.md`** — MMKV storage wrapper
7. **`state-management.md`** — React Query setup & patterns
8. **`performance.md`** — React Native performance best practices
9. **`providers.md`** — Provider stack architecture
10. **`typescript.md`** — TypeScript conventions
11. **`linting.md`** — ESLint, Prettier, Husky, lint-staged
12. **`i18n.md`** — Internationalization with Lingui

**Mobile Design System (read selectively based on needs):**
12. **`mobile-design/GUIDE.md`** — Master checklist & anti-patterns
13. **`mobile-design/touch-psychology.md`** — Touch targets, thumb zones, haptics
14. **`mobile-design/mobile-performance.md`** — Deep performance optimization
15. **`mobile-design/mobile-navigation.md`** — Navigation UX patterns
16. **`mobile-design/mobile-typography.md`** — Type scales, Dynamic Type
17. **`mobile-design/mobile-color-system.md`** — OLED, dark mode, contrast
18. **`mobile-design/platform-ios.md`** — iOS HIG specifics
19. **`mobile-design/platform-android.md`** — Material Design 3 specifics
20. **`mobile-design/mobile-backend.md`** — Offline sync, push, auth patterns
21. **`mobile-design/decision-trees.md`** — Framework & architecture decisions

## Template Files

Read templates from `skills/react-native-arc/templates/` for code generation patterns:
- **`component.md`** — Component file templates
- **`screen.md`** — Screen file templates
- **`hook.md`** — Custom hook templates
- **`api-service.md`** — API service module templates

## Execution Steps

### Step 1: Initialize Expo Project
```bash
npx create-expo-app@latest <project-name> --template blank-typescript
cd <project-name>
```

### Step 2: Install Core Dependencies
Install in this order (groups can be parallel):

**Navigation:**
```bash
npx expo install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs react-native-screens react-native-safe-area-context
```

**State & Storage:**
```bash
npx expo install @tanstack/react-query @tanstack/react-query-persist-client react-native-mmkv
```

**HTTP:**
```bash
npx expo install axios
```

**UI & Animations:**
```bash
npx expo install react-native-reanimated react-native-gesture-handler react-native-size-matters expo-blur expo-haptics expo-status-bar expo-image expo-splash-screen expo-font @expo/vector-icons react-native-keyboard-controller
```

**Forms (if auth flow):**
```bash
npx expo install formik yup
```

**i18n (if enabled):**
```bash
npm install @lingui/react @lingui/core
npm install -D @lingui/cli @lingui/macro @lingui/babel-plugin-lingui-macro
npx expo install expo-localization
```

**Dev tools:**
```bash
npm install -D typescript @types/react eslint prettier eslint-config-expo husky lint-staged babel-plugin-module-resolver
```

### Step 3: Generate Project Structure
Create the full `src/` directory structure as defined in `project-structure.md`.

### Step 4: Generate Core Files
Generate files in this order (respecting dependencies):

1. **Types** (`src/types/`) — Base type definitions
2. **Constants** (`src/constants/`) — App constants, keys, platform utils
3. **Theme** (`src/theme/`) — Theme definitions, interfaces, fonts
4. **Storage** (`src/store/storage/`) — MMKV storage wrappers
5. **API Client** (`src/api/http/index.ts`) — Axios instance with interceptors
6. **API Modules** (`src/api/http/`) — Auth, profile, etc.
7. **Store Services** (`src/store/services/`) — React Query client, hooks
8. **Hooks** (`src/hooks/`) — useAppTheme, useAppNavigation, useStyles
9. **Providers** (`src/providers/`) — AppInitialization, i18n
10. **Components** (`src/components/`) — Shared UI components
11. **Screens** (`src/screens/`) — App screens
12. **Navigation** (`src/navigation/`) — Navigator setup
13. **App Entry** (`src/app.tsx`) — Root component with provider stack
14. **i18n** (`src/locale/`) — If i18n enabled

### Step 5: Configure Build Tools
- Update `tsconfig.json` with path aliases (see `typescript.md`)
- Update `babel.config.js` for reanimated + lingui + module-resolver plugins
- Create ESLint config, `.prettierrc`, `.prettierignore` (see `linting.md`)
- Set up Husky + lint-staged for pre-commit hooks (see `linting.md`)
- Add `lint`, `format`, `typecheck`, `check` scripts to `package.json`
- Create `.env.example`
- Update `app.json` with project metadata

### Step 6: Verify
```bash
npx expo start
```

## Important Rules

**Architecture:**
- Always use the component file structure from `components.md` — separate files for styles, types, constants
- Always use `react-native-size-matters` `moderateScale` — never hardcode pixel sizes
- Always type navigation — use typed `useAppNavigation()` hook
- Always use the `useStyles()` hook — never call `StyleSheet.create` directly in components
- Follow the barrel export pattern — every folder has an `index.ts`
- API calls go through React Query — never call API functions directly from components
- Storage access through typed wrappers — never use MMKV directly in components

**Mobile UX (from mobile-design system):**
- Touch targets minimum 44pt (iOS) / 48dp (Android) — see `touch-psychology.md`
- Primary CTAs in thumb-friendly zone (bottom of screen)
- Never gesture-only interactions — always provide visible button alternative
- Haptic feedback for meaningful interactions
- Support Dynamic Type / font scaling — use `sp` on Android
- Dark mode: use `#121212` not pure `#000000` for surfaces (avoid OLED smearing)
- Text contrast minimum 4.5:1 (WCAG AA)
- Platform-respectful: iOS follows HIG, Android follows Material Design 3

**Performance (from performance.md + mobile-performance.md):**
- FlatList with `React.memo` items, `useCallback` renderItem — never ScrollView for lists
- Reanimated for animations (UI thread) — never Animated API for complex animations
- `expo-image` instead of React Native `Image` — with caching and blurhash
- `freezeOnBlur` on tab screens
- Clean up timers, listeners, subscriptions in useEffect cleanup
- Remove all `console.log` before production
