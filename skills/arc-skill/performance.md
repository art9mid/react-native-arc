# React Native Performance Best Practices

Reference: https://reactnative.dev/docs/performance

## React Compiler

Enabled via `app.json` → `expo.experiments.reactCompiler: true`. Expo handles the babel plugin automatically. The compiler auto-memoizes components, hooks, and expressions at build time. This means:

- **Do NOT manually add** `React.memo()`, `useMemo()`, or `useCallback()` — the compiler handles it
- **Exception**: `useStyles` hook keeps its own `useMemo` since the compiler doesn't optimize `StyleSheet.create` calls
- If you see manual memoization in existing code, it's safe to remove — the compiler will optimize it anyway

## Core Principles

1. **Minimize JavaScript bridge traffic** — use native solutions where possible
2. **Reduce re-renders** — React Compiler handles memoization automatically
3. **Offload animations to native thread** — use Reanimated, not Animated API
4. **Lazy load and paginate** — only render and fetch what's needed
5. **Profile first, optimize second** — measure before assuming bottlenecks

## List Performance (FlatList)

FlatList is the #1 performance concern in React Native.

```typescript
// ✅ GOOD: Optimized FlatList
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={keyExtractor}
  // --- Performance Props ---
  removeClippedSubviews={true}        // Unmount off-screen items (Android)
  maxToRenderPerBatch={10}             // Items per render batch
  windowSize={5}                       // Viewport multiplier for render window
  initialNumToRender={10}              // Items to render on first mount
  getItemLayout={getItemLayout}        // Skip measurement (if fixed height)
  // --- Prevent re-renders ---
  extraData={undefined}                // Only pass if list depends on external state
/>

// React Compiler handles memoization — just write plain functions
const renderItem = ({ item }: { item: ItemType }) => <ItemCard item={item} />;

const keyExtractor = (item: ItemType) => item.id;

// Fixed-height layout (skip measurement)
const ITEM_HEIGHT = moderateScale(80);
const getItemLayout = (_: any, index: number) => ({
  length: ITEM_HEIGHT,
  offset: ITEM_HEIGHT * index,
  index,
});
```

```typescript
// No React.memo needed — compiler optimizes automatically
const ItemCard = ({ item }: { item: ItemType }) => {
  const { styles } = useStyles(createItemStyles);
  return (
    <View style={styles.container}>
      <Text style={styles.title}>{item.title}</Text>
    </View>
  );
};
```

### FlashList Alternative

For very large lists (1000+ items), consider `@shopify/flash-list`:
```bash
npx expo install @shopify/flash-list
```

```typescript
import { FlashList } from '@shopify/flash-list';

<FlashList
  data={items}
  renderItem={renderItem}
  estimatedItemSize={80}  // Required for FlashList
  keyExtractor={keyExtractor}
/>
```

## Image Performance

```typescript
// ✅ Use expo-image instead of React Native Image
import { Image } from 'expo-image';

<Image
  source={{ uri: imageUrl }}
  style={styles.image}
  contentFit="cover"
  placeholder={blurhash}        // Show placeholder while loading
  transition={200}               // Fade in transition
  cachePolicy="memory-disk"      // Cache to memory + disk
/>
```

Install: `npx expo install expo-image`

**Rules:**
- Always set explicit `width` and `height` on images
- Use `blurhash` or `thumbhash` for placeholders
- Prefer WebP format over PNG/JPEG for smaller file sizes
- Use `cachePolicy="memory-disk"` for network images

## Animation Performance

**Use Reanimated for all animations** — runs on the UI thread (60fps):

```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
} from 'react-native-reanimated';

const MyComponent = () => {
  const opacity = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: withTiming(opacity.value, { duration: 300 }),
  }));

  useEffect(() => {
    opacity.value = 1;
  }, []);

  return <Animated.View style={[styles.box, animatedStyle]} />;
};
```

**Never use:**
- `Animated` from `react-native` for complex animations (runs on JS thread)
- `setInterval`/`setTimeout` for animations
- `LayoutAnimation` in heavy UI (causes full layout recalculation)

## Memoization (handled by React Compiler)

React Compiler auto-memoizes components, callbacks, and values. Write plain code — no manual wrapping needed:

```typescript
// ✅ Just write normal code — compiler memoizes automatically
const ExpensiveComponent = ({ data }: Props) => {
  return <View>...</View>;
};

const Parent = () => {
  const handlePress = () => {
    navigation.navigate('Detail');
  };

  return <ExpensiveComponent onPress={handlePress} />;
};

const filteredItems = items.filter((item) => item.isActive);
```

## Avoid Re-Renders

```typescript
// ❌ BAD: Inline style objects (new object every render)
<View style={{ flex: 1, backgroundColor: theme.colors.background }}>

// ✅ GOOD: Use useStyles hook
const { styles } = useStyles(createStyles);
<View style={styles.container}>
```

## Navigation Performance

- Use `@react-navigation/native-stack` (native) over `@react-navigation/stack` (JS)
- Enable `freezeOnBlur={true}` on screens to freeze inactive screens
- Lazy load tab screens (default behavior)
- Avoid heavy components in screen `options` — use lightweight header components

```typescript
<Tab.Navigator
  screenOptions={{
    lazy: true,
    freezeOnBlur: true,
  }}
>
```

## Startup Performance

1. **Minimize root component work** — defer non-critical initialization:
   ```typescript
   useEffect(() => {
     // Defer non-critical work
     InteractionManager.runAfterInteractions(() => {
       initAnalytics();
       prefetchData();
     });
   }, []);
   ```

2. **Use splash screen** — keep splash visible until app is ready:
   ```typescript
   import * as SplashScreen from 'expo-splash-screen';

   SplashScreen.preventAutoHideAsync();
   // ...after fonts loaded, auth checked, etc:
   SplashScreen.hideAsync();
   ```

3. **Load fonts asynchronously**:
   ```typescript
   const [fontsLoaded] = useFonts(fontAssets);
   if (!fontsLoaded) return null; // Splash screen stays visible
   ```

## React Query Performance

```typescript
// ✅ Prevent unnecessary refetches
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false,    // Don't refetch on app focus
      refetchOnMount: false,           // Don't refetch on mount if cached
      staleTime: 1000 * 60 * 5,       // Data fresh for 5 minutes
    },
  },
});

// ✅ Use `select` for data transformation (avoids re-render on same result)
const { data: activeItems } = useItemsQuery({
  select: (data) => data.data.filter((item) => item.isActive),
});

// ✅ Use `enabled` flag for conditional fetching
const { data } = useItemDetailQuery(itemId, {
  enabled: !!itemId,  // Only fetch when ID is available
});
```

## Storage Performance

- **Use MMKV over AsyncStorage** — 30x faster for reads, 10x for writes
- **Avoid storing large JSON blobs** — split into smaller keys if > 1MB
- **Don't read storage in render** — read once on mount, store in state/context

## JavaScript Bundle Size

- Use `expo-router` `lazy` imports or `React.lazy()` for screen-level code splitting
- Avoid importing entire libraries — use tree-shakeable imports:
  ```typescript
  // ❌ BAD
  import _ from 'lodash';

  // ✅ GOOD
  import debounce from 'lodash/debounce';
  ```
- Remove unused dependencies regularly
- Use `npx expo-doctor` to check for issues

## Hermes Engine

Expo with SDK 49+ uses Hermes by default. Ensure it's enabled:
```json
// app.json
{
  "expo": {
    "jsEngine": "hermes"
  }
}
```

Hermes provides:
- Faster startup (bytecode precompilation)
- Lower memory usage
- Better garbage collection

## Profiling Tools

1. **React DevTools Profiler** — find unnecessary re-renders
2. **Flipper** — network, layout, and performance inspection
3. **`console.time()`/`console.timeEnd()`** — manual timing
4. **Expo Dev Tools** — performance monitoring
5. **`why-did-you-render`** — detect avoidable re-renders in development

```bash
# Install why-did-you-render
npm install -D @welldone-software/why-did-you-render
```

## Performance Checklist

- [ ] React Compiler enabled (`expo.experiments.reactCompiler: true` in app.json)
- [ ] FlatList with `removeClippedSubviews`, `maxToRenderPerBatch`, `windowSize`
- [ ] Styles via `useStyles()` (not inline objects)
- [ ] Images with explicit dimensions and caching
- [ ] Animations with Reanimated (not Animated API)
- [ ] Native stack navigator (not JS stack)
- [ ] `freezeOnBlur` enabled on tab screens
- [ ] MMKV for storage (not AsyncStorage)
- [ ] React Query with reasonable `staleTime`
- [ ] Hermes engine enabled
- [ ] Tree-shakeable imports (no full library imports)
- [ ] Splash screen until app is fully loaded
