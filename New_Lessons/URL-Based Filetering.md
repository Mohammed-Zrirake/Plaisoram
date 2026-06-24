# URL-Based Filtering & Debouncing

This document describes how state is managed for query filters, search inputs, and page states in the **Plaisoram** Next.js dashboard using **URL Search Parameters** coupled with a custom **`useDebounce`** hook.

---

## 1. Architectural Philosophy

Managing state directly in the URL search parameters (e.g. `?query=default&layout=main`) is a Next.js best practice that delivers several core benefits:

- **Bookmarkable / Shareable**: Users can copy the URL and share it with others. When someone loads the link, they see the exact same filtered state.
- **Initial Server-Side Sync**: If components are loaded on the server, search params are immediately available without waiting for hydrate-mount.
- **History Navigation**: Standard back/forward browser buttons automatically navigate between filter states.

---

## 2. The Custom `useDebounce` Hook

A key issue with search inputs is that trigger functions (like API fetches or URL updates) fire on **every keystroke**. We solve this by implementing a **debounce delay**.

The custom hook `useDebounce` (found at `plaisoram_web/src/hooks/useDebounce.ts`) delays propagating value updates:

```typescript
import { useEffect, useState } from "react";

export function useDebounce<T>(value: T, delay: number = 300): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

---

## 3. How It Is Integrated (Example: Playlists Page)

Inside `src/app/(dashboard)/playlists/page.tsx`, we bind filter fields to both React state and URL search parameters:

### Client Navigation Hooks
We import the following navigation hooks from `next/navigation`:
- `useSearchParams`: To read parameters from the current URL.
- `usePathname`: To read the base path (`/playlists`).
- `useRouter`: To trigger page routing replacements without a full page refresh.

### Controlled States & Syncing
We instantiate the local inputs with default values from the URL, then use the debounced value to replace the URL route:

```typescript
const searchParams = useSearchParams();
const pathname = usePathname();
const router = useRouter();

// 1. Instantiated state from current URL params
const [searchQuery, setSearchQuery] = useState(searchParams.get("query") || "");
const [selectedLayout, setSelectedLayout] = useState(searchParams.get("layout") || "");

// 2. Debounce the query string by 300ms
const debouncedQuery = useDebounce(searchQuery, 300);

// 3. Write updates to the URL Search Params
useEffect(() => {
  const params = new URLSearchParams(searchParams.toString());
  
  if (debouncedQuery) {
    params.set("query", debouncedQuery);
  } else {
    params.delete("query");
  }

  if (selectedLayout) {
    params.set("layout", selectedLayout);
  } else {
    params.delete("layout");
  }

  router.replace(`${pathname}?${params.toString()}`);
}, [debouncedQuery, selectedLayout, pathname, router, searchParams]);
```

### In-Memory Filtering
By filtering the local data array using parameters read directly from the URL, we guarantee that input synchronization is both **lightning-fast** and **highly responsive**:

```typescript
const filteredPlaylists = playlists.filter(playlist => {
  const queryParam = searchParams.get("query") || "";
  const layoutParam = searchParams.get("layout") || "";

  const matchesQuery = playlist.name.toLowerCase().includes(queryParam.toLowerCase());
  const matchesLayout = !layoutParam || playlist.layoutType === layoutParam;

  return matchesQuery && matchesLayout;
});
```

---

## 4. UI Bindings

Filter inputs are simply bound to the React state. For example:
- **Search input**: `<input value={searchQuery} onChange={e => setSearchQuery(e.target.value)} />`
- **Layout dropdown**: populated dynamically from `DEFAULT_LAYOUTS` and bound to `selectedLayout`.
