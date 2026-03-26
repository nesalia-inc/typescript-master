# Deep Readonly and Deep Partial

Recursive utility types for nested object transformations.

## Deep Readonly

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends Function
    ? T[P]
    : T[P] extends object
      ? DeepReadonly<T[P]>
      : T[P];
};
```

## Deep Partial

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends Function
    ? T[P]
    : T[P] extends object
      ? T[P] extends Array<infer U>
        ? Array<DeepPartial<U>>
        : DeepPartial<T[P]>
      : T[P];
};
```

## Deep Required

```typescript
type DeepRequired<T> = {
  [P in keyof T]-?: T[P] extends Function
    ? T[P]
    : T[P] extends object
      ? DeepRequired<T[P]>
      : T[P];
};
```

## Deep Mutable (Remove Readonly)

```typescript
type DeepMutable<T> = {
  -readonly [P in keyof T]: T[P] extends Function
    ? T[P]
    : T[P] extends object
      ? DeepMutable<T[P]>
      : T[P];
};
```

## Usage Example

```typescript
interface Config {
  server: {
    host: string;
    port: number;
    ssl: {
      enabled: boolean;
      cert: string;
    };
  };
  database: {
    url: string;
    pool: {
      min: number;
      max: number;
    };
  };
  features: string[];
}

// All nested properties are readonly
type ReadonlyConfig = DeepReadonly<Config>;
const config: ReadonlyConfig = {
  server: { host: "localhost", port: 8080, ssl: { enabled: true, cert: "..." } },
  database: { url: "postgres://...", pool: { min: 2, max: 10 } },
  features: ["auth", "logging"],
};

// Error: Cannot assign to 'host' because it is a read-only property
config.server.host = "0.0.0.0"; // Error!

// Error: Cannot assign to 'enabled' because it is a read-only property
config.server.ssl.enabled = false; // Error!

// All nested properties are optional
type PartialConfig = DeepPartial<Config>;
const partial: PartialConfig = {
  server: {
    ssl: {
      cert: "...",
      // host and port are optional
    },
  },
};
```

## Array Handling

The `DeepPartial` above handles arrays:

```typescript
type DeepPartialArr<T> = {
  [P in keyof T]?: T[P] extends Array<infer U>
    ? Array<DeepPartial<U>>
    : T[P] extends object
      ? DeepPartial<T[P]>
      : T[P];
};

interface ApiConfig {
  endpoints: Array<{ url: string; timeout: number }>;
}

type PartialApiConfig = DeepPartial<ApiConfig>;
// {
//   endpoints?: Array<{
//     url?: string;
//     timeout?: number;
//   }>;
// }
```

## Without Function Type Special-Casing

If you want functions to also be made partial (usually not desired):

```typescript
type NaiveDeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? T[P] extends Array<infer U>
      ? Array<NaiveDeepPartial<U>>
      : NaiveDeepPartial<T[P]>
    : T[P];
};
```

## Key Techniques

1. **Recursive conditional** — `T[P] extends object ? Recursive<T[P]> : T[P]`
2. **Array special-casing** — `T[P] extends Array<infer U> ? Array<Recursive<U>> : ...`
3. **Function preservation** — Functions should usually stay as-is, not become partial
4. **Readonly preservation** — Nested readonly is maintained

## Performance Note

Deep recursive types can slow down compilation in large projects. Use sparingly and consider caching complex types if needed.
