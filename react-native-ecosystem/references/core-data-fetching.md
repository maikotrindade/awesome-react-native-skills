# Core Data Fetching — TanStack Query v5 + Axios

TanStack Query (React Query) v5 is the standard for server state in React Native. Pair it with Axios for a robust data layer.

## Installation
```bash
npm install @tanstack/react-query axios
npm install @tanstack/react-query-devtools  # optional, for debugging
```

## QueryClient Setup

```tsx
// lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,    // 5 minutes
      gcTime: 1000 * 60 * 10,      // 10 minutes (was cacheTime in v4)
      retry: 2,
      refetchOnWindowFocus: false, // mobile apps don't have window focus
      refetchOnReconnect: true,
    },
    mutations: {
      retry: 0,
    },
  },
});

// App.tsx
import { QueryClientProvider } from '@tanstack/react-query';

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* rest of app */}
    </QueryClientProvider>
  );
}
```

## Axios Instance

```ts
// lib/api.ts
import axios from 'axios';
import { queryClient } from './queryClient';

export const api = axios.create({
  baseURL: process.env.EXPO_PUBLIC_API_URL ?? 'https://api.example.com',
  timeout: 10_000,
  headers: { 'Content-Type': 'application/json' },
});

// Request interceptor — attach auth token
api.interceptors.request.use((config) => {
  const token = getAuthToken(); // from MMKV / SecureStore
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Response interceptor — handle 401 token refresh
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      try {
        const newToken = await refreshToken();
        error.config.headers.Authorization = `Bearer ${newToken}`;
        return api.request(error.config);
      } catch {
        queryClient.clear();
        navigateToLogin();
      }
    }
    return Promise.reject(error);
  }
);
```

## Query Keys — Factory Pattern

```ts
// queries/userKeys.ts
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};
```

## useQuery

```tsx
import { useQuery } from '@tanstack/react-query';
import { api } from '../lib/api';
import { userKeys } from '../queries/userKeys';

async function fetchUser(id: string): Promise<User> {
  const { data } = await api.get<User>(`/users/${id}`);
  return data;
}

function ProfileScreen({ userId }: { userId: string }) {
  const { data: user, isPending, isError, error } = useQuery({
    queryKey: userKeys.detail(userId),
    queryFn: () => fetchUser(userId),
    enabled: !!userId, // only run when userId is truthy
  });

  if (isPending) return <ActivityIndicator />;
  if (isError) return <Text>Error: {error.message}</Text>;

  return <Text>{user.name}</Text>;
}
```

## useMutation

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

async function updateUser(payload: UpdateUserPayload): Promise<User> {
  const { data } = await api.patch<User>(`/users/${payload.id}`, payload);
  return data;
}

function EditProfileForm({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: updateUser,
    onSuccess: (updatedUser) => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: userKeys.detail(userId) });
      // Or update cache directly (optimistic-style)
      queryClient.setQueryData(userKeys.detail(userId), updatedUser);
    },
    onError: (error) => {
      Alert.alert('Error', error.message);
    },
  });

  return (
    <Button
      title="Save"
      disabled={mutation.isPending}
      onPress={() => mutation.mutate({ id: userId, name: 'New Name' })}
    />
  );
}
```

## Optimistic Updates

```ts
const mutation = useMutation({
  mutationFn: toggleLike,
  onMutate: async (postId) => {
    await queryClient.cancelQueries({ queryKey: postKeys.detail(postId) });
    const previous = queryClient.getQueryData(postKeys.detail(postId));

    queryClient.setQueryData(postKeys.detail(postId), (old: Post) => ({
      ...old,
      liked: !old.liked,
      likeCount: old.liked ? old.likeCount - 1 : old.likeCount + 1,
    }));

    return { previous }; // returned as context
  },
  onError: (_err, postId, context) => {
    queryClient.setQueryData(postKeys.detail(postId), context?.previous);
  },
  onSettled: (_, __, postId) => {
    queryClient.invalidateQueries({ queryKey: postKeys.detail(postId) });
  },
});
```

## useInfiniteQuery + FlatList

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';

async function fetchPosts({ pageParam = 1 }: { pageParam: number }) {
  const { data } = await api.get<{ posts: Post[]; nextPage: number | null }>(
    `/posts?page=${pageParam}&limit=20`
  );
  return data;
}

function PostsFeed() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isPending,
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    initialPageParam: 1,
    getNextPageParam: (lastPage) => lastPage.nextPage ?? undefined,
  });

  const posts = data?.pages.flatMap((page) => page.posts) ?? [];

  return (
    <FlatList
      data={posts}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <PostCard post={item} />}
      onEndReached={() => { if (hasNextPage) fetchNextPage(); }}
      onEndReachedThreshold={0.5}
      ListFooterComponent={isFetchingNextPage ? <ActivityIndicator /> : null}
    />
  );
}
```

## Offline Support with NetInfo

```ts
// lib/networkObserver.ts
import NetInfo from '@react-native-community/netinfo';
import { onlineManager } from '@tanstack/react-query';

// TanStack Query will pause/resume queries based on network state
onlineManager.setEventListener((setOnline) => {
  return NetInfo.addEventListener((state) => {
    setOnline(!!state.isConnected);
  });
});
```

## Prefetching

```ts
// Prefetch data before navigating
async function handleNavToProfile(userId: string) {
  await queryClient.prefetchQuery({
    queryKey: userKeys.detail(userId),
    queryFn: () => fetchUser(userId),
    staleTime: 1000 * 60, // skip if fresh within 1 min
  });
  navigation.navigate('Profile', { userId });
}
```

## Error Boundaries

```tsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function ScreenWithQuery() {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ resetErrorBoundary }) => (
            <View>
              <Text>Something went wrong.</Text>
              <Button onPress={resetErrorBoundary} title="Retry" />
            </View>
          )}
        >
          <Suspense fallback={<ActivityIndicator />}>
            <DataComponent />
          </Suspense>
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}
```
