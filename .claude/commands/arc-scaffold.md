---
name: arc-scaffold
description: "Scaffold a new React Native Expo project with folder structure, dependencies, TypeScript config, linting, and navigation shell. Use when starting a new mobile app from scratch."
---

# Arc Scaffold — Project Foundation

You are setting up the foundation of a new React Native (Expo) project. This agent handles structure, dependencies, config, and navigation — NOT UI design or API connection.

## Step 1: Gather Requirements

Ask the user for:
- **Project name** (lowercase, used in `app.json` and package name)
- **Brief description** of the app (1-2 sentences)
- **Target platforms**: iOS, Android, or both
- **Navigation pattern**: `tabs + stack` (default), `stack only`, or `drawer + stack`
- **How many main tabs/screens** (just names, e.g. "Home, Search, Profile")

## Step 2: Read Reference Files

Read these from `skills/react-native-arc/`:
1. `project-structure.md` — folder tree, naming, barrel exports
2. `navigation.md` — React Navigation setup, typed hooks
3. `typescript.md` — TSConfig, path aliases, babel config
4. `linting.md` — ESLint, Prettier, Husky, lint-staged
5. `providers.md` — provider stack order
6. `mobile-design/GUIDE.md` — anti-patterns checklist

## Step 3: Initialize Project

```bash
npx create-expo-app@latest <project-name> --template blank-typescript
cd <project-name>
```

## Step 4: Install ALL Dependencies

Always use `npx expo install` for Expo-compatible packages (resolves correct versions automatically). Use latest versions — do NOT pin specific versions.

```bash
# Navigation
npx expo install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs react-native-screens react-native-safe-area-context

# State & Storage
npx expo install @tanstack/react-query @tanstack/react-query-persist-client react-native-mmkv

# HTTP
npx expo install axios

# UI & Animations
npx expo install react-native-reanimated react-native-gesture-handler react-native-size-matters expo-blur expo-haptics expo-status-bar expo-image expo-splash-screen expo-font @expo/vector-icons react-native-keyboard-controller

# Dev tools
npm install -D typescript @types/react eslint prettier eslint-config-expo eslint-plugin-react-hooks eslint-plugin-react-native eslint-config-prettier husky lint-staged babel-plugin-module-resolver
```

### Verify compatibility

```bash
npx expo doctor
```

Fix any version mismatches or warnings before proceeding. If `expo doctor` suggests different versions, follow its recommendations.

## Step 5: Create Folder Structure

Generate the full `src/` tree from `project-structure.md`. Create every directory and empty `index.ts` barrel files. Structure should include:

```
src/
├── api/http/
├── components/
├── constants/
├── helpers/
├── hooks/
├── mocks/
├── navigation/
├── providers/app-initialization/
├── screens/ (with tab groups based on user's answer)
├── store/services/
├── store/storage/
├── theme/
├── types/
├── utils/
├── app.tsx
├── app.styles.ts
└── index.ts
```

## Step 6: Generate Config Files

Using patterns from `typescript.md` and `linting.md`:

1. **`tsconfig.json`** — strict mode + all path aliases (`@/*`, `@components/*`, etc.)
2. **`babel.config.js`** — module-resolver aliases + reanimated plugin (last)
3. **ESLint config** — expo + react-hooks + react-native + prettier
4. **`.prettierrc`** + **`.prettierignore`**
5. **Husky + lint-staged** — `npx husky init`, pre-commit hook
6. **`package.json` scripts** — `lint`, `lint:fix`, `format`, `typecheck`, `check`
7. **`.env.example`** — `EXPO_PUBLIC_API_URL=`
8. **`app.json`** — project name, slug, scheme for deep linking

## Step 7: Generate Foundation Code

1. **`src/constants/`** — `app.ts` (APP_NAME, USE_MOCK_DATA), `keys.ts` (QUERY_KEYS, STORAGE_KEYS), `platform.ts` (isIOS, isAndroid)
2. **`src/types/navigation.types.ts`** — RootStackParamList + TabParamList based on user's screens
3. **`src/hooks/use-app-navigation.ts`** — typed navigation hook
4. **`src/navigation/index.tsx`** — Root navigator with tab + stack structure (placeholder screens)
5. **`src/app.tsx`** — GestureHandler → NavigationContainer → RootNavigator (minimal provider stack, no theme/query yet)
6. **`src/app.styles.ts`** + **`src/index.ts`**

## Step 8: Verify

```bash
npx expo start
```

App should launch with tab navigation and placeholder screens.

## What This Agent Does NOT Do

- Theme system → use `/arc-ui`
- API connection, storage, auth → use `/arc-connect`
- UI components, screens with real content → use `/arc-ui`
- Adding new features/domains → use `/arc-feature`

## Output

Tell the user:
1. What was created (file count, structure overview)
2. Next steps: run `/arc-connect` to add API + storage + state, then `/arc-ui` for design
