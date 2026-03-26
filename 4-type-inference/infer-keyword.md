# Infer Keyword

Extract types from within conditional types.

## Basic Usage

```typescript
// Extract array element type
type ElementType<T> = T extends (infer U)[] ? U : never;

type Num = ElementType<number[]>; // number
type Str = ElementType<string[]>; // string
type Never = ElementType<boolean>; // never (not an array)
```

## Extracting Promise Type

```typescript
type Awaited<T> = T extends Promise<infer U>
  ? U extends Promise<infer V>
    ? Awaited<U> // Recurse for nested promises
    : U
  : T;

type A = Awaited<Promise<string>>;              // string
type B = Awaited<Promise<Promise<number>>>;   // number
type C = Awaited<boolean>;                     // boolean
```

## Extracting Function Return Type

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type R1 = ReturnType<() => string>;           // string
type R2 = ReturnType<() => Promise<number>>;  // Promise<number>
type R3 = ReturnType<(x: number) => boolean>; // boolean
```

## Extracting Function Parameters

```typescript
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

type P1 = Parameters<(a: string, b: number) => void>; // [a: string, b: number]
type P2 = Parameters<() => string>;                   // []
type P3 = Parameters<(x: { a: number }) => void>;    // [{ a: number }]
```

## Constructor Types

```typescript
type ConstructorParameters<T extends new (...args: any[]) => any> =
  T extends new (...args: infer P) => any ? P : never;

type CP = ConstructorParameters<new (name: string, age: number) => object>;
// [name: string, age: number]
```

## Instance Type

```typescript
type InstanceType<T extends new (...args: any[]) => any> =
  T extends new (...args: any[]) => infer R ? R : any;

class User {
  constructor(public name: string) {}
}

type UserInstance = InstanceType<typeof User>; // User
```

## Chaining Inferences

```typescript
// Extract from nested structures
type First<T> = T extends [infer F, ...infer Rest] ? F : never;
type Rest<T> = T extends [infer F, ...infer Rest] ? Rest : never;

type F = First<[string, number, boolean]>; // string
type R = Rest<[string, number, boolean]>; // [number, boolean]
```

## Default Values with Infer

```typescript
type InferDefault<T, Default> = T extends infer U ? U : Default;

type A = InferDefault<string, never>;     // string
type B = InferDefault<unknown, string>;   // unknown
```

## Practical: Deep Unwrap

```typescript
type DeepAwaited<T> = T extends Promise<infer U>
  ? DeepAwaited<U>
  : T extends Array<infer V>
    ? Array<DeepAwaited<V>>
    : T;

type Result = DeepAwaited<Promise<Array<Promise<User>>>>;
// User
```

## Common Patterns

```typescript
// Unwrap a value if it matches a pattern
type Unwrap<T> = T extends { value: infer V } ? V : T;

// Extract keys of a mapped type
type KeysOf<T, V> = T extends { [K in keyof T]: T[K] extends V ? K : never }[keyof T];

// Get return type of async function
type AsyncReturn<T> = T extends (...args: any[]) => Promise<infer R> ? R : never;
```

## When to Use

- Extracting types from generics
- Building utility types
- Working with function types
- Creating type-level helpers for libraries

## Real-World Examples

### Example 1: Typed Event Emitter

```typescript
type EventMap = {
  user: { id: string; name: string };
  post: { id: string; title: string };
  comment: { id: string; body: string };
};

type EventHandler<T> = (data: T) => void;

// Extract handler type from event name
type HandlerFor<E extends keyof EventMap> = EventHandler<EventMap[E]>;

// Usage
function addHandler<E extends keyof EventMap>(
  event: E,
  handler: HandlerFor<E>
): void {
  // Implementation
}

addHandler("user", (data) => {
  console.log(data.id); // data is { id: string; name: string }
});

addHandler("post", (data) => {
  console.log(data.title); // data is { id: string; title: string }
});
```

### Example 2: Deep Property Access

```typescript
type Path<T, P extends string> =
  P extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? Path<T[K], Rest>
      : never
    : P extends keyof T
      ? T[P]
      : never;

type User = {
  address: {
    city: {
      name: string;
      population: number;
    };
    zip: string;
  };
  name: string;
};

type DeepResult = Path<User, "address.city.name">;
// string

type DeepPop = Path<User, "address.city.population">;
// number

// Won't compile - wrong path
type Invalid = Path<User, "address.city.invalid">;
// never
```

### Example 3: Extract API Response Types

```typescript
type ApiEndpoint<T> = T extends {
  response: infer R;
  request?: unknown;
} ? R : never;

interface GetUserEndpoint {
  response: User;
}

interface CreatePostEndpoint {
  response: Post;
  request: { title: string; content: string };
}

type UserResponse = ApiEndpoint<GetUserEndpoint>;  // User
type PostResponse = ApiEndpoint<CreatePostEndpoint>; // Post

// Works with function types too
type AsyncFn<T> = T extends (...args: any[]) => Promise<infer R> ? R : T;

async function fetchUser(): Promise<User> { return {} as User; }
async function fetchPost(): Promise<Post> { return {} as Post; }

type FetchUserResult = AsyncFn<typeof fetchUser>; // User
type FetchPostResult = AsyncFn<typeof fetchPost>; // Post
```

### Example 4: Unwrapping Result Types

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// Extract value type from Result
type UnwrapResult<T> = T extends { ok: true; value: infer V } ? V : never;

type Success = UnwrapResult<Result<string>>;     // string
type Fail = UnwrapResult<Result<number, string>>; // never (on failure branch)

// Extract error type
type UnwrapError<T> = T extends { ok: false; error: infer E } ? E : never;

type SuccessErr = UnwrapError<Result<string>>;      // never
type FailErr = UnwrapError<Result<number, string>>; // string
```

### Example 5: Function Overload Inference

```typescript
// Infer return type based on input
type Return<T> = T extends (...args: any[]) => infer R ? R : never;

// For overloaded functions, gets the last overload
declare function parse(value: string): string;
declare function parse(value: number): number;
declare function parse(value: boolean): string;

type StringResult = Return<typeof parse>; // string | number (union of all overloads)
```

### Example 6: Conditional Return Types

```typescript
type ReturnKind<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends null ? "null" :
  "object";

type K1 = ReturnKind<string>;   // "string"
type K2 = ReturnKind<42>;       // "number"
type K3 = ReturnKind<true>;      // "boolean"
```
