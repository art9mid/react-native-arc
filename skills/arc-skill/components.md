# Component Architecture

## File Structure Per Component

Every component MUST follow this structure:

```
component-name/
├── index.tsx                       # Main component + barrel export
├── component-name.styles.ts        # StyleSheet via useStyles callback
├── component-name.types.ts         # Props interface + related types
├── component-name.constants.ts     # (optional) Sizes, variants, enums
├── component-name.utils.ts         # (optional) Pure helper functions
├── component-name.hooks.ts         # (optional) Component-specific hooks
└── components/                     # (optional) Sub-components
    ├── index.ts                    # Re-exports all sub-components
    └── sub-component/
        ├── index.tsx
        └── sub-component.styles.ts
```

## Rules

1. **One component per folder** — never put multiple components in one file
2. **Styles in separate file** — never inline StyleSheet.create in the component
3. **Types in separate file** — Props interface, enums, and types get their own file
4. **Constants in separate file** — variants, sizes, defaults go in constants
5. **index.tsx is the barrel** — it exports the component + re-exports types
6. **Style callbacks are pure functions** — defined outside of the component, not inline
7. **All sizing uses `react-native-size-matters`** — no hardcoded pixel values
8. **useStyles hook for all styles** — provides theme + insets

## Styling Rules

### borderCurve with borderRadius

Always use `borderCurve: 'continuous'` with `borderRadius` for smoother iOS-style corners:

```typescript
// ❌ Incorrect
{ borderRadius: 12 }

// ✅ Correct – smoother iOS-style corners
{ borderRadius: 12, borderCurve: 'continuous' }
```

### gap instead of margin for spacing

Use `gap` on the parent instead of margin on children for spacing between elements:

```tsx
// ❌ Incorrect – margin on children
<View>
  <Text style={{ marginBottom: 8 }}>Title</Text>
  <Text style={{ marginBottom: 8 }}>Subtitle</Text>
</View>

// ✅ Correct – gap on parent
<View style={{ gap: 8 }}>
  <Text>Title</Text>
  <Text>Subtitle</Text>
</View>
```

## Example: Button Component

### Types

```typescript
// src/components/button/button.types.ts

import type { PressableProps } from 'react-native';

export type ButtonVariant = 'primary' | 'secondary' | 'outline' | 'ghost';
export type ButtonSize = 'sm' | 'md' | 'lg';

export interface ButtonProps extends Omit<PressableProps, 'style'> {
  title: string;
  variant?: ButtonVariant;
  size?: ButtonSize;
  loading?: boolean;
  icon?: string;
  iconPosition?: 'left' | 'right';
  fullWidth?: boolean;
}
```

### Constants

```typescript
// src/components/button/button.constants.ts

import { moderateScale } from 'react-native-size-matters';
import type { ButtonSize } from './button.types';

export const BUTTON_HEIGHTS: Record<ButtonSize, number> = {
  sm: moderateScale(32),
  md: moderateScale(44),
  lg: moderateScale(56),
};

export const BUTTON_FONT_SIZES: Record<ButtonSize, number> = {
  sm: moderateScale(12),
  md: moderateScale(14),
  lg: moderateScale(16),
};

export const BUTTON_ICON_SIZES: Record<ButtonSize, number> = {
  sm: moderateScale(14),
  md: moderateScale(18),
  lg: moderateScale(22),
};

export const BUTTON_PADDING_HORIZONTAL: Record<ButtonSize, number> = {
  sm: moderateScale(12),
  md: moderateScale(16),
  lg: moderateScale(24),
};

export const DEFAULT_VARIANT = 'primary' as const;
export const DEFAULT_SIZE = 'md' as const;
```

### Styles

```typescript
// src/components/button/button.styles.ts

import { moderateScale } from 'react-native-size-matters';
import type { AppTheme } from '@/theme/interfaces';
import type { EdgeInsets } from 'react-native-safe-area-context';
import type { ButtonVariant, ButtonSize } from './button.types';
import { BUTTON_HEIGHTS, BUTTON_PADDING_HORIZONTAL } from './button.constants';

export const createStyles = (theme: AppTheme, _insets: EdgeInsets) => ({
  container: {
    borderRadius: sizes.mainBorderRadius,
    borderCurve: 'continuous' as const,
    alignItems: 'center' as const,
    justifyContent: 'center' as const,
    flexDirection: 'row' as const,
  },
  fullWidth: {
    width: '100%' as const,
  },
  label: {
    fontFamily: theme.fonts.semibold,
  },
  disabled: {
    opacity: 0.5,
  },
});

// Variant-specific styles (used dynamically)
export const getVariantStyles = (variant: ButtonVariant, theme: AppTheme) => {
  switch (variant) {
    case 'primary':
      return {
        container: { backgroundColor: theme.colors.primary },
        label: { color: theme.colors.textWhite },
      };
    case 'secondary':
      return {
        container: { backgroundColor: theme.colors.secondary },
        label: { color: theme.colors.text },
      };
    case 'outline':
      return {
        container: {
          backgroundColor: 'transparent',
          borderWidth: 1,
          borderColor: theme.colors.border,
        },
        label: { color: theme.colors.text },
      };
    case 'ghost':
      return {
        container: { backgroundColor: 'transparent' },
        label: { color: theme.colors.primary },
      };
  }
};

export const getSizeStyles = (size: ButtonSize) => ({
  container: {
    height: BUTTON_HEIGHTS[size],
    paddingHorizontal: BUTTON_PADDING_HORIZONTAL[size],
  },
});
```

### Component

```typescript
// src/components/button/index.tsx

import { ActivityIndicator, Pressable, Text, View } from 'react-native';
import { useStyles } from '@/hooks/use-styles';
import { createStyles, getVariantStyles, getSizeStyles } from './button.styles';
import { DEFAULT_VARIANT, DEFAULT_SIZE, BUTTON_FONT_SIZES } from './button.constants';
import type { ButtonProps } from './button.types';

export { type ButtonProps } from './button.types';

export const Button = ({
  title,
  variant = DEFAULT_VARIANT,
  size = DEFAULT_SIZE,
  loading = false,
  icon,
  iconPosition = 'left',
  fullWidth = false,
  disabled,
  ...pressableProps
}: ButtonProps) => {
  const { styles, theme } = useStyles(createStyles);
  const variantStyles = getVariantStyles(variant, theme);
  const sizeStyles = getSizeStyles(size);

  return (
    <Pressable
      disabled={disabled || loading}
      style={({ pressed }) => [
        styles.container,
        variantStyles.container,
        sizeStyles.container,
        fullWidth && styles.fullWidth,
        (disabled || loading) && styles.disabled,
        pressed && { opacity: 0.8 },
      ]}
      {...pressableProps}
    >
      {loading ? (
        <ActivityIndicator color={variantStyles.label.color} />
      ) : (
        <Text
          style={[
            styles.label,
            variantStyles.label,
            { fontSize: BUTTON_FONT_SIZES[size] },
          ]}
        >
          {title}
        </Text>
      )}
    </Pressable>
  );
};
```

## Sub-Component Pattern

When a component has visual sub-parts that are reused internally:

```typescript
// src/components/button/components/button-icon/index.tsx

import { Ionicons } from '@expo/vector-icons';
import type { ButtonSize } from '../../button.types';
import { BUTTON_ICON_SIZES } from '../../button.constants';

interface ButtonIconProps {
  name: string;
  size: ButtonSize;
  color: string;
}

export const ButtonIcon = ({ name, size, color }: ButtonIconProps) => (
  <Ionicons name={name as any} size={BUTTON_ICON_SIZES[size]} color={color} />
);
```

```typescript
// src/components/button/components/index.ts
export { ButtonIcon } from './button-icon';
export { ButtonLabel } from './button-label';
```

## Shared Components Checklist

Every project should have these base components:

| Component | Purpose |
|-----------|---------|
| `button` | Primary action button with variants |
| `pressable` | Themed Pressable wrapper with haptics |
| `app-text-input` | Text input with label, validation, icons |
| `app-scroll-view` | Themed ScrollView with safe area |
| `skeleton` | Loading placeholder |
| `toast` | Global notification system |
| `divider` | Horizontal/vertical separator |
| `avatar` | User avatar with fallback (expo-image) |
| `badge` | Status/notification badge |
| `bottom-sheet` | Modal bottom sheet |
| `app-image` | Wrapper around expo-image with blurhash + caching |

## Touch Target Rules (from mobile-design system)

All interactive components MUST respect minimum touch targets:

| Platform | Minimum Size | Recommended | Spacing Between |
|----------|-------------|-------------|-----------------|
| iOS | 44pt × 44pt | 48pt+ | 8pt minimum |
| Android | 48dp × 48dp | 56dp+ | 8dp minimum |

```typescript
// ✅ Even if visual size is smaller, hitSlop expands touch area
<Pressable
  hitSlop={{ top: 8, bottom: 8, left: 8, right: 8 }}
  style={styles.smallButton}
>

// ✅ Touch target meets minimum
const MINIMUM_TOUCH_SIZE = moderateScale(44);
container: {
  minHeight: MINIMUM_TOUCH_SIZE,
  minWidth: MINIMUM_TOUCH_SIZE,
}
```

## Haptic Feedback Pattern

Interactive components should provide haptic feedback:

```typescript
import * as Haptics from 'expo-haptics';

// Light: toggles, selections
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
// Medium: standard tap, button press
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);
// Success/Error: task completion
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
```

## Image Component (expo-image)

Always use `expo-image` instead of React Native's `Image`:

```typescript
import { Image } from 'expo-image';

<Image
  source={{ uri: imageUrl }}
  style={styles.image}
  contentFit="cover"
  placeholder={{ blurhash: 'LKO2:N%2Tw=w]~RBVZRi};RPxuwH' }}
  transition={200}
  cachePolicy="memory-disk"
/>
```

## Platform-Aware Components

```typescript
import { Platform } from 'react-native';

const shadowStyle = Platform.select({
  ios: {
    shadowColor: theme.colors.shadow,
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  android: {
    elevation: 4,
  },
});
```

## Memoization

React Compiler is enabled — it auto-memoizes components, callbacks, and values at build time. **Do NOT manually add** `React.memo()`, `useCallback()`, or `useMemo()`. Just write plain code.
