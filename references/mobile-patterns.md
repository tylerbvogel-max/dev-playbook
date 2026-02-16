# Mobile Patterns Reference

Detailed patterns for the React Native (Expo) mobile client.

---

## Expo Project Setup

### Initialize

```bash
npx create-expo-app@latest my-app --template blank-typescript
cd my-app
```

### Core Dependencies

```bash
# Navigation
npx expo install react-native-screens react-native-safe-area-context
npm install @react-navigation/native @react-navigation/bottom-tabs @react-navigation/native-stack

# Server state
npm install @tanstack/react-query

# Secure storage
npx expo install expo-secure-store

# Fonts (choose your font family)
npx expo install expo-font @expo-google-fonts/space-grotesk

# Charts (if needed)
npm install react-native-chart-kit react-native-svg

# Gestures (if needed)
npx expo install react-native-gesture-handler
```

### TypeScript Config

```json
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

**Strict mode is required.** Catches null/undefined errors, implicit any, and missing return types at build time.

### App Configuration

```json
// app.json
{
  "expo": {
    "name": "App Name",
    "slug": "app-name",
    "version": "1.0.0",
    "orientation": "portrait",
    "userInterfaceStyle": "dark",
    "ios": {
      "supportsTablet": false,
      "bundleIdentifier": "com.yourname.appname"
    },
    "android": {
      "package": "com.yourname.appname"
    },
    "extra": {
      "apiUrl": "https://your-api.onrender.com"
    },
    "eas": {
      "projectId": "your-eas-project-id"
    }
  }
}
```

**Key fields:**
- `extra.apiUrl`: Read by API client — single place to switch between local/production
- `userInterfaceStyle`: Force dark mode globally
- `bundleIdentifier` / `package`: Required for App Store / Play Store

---

## App Root (App.tsx)

### Provider Hierarchy

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { SafeAreaProvider } from "react-native-safe-area-context";
import { NavigationContainer } from "@react-navigation/native";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
      staleTime: 30_000, // 30 seconds before refetch
    },
  },
});

export default function App() {
  const [isReady, setIsReady] = useState(false);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  useEffect(() => {
    // Load stored token on app start
    loadStoredToken().then((hasToken) => {
      setIsAuthenticated(hasToken);
      setIsReady(true);
    });

    // Register global sign-out handler
    registerSignOutHandler(() => setIsAuthenticated(false));
  }, []);

  if (!isReady) return null; // Or splash screen

  return (
    <QueryClientProvider client={queryClient}>
      <SafeAreaProvider>
        <NavigationContainer>
          {isAuthenticated ? <MainTabs /> : <AuthStack />}
        </NavigationContainer>
      </SafeAreaProvider>
    </QueryClientProvider>
  );
}
```

**Order matters:**
1. `QueryClientProvider` — outermost so all screens can use queries
2. `SafeAreaProvider` — handles notch/status bar insets
3. `NavigationContainer` — must wrap all navigators
4. Auth gate — simple conditional rendering
5. Feature contexts — inside authenticated section only

### Font Loading

```tsx
import { useFonts, SpaceGrotesk_400Regular, SpaceGrotesk_700Bold } from "@expo-google-fonts/space-grotesk";
import * as SplashScreen from "expo-splash-screen";

SplashScreen.preventAutoHideAsync();

export default function App() {
  const [fontsLoaded] = useFonts({ SpaceGrotesk_400Regular, SpaceGrotesk_700Bold });

  useEffect(() => {
    if (fontsLoaded) SplashScreen.hideAsync();
  }, [fontsLoaded]);

  if (!fontsLoaded) return null;

  return (/* ... */);
}
```

---

## Navigation

### Tab Navigator

```tsx
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";

const Tab = createBottomTabNavigator();

function MainTabs() {
  return (
    <Tab.Navigator
      screenOptions={{
        headerShown: false,
        tabBarStyle: { backgroundColor: colors.background, borderTopColor: colors.surface },
        tabBarActiveTintColor: colors.primary,
        tabBarInactiveTintColor: colors.textMuted,
      }}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Items" component={ItemsScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
}
```

### Stack Navigator (for nested flows)

```tsx
import { createNativeStackNavigator } from "@react-navigation/native-stack";

const Stack = createNativeStackNavigator();

function ItemsStack() {
  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      <Stack.Screen name="ItemsList" component={ItemsListScreen} />
      <Stack.Screen name="ItemDetail" component={ItemDetailScreen} />
      <Stack.Screen name="CreateItem" component={CreateItemScreen} />
    </Stack.Navigator>
  );
}
```

Use stack navigators inside tabs when a tab has a drill-down flow.

---

## API Client (client.ts)

### Full Template

```typescript
import * as SecureStore from "expo-secure-store";
import Constants from "expo-constants";

const API_BASE = Constants.expoConfig?.extra?.apiUrl ?? "http://localhost:8000";

let authToken: string | null = null;
let signOutHandler: (() => void) | null = null;

// --- Token Management ---

export function setAuthToken(token: string) {
  authToken = token;
}

export async function persistToken(token: string) {
  authToken = token;
  await SecureStore.setItemAsync("auth_token", token);
}

export async function loadStoredToken(): Promise<boolean> {
  const token = await SecureStore.getItemAsync("auth_token");
  if (token) {
    authToken = token;
    return true;
  }
  return false;
}

export async function clearStoredToken() {
  authToken = null;
  await SecureStore.deleteItemAsync("auth_token");
}

export function registerSignOutHandler(handler: () => void) {
  signOutHandler = handler;
}

async function signOut() {
  await clearStoredToken();
  signOutHandler?.();
}

// --- HTTP ---

async function request<T>(path: string, options?: RequestInit): Promise<T> {
  const headers: Record<string, string> = {
    "Content-Type": "application/json",
  };
  if (authToken) {
    headers.Authorization = `Bearer ${authToken}`;
  }

  const response = await fetch(`${API_BASE}${path}`, {
    ...options,
    headers: { ...headers, ...options?.headers },
  });

  if (response.status === 401) {
    await signOut();
    throw new Error("Session expired");
  }

  if (!response.ok) {
    const text = await response.text();
    throw new Error(text || `Request failed: ${response.status}`);
  }

  return response.json();
}

// --- API Domains ---

export const auth = {
  register: (alias: string, inviteCode: string) =>
    request<{ token: string; user: { id: string; alias: string } }>("/auth/register", {
      method: "POST",
      body: JSON.stringify({ alias, invite_code: inviteCode }),
    }),
  login: (alias: string, token: string) =>
    request<{ user: { id: string; alias: string } }>("/auth/login", {
      method: "POST",
      body: JSON.stringify({ alias, token }),
    }),
  me: () => request<{ id: string; alias: string }>("/auth/me"),
};

export const items = {
  list: () => request<Item[]>("/items"),
  get: (id: string) => request<Item>(`/items/${id}`),
  create: (data: CreateItemRequest) =>
    request<Item>("/items", { method: "POST", body: JSON.stringify(data) }),
};

// --- Types ---

export interface Item {
  id: string;
  name: string;
  description: string | null;
  is_active: boolean;
  created_at: string;
}

export interface CreateItemRequest {
  name: string;
  description?: string;
}
```

### Key Design Decisions

- **In-memory + SecureStore**: Token in memory for fast access, persisted for app restarts
- **Global sign-out handler**: Registered from App.tsx, called on 401 — resets entire app state
- **Grouped by domain**: `auth.register()`, `items.list()` — self-documenting, autocomplete-friendly
- **Types co-located**: Keep interfaces in client.ts or a shared `types.ts`

---

## React Query Hooks (useApi.ts)

### Full Template

```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { auth, items } from "../api/client";

// --- Queries ---

export function useProfile() {
  return useQuery({
    queryKey: ["profile"],
    queryFn: () => auth.me(),
  });
}

export function useItems() {
  return useQuery({
    queryKey: ["items"],
    queryFn: () => items.list(),
  });
}

export function useItem(id: string) {
  return useQuery({
    queryKey: ["items", id],
    queryFn: () => items.get(id),
    enabled: !!id, // Don't fetch until we have an ID
  });
}

// --- Mutations ---

export function useCreateItem() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateItemRequest) => items.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["items"] });
    },
  });
}

// --- Live Data ---

export function useLiveData() {
  return useQuery({
    queryKey: ["live-data"],
    queryFn: () => liveEndpoint.get(),
    refetchInterval: 30_000, // Refetch every 30 seconds
  });
}
```

### Query Key Conventions

```
["items"]                    → All items
["items", itemId]            → Single item
["items", { filter: "..." }] → Filtered list
["profile"]                  → Current user
```

Invalidating `["items"]` also invalidates `["items", itemId]` — React Query matches prefixes.

### Usage in Components

```tsx
function ItemsScreen() {
  const { data: items, isLoading, error } = useItems();
  const createItem = useCreateItem();

  if (isLoading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error.message}</Text>;

  return (
    <FlatList
      data={items}
      renderItem={({ item }) => <ItemCard item={item} />}
      keyExtractor={(item) => item.id}
    />
  );
}
```

---

## Context API Patterns

### Auth Context (if needed beyond token)

```tsx
interface AuthContextType {
  user: User | null;
  signIn: (token: string) => Promise<void>;
  signOut: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const signIn = async (token: string) => {
    await persistToken(token);
    const profile = await auth.me();
    setUser(profile);
  };

  const signOut = async () => {
    await clearStoredToken();
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be used within AuthProvider");
  return ctx;
}
```

### Preference/Mode Context

```tsx
type Mode = "default" | "advanced";

const ModeContext = createContext<{ mode: Mode; setMode: (m: Mode) => void }>(undefined!);

export function ModeProvider({ children }: { children: ReactNode }) {
  const [mode, setMode] = useState<Mode>("default");

  // Persist to AsyncStorage
  useEffect(() => {
    AsyncStorage.getItem("app_mode").then((m) => m && setMode(m as Mode));
  }, []);

  const updateMode = (m: Mode) => {
    setMode(m);
    AsyncStorage.setItem("app_mode", m);
  };

  return (
    <ModeContext.Provider value={{ mode, setMode: updateMode }}>
      {children}
    </ModeContext.Provider>
  );
}
```

### Rules

- Keep contexts small and focused (one concern per context)
- Always provide a custom hook (`useAuth()`, `useMode()`) with a missing-provider error
- Persist user preferences to AsyncStorage, auth tokens to SecureStore
- Don't use context for server state — that's React Query's job

---

## Screen Patterns

### Standard List Screen

```tsx
function ItemsScreen({ navigation }) {
  const { data, isLoading, error, refetch } = useItems();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Items</Text>
      {isLoading ? (
        <ActivityIndicator color={colors.primary} />
      ) : error ? (
        <Text style={styles.error}>Failed to load</Text>
      ) : (
        <FlatList
          data={data}
          keyExtractor={(item) => item.id}
          renderItem={({ item }) => (
            <TouchableOpacity
              style={styles.card}
              onPress={() => navigation.navigate("ItemDetail", { id: item.id })}
            >
              <Text style={styles.cardTitle}>{item.name}</Text>
            </TouchableOpacity>
          )}
          refreshing={isLoading}
          onRefresh={refetch}
          ListEmptyComponent={<Text style={styles.empty}>No items yet</Text>}
        />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: colors.background, padding: spacing.lg },
  title: { fontSize: fontSize.xl, color: colors.text, fontFamily: "SpaceGrotesk_700Bold" },
  card: { backgroundColor: colors.card, borderRadius: radius.md, padding: spacing.lg, marginBottom: spacing.sm },
  cardTitle: { fontSize: fontSize.md, color: colors.text },
  error: { color: colors.error, textAlign: "center" },
  empty: { color: colors.textSecondary, textAlign: "center", marginTop: spacing.xxl },
});
```

### Standard Form Screen

```tsx
function CreateItemScreen({ navigation }) {
  const [name, setName] = useState("");
  const createItem = useCreateItem();

  const handleSubmit = async () => {
    try {
      await createItem.mutateAsync({ name });
      navigation.goBack();
    } catch (e) {
      Alert.alert("Error", e.message);
    }
  };

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="Item name"
        placeholderTextColor={colors.textMuted}
        value={name}
        onChangeText={setName}
      />
      <TouchableOpacity
        style={[styles.button, createItem.isPending && styles.buttonDisabled]}
        onPress={handleSubmit}
        disabled={createItem.isPending || !name.trim()}
      >
        <Text style={styles.buttonText}>
          {createItem.isPending ? "Creating..." : "Create"}
        </Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Conventions

- Always handle loading, error, and empty states
- Use `FlatList` with `onRefresh` for pull-to-refresh
- Disable buttons during mutations (`isPending`)
- Use `Alert.alert` for error feedback
- Navigation: `goBack()` after successful creation/update
- All styles use theme tokens — no inline magic numbers
