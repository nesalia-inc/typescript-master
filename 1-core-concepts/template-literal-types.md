# Template Literal Types

Create string-based types with pattern matching and transformation.

## Basic Template Literal

```typescript
type EventName = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventName>}`;
// Type: "onClick" | "onFocus" | "onBlur"
```

## Built-in String Manipulation

```typescript
type UppercaseGreeting = Uppercase<"hello">;      // "HELLO"
type LowercaseGreeting = Lowercase<"HELLO">;      // "hello"
type CapitalizedName = Capitalize<"john">;        // "John"
type UncapitalizedName = Uncapitalize<"John">;    // "john"
```

## Combining Mapped Types with Template Literals

```typescript
type PropEventSource<T> = {
  on<K extends keyof T & string>(
    eventName: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void;
};

interface Person {
  name: string;
  age: number;
}

declare const source: PropEventSource<Person>;

// Type-safe: listens to "nameChanged" | "ageChanged"
source.on("nameChanged", (newName) => {
  console.log(newName.toUpperCase()); // Type: string
});

source.on("ageChanged", (newAge) => {
  console.log(newAge.toFixed(2)); // Type: number
});

// Error: "heightChanged" doesn't exist
source.on("heightChanged", (h) => {}); // Error!
```

## Path Building

```typescript
type Path<T> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? `${K}` | `${K}.${Path<T[K]>}`
        : never;
    }[keyof T]
  : never;

interface Config {
  server: {
    host: string;
    port: number;
  };
  database: {
    url: string;
  };
}

type ConfigPath = Path<Config>;
// Type: "server" | "server.host" | "server.port" | "database" | "database.url"
```

## Parsing String Types

```typescript
type ParsePath<P extends string> = P extends `${infer T}.${infer Rest}`
  ? [T, ...ParsePath<Rest>]
  : [P];

type Parts = ParsePath<"server.host.port">;
// Type: ["server", "host", "port"]
```

## Extracting HTTP Method and Path

```typescript
type ExtractMethod<P extends string> = P extends `${infer M} ${string}`
  ? M extends "GET" | "POST" | "PUT" | "DELETE"
    ? M
    : never
  : never;

type Method = ExtractMethod<"GET /users">;   // "GET"
type Invalid = ExtractMethod<"INVALID /users">; // never
```

## Union Distribution

```typescript
type Events = "click" | "focus" | "blur";
type HandlerPrefix = `on${Capitalize<Events>}`;
// Type: "onClick" | "onFocus" | "onBlur"
```

## Practical Use: Typed CSS Classes

```typescript
type Size = "sm" | "md" | "lg";
type Color = "red" | "blue" | "green";
type ButtonClass = `btn-${Size}-${Color}`;
// Type: "btn-sm-red" | "btn-sm-blue" | "btn-sm-green" | ...

type ButtonClass2 = `btn-${Size}-${Color}` | "btn-disabled";
```

## Real-World Examples

### Example 1: Type-Safe CSS-in-JS Library

When building a CSS-in-JS library, enforce valid class name combinations:

```typescript
type CSSSize = "xs" | "sm" | "md" | "lg" | "xl";
type CSSColor = "slate" | "red" | "blue" | "green" | "yellow";
type CSSVariant = "solid" | "outline" | "ghost";

// Generate all valid utility classes
type UtilityClass = `u-${CSSSize}` | `bg-${CSSColor}` | `text-${CSSColor}-${CSSVariant}`;

// Create a typed style function
function styled<T extends string>(className: T): { className: T } {
  return { className };
}

// Usage
const button = styled("bg-blue-solid");   // OK
const warning = styled("bg-yellow-outline"); // OK
const invalid = styled("bg-purple-solid");   // Error: "purple" is not a valid color
```

### Example 2: Event System with Namespaced Events

When building a plugin or module system with namespaced events:

```typescript
type ModuleName = "user" | "post" | "auth";
type Action = "created" | "updated" | "deleted" | "loading";
type ModuleEvent = `${ModuleName}:${Action}`;

// Type-safe event emitter
class TypedEventEmitter {
  private listeners: Map<string, Set<Function>> = new Map();

  on<T extends string>(event: T, callback: Function): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback);
  }

  emit<T extends ModuleEvent>(event: T, data: any): void {
    this.listeners.get(event)?.forEach((cb) => cb(data));
  }
}

const emitter = new TypedEventEmitter();

// All these are valid:
emitter.on("user:created", (data) => console.log("User created", data));
emitter.on("post:loading", (data) => console.log("Post loading", data));
emitter.on("auth:deleted", (data) => console.log("Auth deleted", data));

// Error: "admin:created" is not a valid ModuleEvent
emitter.on("admin:created", (data) => {}); // Error!
```

### Example 3: Internationalization Key Paths

When building an i18n library with typed translation keys:

```typescript
type FlattenKeys<T, Prefix extends string = ""> = T extends object
  ? {
      [K in keyof T & string]: T[K] extends object
        ? FlattenKeys<T[K], `${Prefix}${K}.`>
        : `${Prefix}${K}`;
    }[keyof T & string]
  : never;

interface Translations {
  common: {
    buttons: {
      save: string;
      cancel: string;
      delete: string;
    };
    messages: {
      success: string;
      error: string;
    };
  };
  auth: {
    login: string;
    logout: string;
    register: string;
  };
}

type TranslationKey = FlattenKeys<Translations>;
// "common.buttons.save" | "common.buttons.cancel" | "common.buttons.delete" |
// "common.messages.success" | "common.messages.error" |
// "auth.login" | "auth.logout" | "auth.register"

function t(key: TranslationKey): string {
  // Implementation
  return key.split(".").reduce((obj, k) => (obj as any)[k], translations);
}

t("common.buttons.save");  // OK
t("auth.login");           // OK
t("invalid.key");          // Error!
```

### Example 4: Type-Safe SQL Query Builder

When building a SQL query builder package:

```typescript
type SQLKeyword = "SELECT" | "FROM" | "WHERE" | "ORDER BY" | "LIMIT";
type ValidColumn = string;
type QueryPart = `${SQLKeyword} ${ValidColumn}`;

// Validate SQL is properly formed
function buildQuery<T extends string>(sql: T): { query: T } {
  return { query: sql };
}

// Usage
const query = buildQuery("SELECT id, name FROM users WHERE age > 18"); // OK
const bad = buildQuery("SELECT id FROM");  // Error: incomplete query
const invalid = buildQuery("DELETE id");   // Error: wrong keyword order
```

### Example 5: API Endpoint Builder

When building an API client with type-safe endpoints:

```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "PATCH" | "DELETE";
type ApiVersion = "v1" | "v2";
type Resource = "users" | "posts" | "comments";
type ApiEndpoint = `${HTTPMethod} /api/${ApiVersion}/${Resource}`;

// Create typed API client
class ApiClient {
  request<T extends ApiEndpoint>(endpoint: T): Promise<any> {
    const [method, path] = endpoint.split(" ");
    // Implementation
    return fetch(path, { method });
  }
}

const api = new ApiClient();

// All valid endpoints
api.request("GET /api/v1/users");
api.request("POST /api/v1/posts");
api.request("DELETE /api/v2/comments");

// Error: invalid method
api.request("GETS /api/v1/users"); // Error!

// Error: invalid version
api.request("GET /api/v3/users");  // Error!
```

### Example 6: Typed Local Storage Keys

When building a storage abstraction with type-safe keys:

```typescript
type StorageKey = "user:token" | "user:preferences" | "app:theme" | "app:version";

// Create typed storage wrapper
function getStorage<T extends StorageKey>(key: T): string | null {
  return localStorage.getItem(key);
}

function setStorage<T extends StorageKey>(key: T, value: string): void {
  localStorage.setItem(key, value);
}

// Usage
const token = getStorage("user:token");     // OK
const theme = getStorage("app:theme");      // OK

// Error: invalid key
const invalid = getStorage("session:token"); // Error!
```

## When to Use Template Literal Types

| Scenario | Example |
|----------|---------|
| Event naming conventions | `"user:created"`, `"post:updated"` |
| CSS utility classes | `"bg-${color}-${variant}"` |
| Translation/i18n keys | `"common.buttons.save"` |
| API endpoint definitions | `"GET /api/v1/users"` |
| SQL query validation | `"SELECT ${column} FROM ${table}"` |
| Storage keys | `"user:${key}"`, `"app:${key}"` |

## Key Takeaways

- Template literals create precise string types
- Combine with `infer` to extract parts of strings
- Use with mapped types for dynamic property names
- Distributes over union types automatically
- Enables compile-time checking of string patterns
- Essential for building type-safe DSLs and APIs
