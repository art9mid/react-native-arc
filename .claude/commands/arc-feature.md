---
name: arc-feature
description: "Add a complete new feature domain to an existing React Native Expo project — types, API module, React Query hooks, screen, and components in one go. Use when adding a new entity like products, orders, or chat."
---

# Arc Feature — Add a New Domain

You are adding a complete new feature/domain to an existing React Native (Expo) project that follows the arc-skill architecture. This generates everything needed for one domain in a single pass.

## Step 1: Gather Requirements

Ask the user for:
- **Domain name** (singular, e.g. "product", "order", "message", "recipe")
- **Fields** — list the key fields (e.g. "id, title, description, price, imageUrl, category")
- **API endpoints** — or "standard CRUD" (list, detail, create, update, delete)
- **Screens needed** — list screen (default), detail screen, create/edit form, or "all"
- **Which tab** does this belong to? (e.g. "HomeTab", "SearchTab")
- **Pagination type** — offset (default) or cursor-based

## Step 2: Read Reference Files

Read templates from `skills/arc-skill/templates/`:
1. `api-service.md` — API module + React Query hooks + query keys
2. `component.md` — component structure
3. `screen.md` — screen with data fetching
4. `hook.md` — if custom hooks needed

Read architecture:
5. `components.md` — component rules, touch targets, memoization
6. `performance.md` — FlatList optimization, React.memo

## Step 3: Generate Type Definitions

**`src/types/[domain].types.ts`**:
- `[Domain]` interface (the entity with all fields)
- `Create[Domain]DTO` interface (fields for creation)
- `Update[Domain]DTO` interface (partial fields for update)

## Step 4: Generate API Module

**`src/api/http/[domain].ts`**:
- `get[Domains](params?)` — list (paginated)
- `get[Domain]ById(id)` — detail
- `create[Domain](data)` — create
- `update[Domain](id, data)` — update
- `delete[Domain](id)` — delete

## Step 5: Update Query Keys

**Update `src/constants/keys.ts`**:
```typescript
[DOMAINS]: ['[domains]'] as const,
[DOMAIN]_DETAIL: (id: string) => ['[domains]', id] as const,
```

## Step 6: Generate React Query Hooks

**`src/store/services/[domain].ts`**:
- `use[Domains]Query(params?)` — list query
- `use[Domain]Query(id)` — detail query (enabled when id exists)
- `use[Domains]InfiniteQuery()` — infinite scroll (if applicable)
- `useCreate[Domain]Mutation()` — create + invalidate list
- `useUpdate[Domain]Mutation()` — update + invalidate list + update detail cache
- `useDelete[Domain]Mutation()` — delete + invalidate list

## Step 7: Generate Mock Data

**`src/mocks/[domain].ts`**:
- `MOCK_[DOMAINS]` — array of 5-10 mock items with realistic data

## Step 8: Generate Screen Components

### List Screen
**`src/screens/[tab]/[domain]-list/`**:
- `[domain]-list.tsx` — FlatList with loading/error/empty states
- `[domain]-list.styles.ts`
- `components/[domain]-card/` — list item component (React.memo)
  - `index.tsx` + `[domain]-card.styles.ts` + `[domain]-card.types.ts`

FlatList must include:
- `React.memo` on card component
- `useCallback` on renderItem and keyExtractor
- `removeClippedSubviews`, `maxToRenderPerBatch={10}`, `windowSize={5}`
- Pull-to-refresh via `onRefresh`

### Detail Screen (if requested)
**`src/screens/[tab]/[domain]-detail/`**:
- `[domain]-detail.tsx` — ScrollView with sections
- `[domain]-detail.styles.ts`

### Form Screen (if requested)
**`src/screens/[tab]/[domain]-form/`**:
- `[domain]-form.tsx` — Formik form with Yup validation
- `[domain]-form.styles.ts`

## Step 9: Update Navigation

1. Add new screens to `RootStackParamList` in `src/types/navigation.types.ts`
2. Add `<Stack.Screen>` entries in `src/navigation/index.tsx`
3. Wire up navigation from list → detail, list → form

## Step 10: Update Barrel Exports

- `src/screens/[tab]/index.ts` — export new screens
- `src/store/services/index.ts` — re-export domain hooks (if using barrel)

## Output

Tell the user:
1. Files created (count + tree)
2. How to test: set `USE_MOCK_DATA = true`, navigate to the screen
3. What to do next: connect real API endpoint in `.env`
