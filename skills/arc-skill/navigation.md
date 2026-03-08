# Navigation Architecture

## Stack: React Navigation

- `@react-navigation/native` — core
- `@react-navigation/native-stack` — native stack (not JS stack)
- `@react-navigation/bottom-tabs` — tab navigator

## Navigation Type Definitions

```typescript
// src/types/navigation.types.ts

export type RootStackParamList = {
  // Onboarding / Auth
  Login: undefined;
  Register: undefined;
  ForgotPassword: undefined;

  // Main
  MainTabs: undefined;

  // Stack screens accessible from any tab
  Settings: undefined;
  AccountDetails: undefined;
  Appearance: undefined;
  Language: undefined;

  // Domain screens
  // [ScreenName]: { paramName: ParamType } | undefined;
};

export type MainTabParamList = {
  HomeTab: undefined;
  SearchTab: undefined;
  ProfileTab: undefined;
  // Add tabs based on app needs
};
```

## Typed Navigation Hook

```typescript
// src/hooks/use-app-navigation.ts

import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';
import type { RootStackParamList } from '@/types/navigation.types';

export const useAppNavigation = () =>
  useNavigation<NativeStackNavigationProp<RootStackParamList>>();
```

Usage in components:
```typescript
const navigation = useAppNavigation();
navigation.navigate('Settings');  // ✅ Fully typed
navigation.navigate('Foo');       // ❌ Type error
```

## Root Navigator Structure

```typescript
// src/navigation/index.tsx

import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import type { RootStackParamList, MainTabParamList } from '@/types/navigation.types';

const Stack = createNativeStackNavigator<RootStackParamList>();
const Tab = createBottomTabNavigator<MainTabParamList>();

// --- Tab Navigator ---
const MainTabs = () => {
  const { colors } = useAppTheme();

  return (
    <Tab.Navigator
      screenOptions={{
        headerShown: false,
        tabBarActiveTintColor: colors.primary,
        tabBarInactiveTintColor: colors.icon,
        tabBarStyle: {
          backgroundColor: colors.card,
          borderTopColor: colors.border,
        },
      }}
    >
      <Tab.Screen
        name="HomeTab"
        component={HomeScreen}
        options={{
          tabBarLabel: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home-outline" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen name="SearchTab" component={SearchScreen} />
      <Tab.Screen name="ProfileTab" component={ProfileScreen} />
    </Tab.Navigator>
  );
};

// --- Root Stack Navigator ---
export const RootNavigator = () => {
  const isAuthenticated = useIsAuthenticated(); // from auth store
  const hasCompletedOnboarding = useHasCompletedOnboarding(); // from storage

  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      {!isAuthenticated ? (
        // Auth screens
        <>
          <Stack.Screen name="Login" component={LoginScreen} />
          <Stack.Screen name="Register" component={RegisterScreen} />
          <Stack.Screen name="ForgotPassword" component={ForgotPasswordScreen} />
        </>
      ) : (
        // Authenticated screens
        <>
          <Stack.Screen name="MainTabs" component={MainTabs} />
          <Stack.Screen name="Settings" component={SettingsScreen} />
          <Stack.Screen name="AccountDetails" component={AccountDetailsScreen} />
          <Stack.Screen name="Appearance" component={AppearanceScreen} />
          <Stack.Screen name="Language" component={LanguageScreen} />
        </>
      )}
    </Stack.Navigator>
  );
};
```

## Navigation Patterns

### Conditional Initial Route
```typescript
// Determine initial route based on stored state
const initialRoute = !hasCompletedOnboarding
  ? 'Onboarding'
  : !isAuthenticated
  ? 'Login'
  : 'MainTabs';

<Stack.Navigator initialRouteName={initialRoute}>
```

### Screen Options Per Screen
```typescript
<Stack.Screen
  name="Settings"
  component={SettingsScreen}
  options={{
    headerShown: true,
    title: 'Settings',
    headerBackTitle: 'Back',
    animation: 'slide_from_right',
  }}
/>
```

### Navigation from Components
```typescript
const SomeComponent = () => {
  const navigation = useAppNavigation();

  const handlePress = () => {
    navigation.navigate('AccountDetails');
  };

  const handleBack = () => {
    navigation.goBack();
  };

  return <Button onPress={handlePress} title="Account" />;
};
```

### Passing Parameters
```typescript
// In types:
export type RootStackParamList = {
  ItemDetail: { itemId: string };
};

// Navigate:
navigation.navigate('ItemDetail', { itemId: '123' });

// Receive in screen:
import { useRoute, RouteProp } from '@react-navigation/native';

const route = useRoute<RouteProp<RootStackParamList, 'ItemDetail'>>();
const { itemId } = route.params;
```

## Deep Linking (Optional)

```typescript
const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      MainTabs: {
        screens: {
          HomeTab: 'home',
          SearchTab: 'search',
          ProfileTab: 'profile',
        },
      },
      ItemDetail: 'item/:itemId',
    },
  },
};

// In NavigationContainer:
<NavigationContainer linking={linking}>
```

## Tab Bar UX Guidelines (from mobile-design)

**iOS:**
- Height: 49pt (83pt with home indicator)
- Max items: 5
- Icons: SF Symbols (25×25pt) — always show labels
- Tab bar always visible (don't hide on scroll)

**Android:**
- Height: 80dp
- Max items: 5 (3-5 ideal)
- Icons: Material Symbols (24dp) — always show labels
- Active tab: pill shape + filled icon (Material 3)

**Tab State Preservation (critical):**
Each tab MUST maintain its own navigation stack. Switching tabs and back should preserve position:
```
1. Home tab → Item detail → Add to cart
2. Switch to Profile tab
3. Switch back to Home tab
→ Must return to "Add to cart" screen, NOT root
```

## Back Navigation

**iOS:** Edge swipe from left (system gesture). Never override swipe back.
**Android:** System back button/gesture. Must handle correctly. Predictive back (Android 14+).

## Performance Tips for Navigation

- Use `react-native-screens` (installed with React Navigation) to enable native screen containers
- Lazy load tab screens with `lazy={true}` (default)
- Use `freezeOnBlur={true}` to freeze inactive screens (reduces re-renders)
- Prefer `@react-navigation/native-stack` over `@react-navigation/stack` (uses native navigation APIs)
- Avoid heavy computation in screen components — move to React Query or useMemo
- Plan deep linking from day one — not as an afterthought
