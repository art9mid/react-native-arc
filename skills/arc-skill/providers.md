# Provider Stack Architecture

## Provider Order (Root → Leaf)

The provider stack in `src/app.tsx` must follow this order (outer → inner):

```
GestureHandlerRootView              — Gesture system (required by Reanimated)
  └─ KeyboardProvider                — Keyboard management
     └─ PersistQueryClientProvider   — React Query + MMKV persistence
        └─ AppInitializationProvider — Theme, fonts, splash screen
           └─ SafeAreaProvider       — Safe area insets
              └─ I18nProvider        — Internationalization (if enabled)
                 └─ NavigationContainer — React Navigation
                    └─ ToastProvider — Global notifications
                       └─ RootNavigator — App screens
```

## Root App Component

```typescript
// src/app.tsx

import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { KeyboardProvider } from 'react-native-keyboard-controller';
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { NavigationContainer } from '@react-navigation/native';
import { StatusBar } from 'expo-status-bar';

import { queryClient, appPersister } from '@/store/services';
import { AppInitializationProvider } from '@/providers/app-initialization';
import { I18nProvider } from '@/locale/i18n-provider'; // if i18n enabled
import { ToastProvider } from '@/components/toast';
import { RootNavigator } from '@/navigation';
import { appStyles } from './app.styles';

export const App = () => {
  return (
    <GestureHandlerRootView style={appStyles.root}>
      <KeyboardProvider>
        <PersistQueryClientProvider
          client={queryClient}
          persistOptions={{ persister: appPersister }}
        >
          <AppInitializationProvider>
            <SafeAreaProvider>
              <I18nProvider>
                <NavigationContainer>
                  <StatusBar style="auto" />
                  <ToastProvider>
                    <RootNavigator />
                  </ToastProvider>
                </NavigationContainer>
              </I18nProvider>
            </SafeAreaProvider>
          </AppInitializationProvider>
        </PersistQueryClientProvider>
      </KeyboardProvider>
    </GestureHandlerRootView>
  );
};
```

```typescript
// src/app.styles.ts
import { StyleSheet } from 'react-native';

export const appStyles = StyleSheet.create({
  root: {
    flex: 1,
  },
});
```

```typescript
// src/index.ts
import { registerRootComponent } from 'expo';
import { App } from './app';

registerRootComponent(App);
```

## AppInitializationProvider

Handles theme initialization, font loading, and splash screen:

```typescript
// src/providers/app-initialization/app-initialization.types.ts

import type { AppTheme, AppColorSchemeName } from '@/theme/interfaces';

export interface AppInitializationContextValue {
  theme: AppTheme;
  colorScheme: AppColorSchemeName;
  setColorScheme: (scheme: AppColorSchemeName) => void;
  isReady: boolean;
}
```

```typescript
// src/providers/app-initialization/app-initialization.tsx

import { createContext, useCallback, useEffect, useMemo, useState } from 'react';
import { useFonts } from 'expo-font';
import * as SplashScreen from 'expo-splash-screen';
import { themes, defaultColorScheme } from '@/theme';
import { fontAssets } from '@/theme/fonts';
import { getStoredColorScheme, setStoredColorScheme } from '@/store/storage/theme';
import type { AppColorSchemeName } from '@/theme/interfaces';
import type { AppInitializationContextValue } from './app-initialization.types';

SplashScreen.preventAutoHideAsync();

export const AppInitializationContext =
  createContext<AppInitializationContextValue | null>(null);

export const AppInitializationProvider = ({
  children,
}: {
  children: React.ReactNode;
}) => {
  const [fontsLoaded] = useFonts(fontAssets);
  const [colorScheme, setColorSchemeState] = useState<AppColorSchemeName>(
    () => getStoredColorScheme() ?? defaultColorScheme,
  );

  const theme = useMemo(() => themes[colorScheme], [colorScheme]);

  const setColorScheme = useCallback((scheme: AppColorSchemeName) => {
    setColorSchemeState(scheme);
    setStoredColorScheme(scheme);
  }, []);

  const isReady = fontsLoaded;

  useEffect(() => {
    if (isReady) {
      SplashScreen.hideAsync();
    }
  }, [isReady]);

  const value = useMemo<AppInitializationContextValue>(
    () => ({
      theme,
      colorScheme,
      setColorScheme,
      isReady,
    }),
    [theme, colorScheme, setColorScheme, isReady],
  );

  if (!isReady) return null;

  return (
    <AppInitializationContext.Provider value={value}>
      {children}
    </AppInitializationContext.Provider>
  );
};
```

```typescript
// src/providers/app-initialization/index.tsx
export { AppInitializationProvider, AppInitializationContext } from './app-initialization';
export type { AppInitializationContextValue } from './app-initialization.types';
```

## Custom Provider Pattern

For adding new providers:

```typescript
// src/providers/[provider-name]/[provider-name].types.ts
export interface [ProviderName]ContextValue {
  // context values
}

// src/providers/[provider-name]/[provider-name].tsx
import { createContext, useMemo } from 'react';
import type { [ProviderName]ContextValue } from './[provider-name].types';

export const [ProviderName]Context = createContext<[ProviderName]ContextValue | null>(null);

export const [ProviderName]Provider = ({ children }: { children: React.ReactNode }) => {
  const value = useMemo(() => ({
    // ...
  }), [/* deps */]);

  return (
    <[ProviderName]Context.Provider value={value}>
      {children}
    </[ProviderName]Context.Provider>
  );
};

// src/providers/[provider-name]/index.tsx
export { [ProviderName]Provider, [ProviderName]Context } from './[provider-name]';
```

## Rules

1. **Provider order matters** — gestureHandler must be outermost, navigation must wrap screens
2. **Context values must be memoized** — wrap value object in `useMemo` to prevent re-renders
3. **Callbacks in context must be memoized** — wrap with `useCallback`
4. **One provider per concern** — don't combine unrelated state in one provider
5. **Throw on missing provider** — hooks should throw if used outside their provider
6. **Keep providers thin** — business logic goes in hooks, not providers
