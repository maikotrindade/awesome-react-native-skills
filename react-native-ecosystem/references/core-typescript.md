# Core TypeScript — React Native

## tsconfig.json

```json
{
  "extends": "@react-native/typescript-config/tsconfig.json",
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"],
      "@store/*": ["src/store/*"]
    }
  }
}
```

For Expo projects, extend `expo/tsconfig.base` instead:
```json
{ "extends": "expo/tsconfig.base" }
```

## Component Typing

```tsx
// Prefer explicit props interface over React.FC
interface ButtonProps {
  label: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  style?: StyleProp<ViewStyle>;
}

// React.FC infers children and return type — often unnecessary
// Direct function declaration is simpler and avoids React.FC pitfalls
export function Button({ label, onPress, variant = 'primary', disabled, style }: ButtonProps) {
  return (
    <Pressable
      onPress={onPress}
      disabled={disabled}
      style={[styles.base, styles[variant], style]}
    >
      <Text style={styles.label}>{label}</Text>
    </Pressable>
  );
}
```

## StyleSheet Typing

```ts
import { StyleSheet, type ViewStyle, type TextStyle, type ImageStyle } from 'react-native';

type NamedStyles<T> = { [P in keyof T]: ViewStyle | TextStyle | ImageStyle };

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  } satisfies ViewStyle,   // 'satisfies' catches style type errors at compile time
  title: {
    fontSize: 18,
    fontWeight: 'bold',
  } satisfies TextStyle,
});

// StyleProp allows arrays, undefined, null — always use for component style props
import type { StyleProp, ViewStyle } from 'react-native';
interface CardProps { style?: StyleProp<ViewStyle>; }
```

## Navigation Typing

```ts
// navigation/types.ts
import type { NavigatorScreenParams } from '@react-navigation/native';

export type RootStackParamList = {
  Splash: undefined;
  Auth: NavigatorScreenParams<AuthStackParamList>;
  Main: NavigatorScreenParams<MainTabParamList>;
  Modal: { title: string; content: string };
};

export type AuthStackParamList = {
  Login: undefined;
  Register: { email?: string };
  ForgotPassword: undefined;
};

export type MainTabParamList = {
  Home: undefined;
  Search: { query?: string };
  Profile: { userId: string };
  Settings: undefined;
};

// Declare globally so you never need to import types in useNavigation calls
declare global {
  namespace ReactNavigation {
    interface RootParamList extends RootStackParamList {}
  }
}
```

```tsx
// Now useNavigation is automatically typed to RootStackParamList
import { useNavigation } from '@react-navigation/native';

function AnyComponent() {
  const navigation = useNavigation(); // fully typed without generic
  navigation.navigate('Modal', { title: 'Hi', content: 'Hello' }); // type-checked
}
```

## Environment Variables

```ts
// env.d.ts — declare Expo public env vars
declare module 'process' {
  global {
    namespace NodeJS {
      interface ProcessEnv {
        EXPO_PUBLIC_API_URL: string;
        EXPO_PUBLIC_APP_ENV: 'development' | 'staging' | 'production';
      }
    }
  }
}
```

## API Response Validation with Zod

```ts
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1),
  role: z.enum(['user', 'admin']),
  createdAt: z.string().datetime(),
});

export type User = z.infer<typeof UserSchema>;

// In data fetching layer
async function fetchUser(id: string): Promise<User> {
  const { data } = await api.get(`/users/${id}`);
  return UserSchema.parse(data); // throws ZodError if shape is wrong
}

// Safe parse (no throw) — useful for optional data
const result = UserSchema.safeParse(rawData);
if (result.success) {
  console.log(result.data.email);
} else {
  console.error(result.error.issues);
}
```

## Generic List Component

```tsx
interface ListProps<T> {
  data: T[];
  renderItem: (item: T, index: number) => React.ReactElement;
  keyExtractor: (item: T) => string;
  emptyMessage?: string;
}

export function TypedList<T>({ data, renderItem, keyExtractor, emptyMessage }: ListProps<T>) {
  if (data.length === 0) {
    return <Text>{emptyMessage ?? 'No items'}</Text>;
  }
  return (
    <FlatList
      data={data}
      renderItem={({ item, index }) => renderItem(item, index)}
      keyExtractor={keyExtractor}
    />
  );
}

// Usage — T is inferred from data
<TypedList
  data={users}
  renderItem={(user) => <UserCard user={user} />}
  keyExtractor={(user) => user.id}
/>
```

## Common Utility Types

```ts
// Make specific keys required
type WithRequired<T, K extends keyof T> = T & Required<Pick<T, K>>;

// Flatten intersection types for readable hover tooltips
type Prettify<T> = { [K in keyof T]: T[K] } & {};

// All keys optional except specified ones
type PartialExcept<T, K extends keyof T> = Partial<Omit<T, K>> & Pick<T, K>;

// Extract the resolved value of a Promise
type Awaited<T> = T extends Promise<infer U> ? U : T; // built-in since TS 4.5

// Examples
type CreateUserPayload = WithRequired<Partial<User>, 'email' | 'name'>;
type ReadableUser = Prettify<User & { displayName: string }>;
```

## Discriminated Unions for State

```ts
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function ProfileDisplay({ state }: { state: RequestState<User> }) {
  switch (state.status) {
    case 'idle': return null;
    case 'loading': return <ActivityIndicator />;
    case 'success': return <Text>{state.data.name}</Text>; // data is User here
    case 'error': return <Text>{state.error}</Text>;       // error is string here
  }
}
```

## Module Augmentation for Theme

```ts
// theme.d.ts — extend DefaultTheme for React Navigation or styled-components
import '@react-navigation/native';

declare module '@react-navigation/native' {
  export interface Theme {
    colors: {
      primary: string;
      background: string;
      card: string;
      text: string;
      border: string;
      notification: string;
      // custom additions
      surface: string;
      onSurface: string;
    };
  }
}
```
