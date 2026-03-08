# Component Templates

Use these templates when generating new components. Replace `[ComponentName]` with PascalCase name and `[component-name]` with kebab-case name.

## Types File

```typescript
// src/components/[component-name]/[component-name].types.ts

export interface [ComponentName]Props {
  // Define props here
}
```

## Constants File (optional)

```typescript
// src/components/[component-name]/[component-name].constants.ts

import { moderateScale } from 'react-native-size-matters';

// Define constants, sizes, variants here
```

## Styles File

```typescript
// src/components/[component-name]/[component-name].styles.ts

import { moderateScale } from 'react-native-size-matters';
import type { AppTheme } from '@/theme/interfaces';
import type { EdgeInsets } from 'react-native-safe-area-context';

export const createStyles = (theme: AppTheme, insets: EdgeInsets) => ({
  container: {
    // styles
  },
});
```

## Component File (index.tsx)

```typescript
// src/components/[component-name]/index.tsx

import { View, Text } from 'react-native';
import { useStyles } from '@/hooks/use-styles';
import { createStyles } from './[component-name].styles';
import type { [ComponentName]Props } from './[component-name].types';

export { type [ComponentName]Props } from './[component-name].types';

export const [ComponentName] = ({ ...props }: [ComponentName]Props) => {
  const { styles, theme } = useStyles(createStyles);

  return (
    <View style={styles.container}>
      {/* Component content */}
    </View>
  );
};
```

## Utils File (optional)

```typescript
// src/components/[component-name]/[component-name].utils.ts

// Pure helper functions specific to this component
```

## Hooks File (optional)

```typescript
// src/components/[component-name]/[component-name].hooks.ts

import { useState, useCallback } from 'react';

// Component-specific hooks
export const use[ComponentName]State = () => {
  // Custom hook logic
};
```

## Sub-Component Structure

```
[component-name]/
└── components/
    ├── index.ts                         # Re-export all sub-components
    └── [sub-component-name]/
        ├── index.tsx                    # Sub-component
        └── [sub-component-name].styles.ts  # Sub-component styles
```

### Sub-Component Barrel

```typescript
// src/components/[component-name]/components/index.ts

export { [SubComponentName] } from './[sub-component-name]';
```

### Sub-Component File

```typescript
// src/components/[component-name]/components/[sub-component-name]/index.tsx

import { View } from 'react-native';
import { useStyles } from '@/hooks/use-styles';
import { createStyles } from './[sub-component-name].styles';

interface [SubComponentName]Props {
  // Props (can be inline for simple sub-components)
}

export const [SubComponentName] = ({ ...props }: [SubComponentName]Props) => {
  const { styles } = useStyles(createStyles);

  return <View style={styles.container} />;
};
```
