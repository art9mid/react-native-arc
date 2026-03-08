# Theme System

## Architecture

The theme system provides:
- Multiple color schemes (light, dark, + custom)
- Type-safe theme access via hooks
- Persistence of user preference in MMKV
- Responsive sizing with `react-native-size-matters`
- Safe area inset awareness

## Theme Interface

```typescript
// src/theme/interfaces.ts

export type AppColorSchemeName =
  | 'light'
  | 'dark'
  | 'oceanBlue'
  | 'forestGreen'
  | 'purple'
  | 'sunsetOrange'
  | 'teal'
  | 'rose';

export interface AppTheme {
  dark: boolean;
  colors: {
    // Core
    primary: string;
    secondary: string;
    background: string;
    card: string;
    border: string;

    // Text
    text: string;
    textSecondary: string;
    textWhite: string;
    textPrimary: string;

    // Icons
    icon: string;
    iconActive: string;

    // Status
    success: string;
    error: string;
    warning: string;
    info: string;

    // Toast
    toastSuccess: string;
    toastError: string;
    toastInfo: string;
    toastText: string;

    // Misc
    skeleton: string;
    skeletonHighlight: string;
    shadow: string;
    overlay: string;
  };
  fonts: {
    regular: string;
    medium: string;
    semibold: string;
    bold: string;
  };
}
```

## Color Scheme Definitions

```typescript
// src/theme/index.ts

import { moderateScale } from 'react-native-size-matters';
import type { AppTheme, AppColorSchemeName } from './interfaces';

// Static sizes — not in theme, they never change and don't need re-renders
export const sizes = {
  paddingVertical: moderateScale(16),
  paddingHorizontal: moderateScale(20),
  mainBorderRadius: moderateScale(12),
  fieldGap: moderateScale(12),
  skeletonBorderRadius: moderateScale(8),
} as const;

const baseFonts = {
  regular: 'Inter-Regular',    // Replace with your font
  medium: 'Inter-Medium',
  semibold: 'Inter-SemiBold',
  bold: 'Inter-Bold',
};

// OLED: Dark backgrounds use #121212 not pure #000000.
// Pure black causes "smearing" on OLED during scroll.
// Near-black (#121212) avoids this while saving ~85% OLED battery vs white.
//
// Dark mode text: #E8E8E8 not pure #FFFFFF to reduce eye strain.
// Status colors shift lighter in dark mode (iOS system color pattern).
// All text contrasts must meet WCAG AA: 4.5:1 body, 3:1 large text.

const colorSet = (
  dark: boolean,
  primary: string,
  secondary: string,
): AppTheme => ({
  dark,
  colors: {
    primary,
    secondary,
    background: dark ? '#121212' : '#FFFFFF',
    card: dark ? '#1E1E1E' : '#F8F8F8',
    border: dark ? '#2C2C2C' : '#E5E5E5',

    text: dark ? '#E8E8E8' : '#1A1A1A',
    textSecondary: dark ? '#B0B0B0' : '#6B7280',
    textWhite: '#FFFFFF',
    textPrimary: primary,

    icon: dark ? '#8E8E93' : '#8E8E93',
    iconActive: primary,

    success: dark ? '#30D158' : '#34C759',
    error: dark ? '#FF453A' : '#FF3B30',
    warning: dark ? '#FF9F0A' : '#FF9500',
    info: dark ? '#0A84FF' : '#007AFF',

    toastSuccess: dark ? '#1B3A26' : '#E8F5E9',
    toastError: dark ? '#3A1B1B' : '#FFEBEE',
    toastInfo: dark ? '#1B2A3A' : '#E3F2FD',
    toastText: dark ? '#E8E8E8' : '#1A1A1A',

    skeleton: dark ? '#2C2C2C' : '#E5E5E5',
    skeletonHighlight: dark ? '#3C3C3C' : '#F0F0F0',
    shadow: dark ? '#000000' : '#000000',
    overlay: 'rgba(0, 0, 0, 0.5)',
  },
  fonts: baseFonts,
});

export const themes: Record<AppColorSchemeName, AppTheme> = {
  light: colorSet(false, '#007AFF', '#5856D6'),
  dark: colorSet(true, '#0A84FF', '#5E5CE6'),
  oceanBlue: colorSet(false, '#3B82F6', '#1E40AF'),
  forestGreen: colorSet(false, '#16A34A', '#15803D'),
  purple: colorSet(false, '#8B5CF6', '#7C3AED'),
  sunsetOrange: colorSet(false, '#EA580C', '#C2410C'),
  teal: colorSet(false, '#0D9488', '#0F766E'),
  rose: colorSet(false, '#E11D48', '#BE123C'),
};

export const defaultColorScheme: AppColorSchemeName = 'light';
```

## Font Configuration

```typescript
// src/theme/fonts.ts

export const fontFamilies = {
  regular: 'Inter-Regular',
  medium: 'Inter-Medium',
  semibold: 'Inter-SemiBold',
  bold: 'Inter-Bold',
} as const;

// Font assets for expo-font loading
export const fontAssets = {
  'Inter-Regular': require('../../assets/fonts/Inter-Regular.ttf'),
  'Inter-Medium': require('../../assets/fonts/Inter-Medium.ttf'),
  'Inter-SemiBold': require('../../assets/fonts/Inter-SemiBold.ttf'),
  'Inter-Bold': require('../../assets/fonts/Inter-Bold.ttf'),
};
```

## Theme Hook

```typescript
// src/hooks/use-app-theme.ts

import { useContext } from 'react';
import { AppInitializationContext } from '@/providers/app-initialization';
import type { AppTheme } from '@/theme/interfaces';

export const useAppTheme = (): AppTheme => {
  const context = useContext(AppInitializationContext);
  if (!context) {
    throw new Error('useAppTheme must be used within AppInitializationProvider');
  }
  return context.theme;
};
```

## useStyles Hook — Responsive Styles + Theme

```typescript
// src/hooks/use-styles.ts

import { useMemo } from 'react';
import { StyleSheet } from 'react-native';
import { useSafeAreaInsets, EdgeInsets } from 'react-native-safe-area-context';
import { useAppTheme } from './use-app-theme';
import type { AppTheme } from '@/theme/interfaces'

type StyleCallback<T extends StyleSheet.NamedStyles<T>> = (
  theme: AppTheme,
  insets: EdgeInsets,
) => T;

export const useStyles = <T extends StyleSheet.NamedStyles<T>>(
  styleCallback: StyleCallback<T>,
) => {
  const theme = useAppTheme();
  const insets = useSafeAreaInsets();

  const styles = useMemo(
    () => StyleSheet.create(styleCallback(theme, insets)),
    [theme, insets, styleCallback],
  );

  return { styles, theme, insets };
};
```

Usage:
```typescript
// In a component
const MyComponent = () => {
  const { styles, theme } = useStyles(createStyles);
  return <View style={styles.container}>...</View>;
};

// Style callback (defined OUTSIDE the component)
const createStyles = (theme: AppTheme, insets: EdgeInsets) => ({
  container: {
    flex: 1,
    backgroundColor: theme.colors.background,
    paddingTop: insets.top,
    paddingHorizontal: sizes.paddingHorizontal,
  },
  title: {
    color: theme.colors.text,
    fontFamily: theme.fonts.bold,
    fontSize: moderateScale(24),
  },
});
```

## Theme Persistence

```typescript
// src/store/storage/theme.ts

import { storage } from './index';
import { STORAGE_KEYS } from '@/constants/keys';
import type { AppColorSchemeName } from '@/theme/interfaces';

export const getStoredColorScheme = (): AppColorSchemeName | null =>
  storage.getString(STORAGE_KEYS.COLOR_SCHEME) as AppColorSchemeName | null;

export const setStoredColorScheme = (scheme: AppColorSchemeName): void =>
  storage.set(STORAGE_KEYS.COLOR_SCHEME, scheme);
```

## Theme Switching

The `AppInitializationProvider` exposes a `setColorScheme` function:

```typescript
const { setColorScheme } = useAppInitialization();
setColorScheme('dark');       // Switch to dark mode
setColorScheme('oceanBlue');  // Switch to ocean blue
```

See `providers.md` for full implementation.
