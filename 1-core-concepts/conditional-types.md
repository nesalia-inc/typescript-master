# Conditional Types

Create types that depend on conditions, enabling sophisticated type logic.

## Pattern

```typescript
type Result<T> = T extends string ? "yes" : "no";
```

When the condition is `true`, return the first branch. When `false`, return the second.

---

## When You Need to Filter Union Types

Use conditional types to exclude or include specific union members:

```typescript
// Exclude null and undefined
type NonNullable<T> = T extends null | undefined ? never : T;

type Clean = NonNullable<string | null | undefined>; // string

// Exclude specific string literals
type NotAdmin<T> = T extends "admin" ? never : T;

type Role = NotAdmin<"admin" | "user" | "guest">; // "user" | "guest"
```

---

## When You Need to Extract Nested Types

Use `infer` inside a conditional to extract parts of a type:

```typescript
// Extract the return type of a function
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>; // { id: number; name: string }

// Extract function parameters
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number): void {}

type Params = Parameters<typeof greet>; // [name: string, age: number]
```

---

## When You Need to Unwrap Promises

Use recursive conditional types to flatten nested promises:

```typescript
type Awaited<T> = T extends Promise<infer U>
  ? U extends Promise<infer V>
    ? Awaited<U> // Recurse for deeper nesting
    : U
  : T;

type A = Awaited<Promise<number>>;              // number
type B = Awaited<Promise<Promise<User>>>;       // User
type C = Awaited<Promise<Promise<Promise<T>>>>;  // T
```

---

## When You Need to Transform Each Union Member

Let the conditional type distribute over unions:

```typescript
type ToArray<T> = T extends any ? T[] : never;

type Arrays = ToArray<string | number | boolean>;
// string[] | number[] | boolean[]
```

---

## When You Need to Check the Type Category

Chain conditions for type-level branching:

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
type T2 = TypeName<() => void>;           // "function"
type T3 = TypeName<{ a: number }>;        // "object"
```

---

## When You Need to Build Type-Safe APIs

Extract response types from API endpoint definitions:

```typescript
type UnwrapResponse<T> = T extends () => Promise<{ data: infer R }>
  ? R
  : T extends () => Promise<infer R>
    ? R
    : never;

async function fetchUser(): Promise<{ data: User }> {
  return { data: { id: "1", name: "John" } };
}

async function fetchPosts(): Promise<Post[]> {
  return [{ id: "1", title: "Post 1" }];
}

type UserResponse = UnwrapResponse<typeof fetchUser>;    // User
type PostsResponse = UnwrapResponse<typeof fetchPosts>;  // Post[]
```

---

## When You Need to Extract from Discriminated Unions

Pull out specific variants from a union:

```typescript
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: Error };

// Extract the data type from success states
type ExtractData<T> = T extends { status: "success"; data: infer D } ? D : never;

type UserData = ExtractData<State>; // User

// Extract error type
type ExtractError<T> = T extends { status: "error"; error: infer E } ? E : never;

type ErrorType = ExtractError<State>; // Error
```

---

## When You Need to Extract Route Parameters

Use template literals with infer to parse paths:

```typescript
type ExtractParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
      ? Param
      : never;

type Params = ExtractParams<"/users/:userId/posts/:postId">;
// "userId" | "postId"

type Single = ExtractParams<"/users/:id">;
// "id"
```

---

## When You Need to Extract from Plugin Systems

Use conditional types to unwrap plugin state:

```typescript
interface Plugin<State = any> {
  name: string;
  state: State;
}

type PluginState<T> = T extends Plugin<infer S> ? S : never;

const authPlugin = { name: "auth", state: { isAuthenticated: false, user: null } };
const configPlugin = { name: "config", state: { theme: "dark", language: "en" } };

type AuthState = PluginState<typeof authPlugin>;    // { isAuthenticated: boolean; user: null }
type ConfigState = PluginState<typeof configPlugin>; // { theme: string; language: string }
```

---

## When You Need to Prevent Distribution

Wrap in brackets to treat union as a single type:

```typescript
// Without brackets - distributes over union
type Test1 = string extends any ? "yes" : "no"; // "yes" | "no"

// With brackets - treats as single type
type Test2 = [string] extends [any] ? "yes" : "no"; // "yes"

// Practical use: strict equality check
type IsNever<T> = [T] extends [never] ? true : false;

type T1 = IsNever<never>;       // true
type T2 = IsNever<string>;      // false
```

---

## When You Need to Build a Query Builder

Combine conditional types with generics for type-safe builders:

```typescript
interface Table<T> {
  name: string;
}

interface QueryResult<T> {
  rows: T[];
  rowCount: number;
}

type ExtractEntity<T> = T extends Table<infer E> ? E : never;

function createTable<T>(name: string): Table<T> {
  return { name } as Table<T>;
}

async function getAll<T extends Table<any>>(
  table: T
): Promise<QueryResult<ExtractEntity<T>>[]> {
  return db.query(`SELECT * FROM ${table.name}`);
}

const users = createTable<User>("users");
const results = await getAll(users);
// results is QueryResult<User>[]
```

---

## Key Takeaways

- Conditional types distribute over unions automatically
- Use `infer` to extract types from branches
- Use `[T] extends [U]` to prevent distribution
- Chain conditions for complex type-level logic
- Recursive conditional types handle nested types
- Combine with template literals for string parsing
