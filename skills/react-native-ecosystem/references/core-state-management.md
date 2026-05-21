# Core State Management

## Decision Guide

| Library | Use for |
|---------|---------|
| **Zustand** | Client/UI state — simple, minimal boilerplate, great for most apps |
| **Jotai** | Atomic state — fine-grained subscriptions, derived state, lazy atoms |
| **Redux Toolkit** | Complex state — large teams, strict patterns, RTK Query integration |
| **TanStack Query** | Server/async state — see `core-data-fetching.md` |

Rule: **never put server state in Zustand/Jotai/Redux**. Use TanStack Query for anything fetched from an API.

---

## Zustand

### Installation
```bash
npm install zustand
```

### Basic Store
```ts
// store/useCounterStore.ts
import { create } from 'zustand';

interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```

### Slices Pattern (large stores)
```ts
// store/slices/authSlice.ts
import type { StateCreator } from 'zustand';

export interface AuthSlice {
  user: User | null;
  token: string | null;
  setUser: (user: User, token: string) => void;
  logout: () => void;
}

export const createAuthSlice: StateCreator<AuthSlice> = (set) => ({
  user: null,
  token: null,
  setUser: (user, token) => set({ user, token }),
  logout: () => set({ user: null, token: null }),
});

// store/useAppStore.ts
import { create } from 'zustand';
import { createAuthSlice, type AuthSlice } from './slices/authSlice';
import { createSettingsSlice, type SettingsSlice } from './slices/settingsSlice';

type AppStore = AuthSlice & SettingsSlice;

export const useAppStore = create<AppStore>((...args) => ({
  ...createAuthSlice(...args),
  ...createSettingsSlice(...args),
}));
```

### Persist Middleware with MMKV
```ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

const mmkvStorage = {
  getItem: (key: string) => storage.getString(key) ?? null,
  setItem: (key: string, value: string) => storage.set(key, value),
  removeItem: (key: string) => storage.delete(key),
};

export const useSettingsStore = create(
  persist(
    (set) => ({
      theme: 'light' as 'light' | 'dark',
      language: 'en',
      setTheme: (theme: 'light' | 'dark') => set({ theme }),
    }),
    {
      name: 'settings-storage',
      storage: createJSONStorage(() => mmkvStorage),
    }
  )
);
```

### Devtools Middleware
```ts
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

const useStore = create(
  devtools(
    (set) => ({ count: 0, increment: () => set((s) => ({ count: s.count + 1 })) }),
    { name: 'CounterStore' }
  )
);
```

### Selecting State (prevent re-renders)
```tsx
// Good — component only re-renders when `count` changes
const count = useCounterStore((state) => state.count);

// Bad — component re-renders on any store change
const { count, increment } = useCounterStore();

// Stable action selector — actions never change, safe to select all at once
const { increment, decrement } = useCounterStore((state) => ({
  increment: state.increment,
  decrement: state.decrement,
}));
```

---

## Jotai

### Installation
```bash
npm install jotai
```

### Basic Atoms
```ts
// atoms/userAtom.ts
import { atom } from 'jotai';

export const userAtom = atom<User | null>(null);
export const isLoggedInAtom = atom((get) => get(userAtom) !== null); // derived, read-only
```

### Using Atoms in Components
```tsx
import { useAtom, useAtomValue, useSetAtom } from 'jotai';

function ProfileHeader() {
  const user = useAtomValue(userAtom);          // read-only
  const setUser = useSetAtom(userAtom);          // write-only (no subscription)
  const [theme, setTheme] = useAtom(themeAtom); // read + write
}
```

### Async Atoms
```ts
import { atom } from 'jotai';

const userIdAtom = atom<string>('1');

// Async derived atom — suspends until resolved
export const userDataAtom = atom(async (get) => {
  const id = get(userIdAtom);
  const res = await fetch(`/api/users/${id}`);
  return res.json() as Promise<User>;
});
```

### Persist with AsyncStorage
```ts
import { atomWithStorage } from 'jotai/utils';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const themeAtom = atomWithStorage<'light' | 'dark'>(
  'theme',
  'light',
  {
    getItem: async (key) => (await AsyncStorage.getItem(key)) as 'light' | 'dark' | null ?? 'light',
    setItem: (key, value) => AsyncStorage.setItem(key, value),
    removeItem: (key) => AsyncStorage.removeItem(key),
  }
);
```

---

## Redux Toolkit

### Installation
```bash
npm install @reduxjs/toolkit react-redux
```

### Store Setup
```ts
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import cartReducer from './slices/cartSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    cart: cartReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Typed Hooks
```ts
// store/hooks.ts
import { useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
```

### Slice
```ts
// store/slices/cartSlice.ts
import { createSlice, type PayloadAction } from '@reduxjs/toolkit';

interface CartItem { id: string; quantity: number; }
interface CartState { items: CartItem[]; }

const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] } as CartState,
  reducers: {
    addItem(state, action: PayloadAction<CartItem>) {
      const existing = state.items.find((i) => i.id === action.payload.id);
      if (existing) existing.quantity += action.payload.quantity;
      else state.items.push(action.payload);
    },
    removeItem(state, action: PayloadAction<string>) {
      state.items = state.items.filter((i) => i.id !== action.payload);
    },
  },
});

export const { addItem, removeItem } = cartSlice.actions;
export default cartSlice.reducer;
```

### createAsyncThunk
```ts
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk('auth/fetchUser', async (userId: string, { rejectWithValue }) => {
  try {
    const res = await fetch(`/api/users/${userId}`);
    if (!res.ok) return rejectWithValue('Not found');
    return res.json() as Promise<User>;
  } catch (err) {
    return rejectWithValue((err as Error).message);
  }
});

// In slice extraReducers
builder
  .addCase(fetchUser.pending, (state) => { state.loading = true; })
  .addCase(fetchUser.fulfilled, (state, action) => { state.user = action.payload; state.loading = false; })
  .addCase(fetchUser.rejected, (state, action) => { state.error = action.payload as string; state.loading = false; });
```
