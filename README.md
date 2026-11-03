# The Capsule

### Keep your data close. Keep your applications fast.

**The Capsule** is a lightweight, reusable persistent caching library built on IndexedDB for JavaScript and TypeScript applications.

It simplifies browser-side data caching with a clean API for storing, retrieving, managing, and preloading cached data without repeatedly writing low-level IndexedDB operations.

Designed with a framework-independent core and separate framework integrations, The Capsule helps developers reduce unnecessary API requests and manage persistent client-side data more easily.

## Features

- **Persistent storage:** Store data using browser IndexedDB.
- **Simple cache API:** Get, set, check, remove, and clear cached entries.
- **TTL-based expiration:** Automatically treat expired entries as cache misses.
- **Cache-first fetching:** Retrieve cached data or fetch and store it when unavailable.
- **Request deduplication:** Prevent duplicate concurrent fetches for the same key through `getOrFetch()`.
- **Preloading:** Preload multiple items using the existing preload utility.
- **Prefix-based invalidation:** Remove related cache entries using key prefixes.
- **Configurable storage:** Configure database name, object store name, and database version.
- **Database recovery:** Handle missing object stores, concurrent database initialization, and version-change events.
- **React integration:** Separate React adapter for integrating caching and preloading into React applications.
- **TypeScript support:** Type definitions for a better development experience.

## Installation

```bash
npm install the-capsule
```

## Quick Start

### 1. Create a cache instance

```typescript
import { IndexedDBCache } from "the-capsule";

const cache = new IndexedDBCache({
  dbName: "my-app-cache",
  storeName: "api-data",
  version: 1,
});
```

### 2. Store data

```typescript
await cache.set("user:101", {
  id: 101,
  name: "John",
  email: "john@example.com",
});
```

### 3. Retrieve data

```typescript
const user = await cache.get("user:101");

console.log(user);
```

### 4. Check whether data exists

```typescript
const exists = await cache.has("user:101");

console.log(exists);
```

## API Reference

### `IndexedDBCache`

Creates a cache instance backed by IndexedDB.

```typescript
const cache = new IndexedDBCache({
  dbName: "my-app-cache",
  storeName: "api-data",
  version: 1,
});
```

| Option | Type | Description |
|---|---|---|
| `dbName` | `string` | IndexedDB database name |
| `storeName` | `string` | Object store name |
| `version` | `number` | Database version |

### Cache Operations

#### `get(key)`

Retrieves a cached value. Expired or unavailable entries are treated as cache misses.

```typescript
const data = await cache.get("products");
```

#### `set(key, value, options?)`

Stores a value in IndexedDB.

```typescript
await cache.set("products", productList, {
  ttl: 60000,
});
```

TTL is specified in milliseconds.

#### `has(key)`

Checks whether a valid cached entry exists.

```typescript
const exists = await cache.has("products");
```

#### `remove(key)`

Removes a specific cache entry.

```typescript
await cache.remove("products");
```

#### `removeByPrefix(prefix)`

Removes entries whose keys match a specified prefix.

```typescript
await cache.removeByPrefix("user:");
```

Useful for invalidating related entries without manually removing each key.

#### `clear()`

Clears the configured object store.

```typescript
await cache.clear();
```

### Cache-First Fetching

#### `getOrFetch(key, fetcher, options?)`

Retrieves data from the cache when available. If the data is unavailable or expired, executes the provided fetcher and stores the successfully fetched result.

```typescript
const products = await cache.getOrFetch(
  "products",
  async () => {
    const response = await fetch("/api/products");

    if (!response.ok) {
      throw new Error("Failed to fetch products");
    }

    return response.json();
  },
  {
    ttl: 5 * 60 * 1000,
  }
);
```

**Benefits:**

- Avoids unnecessary API requests when valid cached data exists.
- Automatically stores successfully fetched results.
- Deduplicates concurrent requests for the same cache key.
- Propagates fetch errors instead of treating failed requests as successful cache entries.

### Preloading

The Capsule provides a preload utility for fetching and caching multiple items.

```typescript
import {
  IndexedDBCache,
  preload,
} from "the-capsule";

const cache = new IndexedDBCache({
  dbName: "my-app-cache",
  storeName: "test-data",
  version: 1,
});

const items = [
  {
    key: "test:101",
    fetcher: () => fetchTestData("101"),
    ttl: 60000,
  },
  {
    key: "test:102",
    fetcher: () => fetchTestData("102"),
    ttl: 60000,
  },
];

await preload(cache, items);
```

Preloading is useful when applications need reference data or frequently accessed resources available before users navigate to particular screens.

## React Integration

The React adapter is available through a separate entry point.

```typescript
import {
  PreloadProvider,
} from "the-capsule/react";
```

The provider can be used to integrate preload behavior with React application routes.

Example integration:

```tsx
import { PreloadProvider } from "the-capsule/react";

function App() {
  return (
    <PreloadProvider>
      <ApplicationRoutes />
    </PreloadProvider>
  );
}
```

Refer to the React adapter's exported types and implementation for its supported configuration and usage requirements.

The core package can be used independently without importing the React adapter.

## Suggested Cache Key Conventions

Use predictable cache keys to make cache management and invalidation easier.

| Data | Suggested key |
|---|---|
| User details | `user:101` |
| Product details | `product:101` |
| Product list | `products:list` |
| Test details | `test:101` |
| Location-specific data | `test:101:location:BLR` |
| Application settings | `settings:application` |

For applications with multiple users, tenants, or locations, include the appropriate context in cache keys to avoid accidentally reusing data across contexts.

## Error Handling

Cache and fetch operations are asynchronous. Use `try/catch` to handle errors.

```typescript
try {
  const data = await cache.getOrFetch(
    "products",
    fetchProducts
  );

  console.log(data);
} catch (error) {
  console.error("Unable to retrieve products:", error);
}
```

A failed fetch should be handled by the consuming application. Do not assume that a failed network request has produced a valid cached result.

## Testing

The project uses:

- **Vitest** for unit testing.
- **fake-indexeddb** for simulating IndexedDB in tests.

Run the test suite:

```bash
npm test
```

Run tests in watch mode:

```bash
npm run test:watch
```

Build the package:

```bash
npm run build
```

## Project Structure

The library separates core caching functionality from framework-specific integrations.

```text
the-capsule/
├── src/
│   ├── core/
│   │   ├── cache.ts
│   │   ├── database.ts
│   │   ├── type.ts
│   │   └── errors.ts
│   ├── features/
│   │   ├── preload.ts
│   │   └── polling.ts
│   ├── react/
│   │   ├── PreloadProvider.tsx
│   │   └── index.ts
│   ├── index.ts
│   └── react.ts
├── tests/
├── package.json
├── tsconfig.json
└── vite.config.ts
```

## Browser Compatibility

The Capsule uses IndexedDB, which is supported by modern browsers.

Availability and storage persistence are subject to browser capabilities, storage quotas, and browser eviction policies.

## Contributing

Contributions, bug reports, and suggestions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Implement your changes.
4. Add or update tests.
5. Run the test suite.
6. Submit a pull request.

Please include relevant test cases when introducing new functionality.

## License

MIT License. See the `LICENSE` file for details.

---

**The Capsule — Keep your data close. Keep your applications fast.**
