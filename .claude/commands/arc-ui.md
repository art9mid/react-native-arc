---
name: arc-ui
description: "Add theme system, reusable components, screen layouts, and mobile UX design to a React Native Expo project. Use after arc-scaffold to build the visual layer."
---

# Arc UI — Theme, Components & Screens

You are building the visual layer of a React Native (Expo) project. This agent handles theme system, reusable components, screen content, and mobile UX — NOT API connection or state management.

## Step 1: Gather Requirements

Ask the user for:
- **Brand color** (primary hex, e.g. `#007AFF`) — or "default blue"
- **Font family** — custom font name or "system default" (Inter, SF Pro, Roboto)
- **Dark mode?** — yes (default), no, or system-follow
- **Which screens need content?** — list the screens from their navigation
- **Design reference?** — Figma link, screenshot, or "just make it clean"

If the user provides a Figma link, use the Figma MCP tools (`get_design_context`, `get_screenshot`) to extract design tokens, colors, spacing, and component specs.

## Step 2: Read Reference Files

Read these from `skills/react-native-arc/`:
1. `theme.md` — color schemes, useStyles, useAppTheme, OLED dark mode
2. `components.md` — component file structure, touch targets, haptics, expo-image
3. `performance.md` — memoization, FlatList, animations
4. `providers.md` — AppInitializationProvider

Read mobile design system:
5. `mobile-design/touch-psychology.md` — touch targets, thumb zones
6. `mobile-design/mobile-typography.md` — type scales, Dynamic Type, min sizes
7. `mobile-design/mobile-color-system.md` — OLED, dark mode, contrast
8. `mobile-design/platform-ios.md` — iOS HIG (if targeting iOS)
9. `mobile-design/platform-android.md` — Material Design 3 (if targeting Android)

Read templates:
10. `templates/component.md` — component scaffolding
11. `templates/screen.md` — screen scaffolding

## Step 3: Generate Theme System

1. **`src/theme/interfaces.ts`** — `AppTheme`, `AppColorSchemeName` types
2. **`src/theme/fonts.ts`** — font family config + `fontAssets` for expo-font
3. **`src/theme/index.ts`** — `colorSet()` function, all color schemes (light, dark, + user's brand), `themes` record

Color rules from mobile-design:
- Dark background: `#121212` not pure black (OLED smearing)
- Dark text: `#E8E8E8` not pure white (eye strain)
- Status colors shift lighter in dark mode (iOS system color pattern)
- All text meets WCAG AA: 4.5:1 for body, 3:1 for large

## Step 4: Generate Core Hooks

1. **`src/hooks/use-app-theme.ts`** — access theme from context
2. **`src/hooks/use-styles.ts`** — `useStyles(callback)` with theme + safe area insets
3. **`src/hooks/use-app-initialization.ts`** — access full init context (theme + setColorScheme)

## Step 5: Generate AppInitializationProvider

1. **`src/providers/app-initialization/app-initialization.types.ts`**
2. **`src/providers/app-initialization/app-initialization.tsx`** — font loading, splash screen, theme state, `setColorScheme`
3. **`src/providers/app-initialization/index.tsx`** — barrel export

## Step 6: Generate Base Components

Create each following the structure from `components.md` (index.tsx + .styles.ts + .types.ts):

1. **`button`** — variants (primary, secondary, outline, ghost), sizes (sm, md, lg), loading state, haptic feedback
2. **`pressable`** — themed Pressable wrapper with optional haptics
3. **`app-text-input`** — label, placeholder, error message, icon, secure toggle
4. **`app-scroll-view`** — themed ScrollView with safe area padding
5. **`skeleton`** — animated loading placeholder
6. **`divider`** — horizontal/vertical separator
7. **`avatar`** — user avatar with expo-image, fallback initials

Component rules:
- All sizing via `moderateScale`
- Touch targets min 44pt (iOS) / 48dp (Android)
- Haptic feedback on meaningful interactions
- Platform-specific shadows (iOS shadowColor vs Android elevation)

## Step 7: Generate Toast System

```
src/components/toast/
├── index.tsx
├── toast.types.ts
├── context/index.tsx              — ToastContext + useToast hook
└── components/
    ├── index.ts
    ├── toast-provider/index.tsx   — state manager (max 3 concurrent)
    ├── toast-container/           — animated container + styles
    └── toast-content/             — message UI + styles
```

## Step 8: Generate Screen Content

For each screen the user listed, generate:
1. `[screen-name].tsx` — screen component with real layout
2. `[screen-name].styles.ts` — styles using theme
3. Screen-specific `components/` if needed

Screen patterns (pick based on screen purpose):
- **List screen** — FlatList with React.memo items, loading/error/empty states
- **Detail screen** — ScrollView with sections, images, actions
- **Form screen** — Formik form with validation, KeyboardAvoidingView
- **Settings screen** — sectioned list with toggle items
- **Profile screen** — avatar, info cards, settings list

All screens must:
- Use `useStyles(createStyles)` for all styling
- Handle loading, error, and empty states
- Respect safe area insets
- Place primary CTAs in thumb-friendly zone (bottom)

## Step 9: Update Provider Stack

Update `src/app.tsx` to add AppInitializationProvider and ToastProvider in the correct position (see `providers.md`).

## Step 10: Add Appearance Screen (if dark mode)

Generate `src/screens/profile/appearance/` — screen with color scheme picker showing all available themes with preview swatches.

## What This Agent Does NOT Do

- Project init, folder structure → `/arc-scaffold`
- API connection, storage, state → `/arc-connect`
- Adding new feature domains → `/arc-feature`

## Output

Tell the user:
1. What was created (theme, components, screens)
2. Color scheme count and how to switch
3. Next steps: run the app with `npx expo start`
