# Conditional Types

Create types that depend on conditions, enabling sophisticated type logic.

## Basic Conditional Type

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```

## Extracting with Infer

The `infer` keyword extracts a type from within a conditional:

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// Type: { id: number; name: string; }
```

## Extracting Function Parameters

```typescript
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number): void {}

type Params = Parameters<typeof greet>;
// Type: [name: string, age: number]
```

## Distributive Conditional Types

When the type parameter is a union, conditional types distribute over each member:

```typescript
type ToArray<T> = T extends any ? T[] : never;

type StrOrNumArray = ToArray<string | number>;
// Type: string[] | number[]
```

## Nested Conditions

Chain conditions for complex type logic:

```typescript
type TypeName<T> = T extends string
  ? "string"
  : T extends number
    ? "number"
    : T extends boolean
      ? "boolean"
      : T extends undefined
        ? "undefined"
        : T extends Function
          ? "function"
          : "object";

type T1 = TypeName<string>;              // "string"
type T2 = TypeName<() => void>;         // "function"
type T3 = TypeName<{ a: number }>;      // "object"
```

## Practical Example: Unwrap Promise

```typescript
type Awaited<T> = T extends Promise<infer U>
  ? U extends Promise<infer V>
    ? Awaited<U>
    : U
  : T;

type Num = Awaited<Promise<number>>;           // number
type User = Awaited<Promise<Promise<User>>>;   // User
```

## keyof with Conditional Types

```typescript
type Pick<T, K> = T extends any ? (K extends keyof T ? T[K] : never) : never;

type Name = Pick<{ name: string; age: number }, "name">;
// Type: string
```

## Common Patterns

```typescript
// Filter out null and undefined
type NonNullable<T> = T extends null | undefined ? never : T;

// Get the element type of an array
type ElementOf<T> = T extends (infer E)[] ? E : never;

// Check if type is any
type IsAny<T> = 0 extends 1 & T ? true : false;
```

## Real-World Examples

### Example 1: Extracting API Response Types

When building an HTTP client package, extract response types from fetch functions:

```typescript
type UnwrapResponse<T> = T extends () => Promise<{ data: infer R }>
  ? R
  : T extends () => Promise<infer R>
    ? R
    : never;

// Define your API endpoints
async function fetchUser(): Promise<{ data: User }> {
  return { data: { id: "1", name: "John" } };
}

async function fetchPosts(): Promise<Post[]> {
  return [{ id: "1", title: "Post 1" }];
}

// Extract the data type automatically
type UserResponse = UnwrapResponse<typeof fetchUser>;   // User
type PostsResponse = UnwrapResponse<typeof fetchPosts>;  // Post[]
```

### Example 2: Type-Safe Database Query Builder

When building a query builder, use conditional types to infer return types:

```typescript
interface QueryResult<T> {
  rows: T[];
  rowCount: number;
}

type ExtractEntity<T> = T extends Table<infer E> ? E : never;

interface Table<T> {
  name: string;
}

interface Database {
  query<T extends string>(sql: T): Promise<QueryResult<unknown>>;
}

function createTable<T>(name: string): Table<T> {
  return { name } as Table<T>;
}

// Usage
const usersTable = createTable<User>("users");
const postsTable = createTable<Post>("posts");

async function getAll<T extends Table<any>>(table: T): Promise<QueryResult<ExtractEntity<T>>[]> {
  return db.query(`SELECT * FROM ${table.name}`);
}

const results = await getAll(usersTable);
// Type: QueryResult<User>[]
```

### Example 3: Discriminated Union Type Extraction

When building a state management library, extract specific states:

```typescript
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: Error };

// Extract only the success state type
type SuccessState<T> = T extends { status: "success"; data: infer D } ? D : never;

type UserFromState = SuccessState<State>; // User

// Extract only the error state type
type ErrorState<T> = T extends { status: "error"; error: infer E } ? E : never;

type ErrorFromState = ErrorFromState<State>; // Error
```

### Example 4: Type-Safe Route Parameters

When building a router package, extract parameter types from route patterns:

```typescript
type ExtractRouteParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
      ? Param
      : never;

type Params = ExtractRouteParams<"/users/:userId/posts/:postId">;
// "userId" | "postId"

// Type-safe route handler
interface Route<T extends string> {
  path: T;
  params: ExtractRouteParams<T>;
  handler: (params: Record<ExtractRouteParams<T>, string>) => void;
}

function createRoute<T extends string>(path: T): Route<T> {
  return {
    path,
    params: undefined as any,
    handler: () => {},
  };
}

const route = createRoute("/users/:userId/posts/:postId");
// route.params = "userId" | "postId"
```

### Example 5: Conditional Type for Plugin System

When building an extensible plugin system:

```typescript
interface Plugin<State = any> {
  name: string;
  state: State;
}

type PluginState<T> = T extends Plugin<infer S> ? S : never;

function registerPlugin<T extends Plugin>(plugin: T): PluginState<T> {
  return plugin.state as PluginState<T>;
}

// Usage
const authPlugin = { name: "auth", state: { isAuthenticated: false, user: null } };
const configPlugin = { name: "config", state: { theme: "dark", language: "en" } };

type AuthState = PluginState<typeof authPlugin>;   // { isAuthenticated: boolean; user: null }
type ConfigState = PluginState<typeof configPlugin>; // { theme: string; language: string }
```

## When to Use Conditional Types

| Scenario | Example |
|----------|---------|
| Extracting types from promises | `Awaited<Promise<T>>` |
| Building query builders | `ExtractEntity<Table<T>>` |
| Creating plugin systems | `PluginState<T>` |
| Route parameter extraction | `ExtractRouteParams<"/users/:id">` |
| Filtering union types | `SuccessState<State>` |
| Type-safe API clients | `UnwrapResponse<fetchFn>` |

## Key Takeaways

- Conditional types distribute over unions
- Use `infer` to extract types from within conditional branches
- Use `[T] extends [U]` to prevent distribution when needed
- Essential for building type-safe libraries and packages
- Combine with mapped types for complex transformations
