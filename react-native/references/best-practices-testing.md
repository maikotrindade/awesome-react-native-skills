# Best Practices — Testing

## Stack

| Layer | Tool |
|-------|------|
| Unit / component | Jest + React Native Testing Library (RNTL) |
| Integration | Jest + RNTL with real navigation/store |
| E2E | Maestro |

---

## Jest Setup

### Expo projects
```bash
npx expo install jest-expo jest
```

`jest.config.js`:
```js
module.exports = {
  preset: 'jest-expo',
  setupFilesAfterFramework: ['@testing-library/jest-native/extend-expect'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@unimodules/.*|unimodules|sentry-expo|native-base|react-native-svg))',
  ],
  collectCoverageFrom: ['src/**/*.{ts,tsx}', '!src/**/*.d.ts'],
  coverageThreshold: {
    global: { branches: 70, functions: 70, lines: 70, statements: 70 },
  },
};
```

### Bare React Native
```bash
npm install --save-dev jest @testing-library/react-native @testing-library/jest-native
```

`jest.config.js`:
```js
module.exports = {
  preset: '@react-native/jest-preset',
  setupFilesAfterFramework: ['@testing-library/jest-native/extend-expect'],
};
```

---

## React Native Testing Library

### Query Priority (use in this order)

1. `getByRole` — most accessible, matches what screen readers see
2. `getByLabelText` — form elements with labels
3. `getByPlaceholderText`
4. `getByText`
5. `getByDisplayValue`
6. `getByTestId` — last resort, brittle

```tsx
import { render, screen } from '@testing-library/react-native';

test('shows user name', () => {
  render(<ProfileCard user={{ name: 'Maiko', email: 'a@b.com' }} />);
  expect(screen.getByText('Maiko')).toBeOnTheScreen();
  expect(screen.getByRole('button', { name: 'Edit' })).toBeOnTheScreen();
});
```

### userEvent (preferred over fireEvent)

`userEvent` simulates real user interactions including event propagation:

```tsx
import { render, screen } from '@testing-library/react-native';
import userEvent from '@testing-library/user-event';

test('submits login form', async () => {
  const user = userEvent.setup();
  render(<LoginForm onSubmit={mockSubmit} />);

  await user.type(screen.getByPlaceholderText('Email'), 'test@example.com');
  await user.type(screen.getByPlaceholderText('Password'), 'secret123');
  await user.press(screen.getByRole('button', { name: 'Sign In' }));

  expect(mockSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'secret123',
  });
});
```

### Async Queries

```tsx
import { render, screen, waitFor } from '@testing-library/react-native';

test('loads and displays data', async () => {
  render(<UserList />);

  // findBy* = getBy + waitFor
  const item = await screen.findByText('Maiko');
  expect(item).toBeOnTheScreen();

  // waitFor — poll until expectation passes
  await waitFor(() => {
    expect(screen.queryByText('Loading...')).not.toBeOnTheScreen();
  });
});
```

---

## Mocking Native Modules

```ts
// __mocks__/react-native-mmkv.ts
export const MMKV = jest.fn().mockImplementation(() => ({
  set: jest.fn(),
  getString: jest.fn().mockReturnValue(undefined),
  getNumber: jest.fn().mockReturnValue(undefined),
  getBoolean: jest.fn().mockReturnValue(undefined),
  delete: jest.fn(),
  contains: jest.fn().mockReturnValue(false),
  getAllKeys: jest.fn().mockReturnValue([]),
}));
```

```ts
// __mocks__/@react-native-async-storage/async-storage.ts
import mockStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';
export default mockStorage;
```

```ts
// __mocks__/@react-navigation/native.ts — navigation testing
const mockNavigate = jest.fn();
const mockGoBack = jest.fn();

jest.mock('@react-navigation/native', () => ({
  ...jest.requireActual('@react-navigation/native'),
  useNavigation: () => ({ navigate: mockNavigate, goBack: mockGoBack }),
  useRoute: () => ({ params: {} }),
  useFocusEffect: jest.fn(),
}));
```

---

## Testing with Zustand

```ts
import { act, renderHook } from '@testing-library/react-native';
import { useCounterStore } from '../store/useCounterStore';

beforeEach(() => {
  useCounterStore.setState({ count: 0 }); // reset before each test
});

test('increments counter', () => {
  const { result } = renderHook(() => useCounterStore());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
});
```

---

## Testing with TanStack Query

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { render } from '@testing-library/react-native';

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
      mutations: { retry: false },
    },
    logger: { log: console.log, warn: console.warn, error: () => {} },
  });
}

function renderWithQuery(ui: React.ReactElement) {
  const queryClient = createTestQueryClient();
  return {
    ...render(
      <QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>
    ),
    queryClient,
  };
}
```

---

## Testing Navigation

```tsx
import { NavigationContainer } from '@react-navigation/native';

function renderWithNavigation(ui: React.ReactElement) {
  return render(<NavigationContainer>{ui}</NavigationContainer>);
}

test('navigates to detail on press', async () => {
  const user = userEvent.setup();
  renderWithNavigation(<HomeStack />);

  await user.press(screen.getByText('First Post'));
  expect(await screen.findByText('Post Detail')).toBeOnTheScreen();
});
```

---

## Snapshot Testing

Use sparingly — snapshot tests verify structure but fail on any change, generating noise:

```tsx
// Only for stable, presentational components
test('renders button correctly', () => {
  const { toJSON } = render(<Button label="Save" onPress={() => {}} />);
  expect(toJSON()).toMatchSnapshot();
});
```

---

## E2E with Maestro

Maestro uses YAML flow files — no JavaScript, no Appium.

```bash
# Install CLI
curl -Ls "https://get.maestro.mobile.dev" | bash

# Run a flow
maestro test flows/login.yaml
```

`flows/login.yaml`:
```yaml
appId: com.example.myapp
---
- launchApp
- assertVisible: "Welcome back"
- tapOn: "Email"
- inputText: "test@example.com"
- tapOn: "Password"
- inputText: "secret123"
- tapOn:
    text: "Sign In"
- assertVisible: "Home"
- takeScreenshot: "login-success"
```

Run flows in CI:
```yaml
# .github/workflows/e2e.yml
- name: Run Maestro E2E
  run: maestro test flows/ --format junit --output report.xml
```
