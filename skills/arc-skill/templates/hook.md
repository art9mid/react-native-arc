# Hook Templates

All custom hooks live in `src/hooks/`. Component-specific hooks live in their component folder.

## Global Hook Template

```typescript
// src/hooks/use-[hook-name].ts

import { useState } from 'react';

/**
 * [Brief description of what the hook does]
 */
export const use[HookName] = (/* params */) => {
  const [value, setValue] = useState(/* initial */);

  // React Compiler auto-memoizes — no useCallback/useMemo needed
  const handler = () => {
    // logic
  };

  const computed = /* derived state */;

  return { value, handler, computed };
};
```

## Core App Hooks

### useAppTheme

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

### useAppNavigation

```typescript
// src/hooks/use-app-navigation.ts

import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';
import type { RootStackParamList } from '@/types/navigation.types';

export const useAppNavigation = () =>
  useNavigation<NativeStackNavigationProp<RootStackParamList>>();
```

### useStyles

```typescript
// src/hooks/use-styles.ts

import { useMemo } from 'react';
import { StyleSheet } from 'react-native';
import { useSafeAreaInsets, type EdgeInsets } from 'react-native-safe-area-context';
import { useAppTheme } from './use-app-theme';
import type { AppTheme } from '@/theme/interfaces';

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

## Component-Specific Hook Template

```typescript
// src/components/[component-name]/[component-name].hooks.ts

import { useState } from 'react';

export const use[ComponentName]State = () => {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = () => setIsOpen((prev) => !prev);
  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);

  return { isOpen, toggle, open, close };
};
```

## Rules

1. **Hooks start with `use`** — React requirement
2. **One hook per file** for global hooks in `src/hooks/`
3. **Component-specific hooks** go in `[component].hooks.ts` in the component folder
4. **No manual memoization** — React Compiler handles `useCallback`/`useMemo` automatically
5. **Throw descriptive errors** when used outside required providers
6. **Return objects, not arrays** — `{ value, handler }` not `[value, handler]`
   (exception: simple two-value hooks like `useDebounce`)
