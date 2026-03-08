# Screen Templates

Screens live in `src/screens/[tab-group]/[screen-name]/`. Replace placeholders accordingly.

## Screen Component

```typescript
// src/screens/[tab-group]/[screen-name]/[screen-name].tsx

import { View, Text } from 'react-native';
import { useStyles } from '@/hooks/use-styles';
import { useAppNavigation } from '@/hooks/use-app-navigation';
import { createStyles } from './[screen-name].styles';

export const [ScreenName]Screen = () => {
  const { styles, theme } = useStyles(createStyles);
  const navigation = useAppNavigation();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>[ScreenName]</Text>
    </View>
  );
};
```

## Screen Styles

```typescript
// src/screens/[tab-group]/[screen-name]/[screen-name].styles.ts

import { moderateScale } from 'react-native-size-matters';
import type { AppTheme } from '@/theme/interfaces';
import type { EdgeInsets } from 'react-native-safe-area-context';

export const createStyles = (theme: AppTheme, insets: EdgeInsets) => ({
  container: {
    flex: 1,
    backgroundColor: theme.colors.background,
    paddingTop: insets.top,
    paddingHorizontal: sizes.paddingHorizontal,
  },
  title: {
    fontSize: moderateScale(24),
    fontFamily: theme.fonts.bold,
    color: theme.colors.text,
    marginBottom: moderateScale(16),
  },
});
```

## Screen Barrel Export

```typescript
// src/screens/[tab-group]/[screen-name]/index.ts

export { [ScreenName]Screen } from './[screen-name]';
```

## Tab Group Barrel Export

```typescript
// src/screens/[tab-group]/index.ts

export { [Screen1Name]Screen } from './[screen-1-name]';
export { [Screen2Name]Screen } from './[screen-2-name]';
```

## Screen with Data Fetching

```typescript
// src/screens/[tab-group]/[screen-name]/[screen-name].tsx

import { View, Text, FlatList, ActivityIndicator } from 'react-native';
import { useStyles } from '@/hooks/use-styles';
import { useAppNavigation } from '@/hooks/use-app-navigation';
import { use[Domain]Query } from '@/store/services/[domain]';
import { formatApiError } from '@/helpers/format-api-error';
import { createStyles } from './[screen-name].styles';
import type { [ItemType] } from '@/types/[domain].types';

export const [ScreenName]Screen = () => {
  const { styles, theme } = useStyles(createStyles);
  const navigation = useAppNavigation();
  const { data, isLoading, error, refetch } = use[Domain]Query();

  // React Compiler auto-memoizes — no useCallback/React.memo needed
  const renderItem = ({ item }: { item: [ItemType] }) => (
    <[ItemCard] item={item} />
  );

  const keyExtractor = (item: [ItemType]) => item.id;

  if (isLoading) {
    return (
      <View style={styles.centered}>
        <ActivityIndicator color={theme.colors.primary} />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centered}>
        <Text style={styles.error}>{formatApiError(error)}</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={data?.data}
        renderItem={renderItem}
        keyExtractor={keyExtractor}
        removeClippedSubviews
        maxToRenderPerBatch={10}
        windowSize={5}
      />
    </View>
  );
};
```

## Screen with Form (Auth Example)

```typescript
// src/screens/onboarding/login/login.tsx

import { View, Text } from 'react-native';
import { KeyboardAwareScrollView } from 'react-native-keyboard-controller';
import { Formik } from 'formik';
import * as Yup from 'yup';
import { useStyles } from '@/hooks/use-styles';
import { useLoginMutation } from '@/store/services/auth';
import { formatApiError } from '@/helpers/format-api-error';
import { AppTextInput } from '@/components/app-text-input';
import { Button } from '@/components/button';
import { createStyles } from './login.styles';
import type { LoginDTO } from '@/types/auth.types';

const loginSchema = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Email is required'),
  password: Yup.string().min(6, 'Min 6 characters').required('Password is required'),
});

const initialValues: LoginDTO = {
  email: '',
  password: '',
};

export const LoginScreen = () => {
  const { styles } = useStyles(createStyles);
  const loginMutation = useLoginMutation();

  const handleLogin = (values: LoginDTO) => {
    loginMutation.mutate(values, {
      onError: (error) => {
        // Show error (via toast or form error)
      },
    });
  };

  return (
    <KeyboardAwareScrollView
      style={styles.container}
      bottomOffset={50}
    >
      <Formik
        initialValues={initialValues}
        validationSchema={loginSchema}
        onSubmit={handleLogin}
      >
        {({ handleChange, handleBlur, handleSubmit, values, errors, touched }) => (
          <View style={styles.form}>
            <Text style={styles.title}>Welcome Back</Text>

            <AppTextInput
              label="Email"
              value={values.email}
              onChangeText={handleChange('email')}
              onBlur={handleBlur('email')}
              error={touched.email ? errors.email : undefined}
              keyboardType="email-address"
              autoCapitalize="none"
            />

            <AppTextInput
              label="Password"
              value={values.password}
              onChangeText={handleChange('password')}
              onBlur={handleBlur('password')}
              error={touched.password ? errors.password : undefined}
              secureTextEntry
            />

            <Button
              title="Login"
              onPress={() => handleSubmit()}
              loading={loginMutation.isPending}
              fullWidth
            />
          </View>
        )}
      </Formik>
    </KeyboardAwareScrollView>
  );
};
```

## Screen-Specific Components

```
[screen-name]/
└── components/
    ├── index.ts
    └── [component-name]/
        ├── index.tsx
        └── [component-name].styles.ts
```

These sub-components are screen-private — they are only used by this screen. If they become reusable, promote them to `src/components/`.
