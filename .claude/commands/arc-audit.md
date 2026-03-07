---
name: arc-audit
description: "Audit a React Native Expo project for mobile UX issues, performance problems, and architecture violations. Use to check code quality against react-native-arc best practices."
---

# Arc Audit — Mobile UX & Architecture Check

You are auditing an existing React Native (Expo) project against the react-native-arc architecture standards and mobile design best practices.

## Step 1: Ask Scope

Ask the user:
- **Full audit** (everything) or **focused** (pick categories)?
- If focused, which: `touch`, `performance`, `architecture`, `styling`, `navigation`, `accessibility`

## Step 2: Read Reference Files

Read from `skills/react-native-arc/`:
1. `mobile-design/SKILL.md` — anti-patterns checklist
2. `mobile-design/touch-psychology.md` — touch target rules
3. `mobile-design/mobile-performance.md` — performance checklist
4. `components.md` — component structure rules
5. `performance.md` — FlatList, memoization, images

## Step 3: Run Checks

Search the codebase for violations in each category:

### Touch & UX
- [ ] Touch targets below 44pt/48dp (search for `height:` and `width:` values < 44 in styles)
- [ ] Missing `hitSlop` on small interactive elements
- [ ] Gesture-only interactions without button alternatives
- [ ] Missing haptic feedback on important actions (search for `onPress` without `Haptics`)
- [ ] Primary CTAs positioned at top instead of bottom (thumb zone)

### Performance
- [ ] `ScrollView` used for lists (should be `FlatList` or `FlashList`)
- [ ] `renderItem` not wrapped in `useCallback`
- [ ] List item components not wrapped in `React.memo`
- [ ] Missing `keyExtractor` or using index as key
- [ ] Missing `removeClippedSubviews` on FlatLists
- [ ] React Native `Image` instead of `expo-image`
- [ ] `Animated` API used instead of `react-native-reanimated`
- [ ] `console.log` statements left in code
- [ ] Missing `useCallback`/`useMemo` for props passed to memoized children
- [ ] Heavy computation inside render (no `useMemo`)

### Architecture
- [ ] Styles defined inline or in the same file as component (should be `.styles.ts`)
- [ ] Types defined in component file (should be `.types.ts`)
- [ ] Missing barrel exports (`index.ts`) in folders
- [ ] Direct MMKV access in components (should use storage wrappers)
- [ ] Direct API calls in components (should use React Query hooks)
- [ ] Hardcoded pixel values (should use `moderateScale`)
- [ ] Hardcoded color values (should use theme)
- [ ] Missing `useStyles()` hook (direct `StyleSheet.create`)

### Styling & Design
- [ ] Hardcoded colors outside theme (search for `#` in style files)
- [ ] Pure black `#000000` used in dark mode backgrounds (should be `#121212`)
- [ ] Pure white `#FFFFFF` used for dark mode text (should be `#E8E8E8`)
- [ ] Text contrast below 4.5:1 (check light text on light backgrounds)
- [ ] Inconsistent spacing (not using theme sizes)

### Navigation
- [ ] Untyped navigation (missing `useAppNavigation` hook)
- [ ] Missing `freezeOnBlur` on tab screens
- [ ] Using JS stack (`@react-navigation/stack`) instead of native stack
- [ ] Deep linking not configured

### Accessibility
- [ ] Missing `accessibilityLabel` on icon-only buttons
- [ ] Missing `accessibilityRole` on interactive elements
- [ ] Images without `accessibilityLabel`
- [ ] No support for Dynamic Type / font scaling

## Step 4: Run Automated Audit Script (if available)

If the project has the audit script:
```bash
python skills/react-native-arc/mobile-design/scripts/mobile_audit.py ./src
```

## Step 5: Report

Format the audit report as:

```
## Audit Report — [Project Name]

### Critical (fix immediately)
- 🔴 [Issue]: [File:Line] — [What's wrong] → [How to fix]

### Warnings (should fix)
- 🟡 [Issue]: [File:Line] — [What's wrong] → [How to fix]

### Info (consider fixing)
- 🔵 [Issue]: [File:Line] — [What's wrong] → [How to fix]

### Passed ✅
- [Category]: [What's good]

### Score: X/10
```

## Step 6: Offer Fixes

Ask the user: "Want me to fix the critical and warning issues automatically?"

If yes, fix them in order of severity. For each fix:
1. Read the file
2. Apply the fix following architecture patterns
3. Verify the fix doesn't break anything
