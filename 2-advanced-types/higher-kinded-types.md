# Higher Kinded Types

Type constructors that take other type constructors as arguments, enabling functional programming patterns like monadic composition.

## The Problem

TypeScript doesn't natively support Higher Kinded Types (HKTs). Consider trying to type a generic `map` function:

```typescript
// What we want: map over a tuple with any functor
map(["hi", "bye"], (x) => `${x}!`); // ["hi!", "bye!"]

// What we can't easily express:
declare const map = <
  X extends unknown[],
  F extends <Y>(x: Y) => unknown
>(x: X, f: F): // ??? What return type?
```

The issue: we can't express "F is a type constructor that takes a type and returns a type."

## Encoding HKTs in TypeScript

### Step 1: Base Infrastructure

```typescript
// A generic function type - all functions are subtypes of this
type GenericFunction = (...x: never[]) => unknown;

// The HKT base class - _1 represents the type parameter
abstract class HKT {
  readonly _1?: unknown;
  new?: GenericFunction;
}

// Type assumption utility
type Assume<T, U> = T extends U ? T : U;
```

### Step 2: Apply Type

The `Apply` type is the core of HKT encoding. It "applies" a type to the HKT:

```typescript
type Apply<F extends HKT, _1> = ReturnType<
  (F & { readonly _1: _1 })["new"]
>;
```

### Step 3: Define Type Constructors

```typescript
// A type constructor: takes a string, returns a doubled string
interface DoubleString extends HKT {
  new: (x: Assume<this["_1"], string>) => `${typeof x}${typeof x}`;
}

// "hihi"
type Result = Apply<DoubleString, "hi">;
```

## Practical Example: Functor Mapping

### Type-Level Tuple Map

```typescript
type MapTuple<X extends readonly unknown[], F extends HKT> = {
  [K in keyof X]: Apply<F, X[K]>;
};

// ["hellohello", "worldworld"]
type Mapped = MapTuple<["hello", "world"], DoubleString>;
```

### Value-Level Implementation

```typescript
type InferredType = string | number | boolean | object | undefined | null;
type InferredTuple = InferredType[] | ReadonlyArray<InferredType>;

type InstanceOf<T> = T extends new (...args: any[]) => infer R ? R : never;

declare function map<X extends InferredTuple, F extends typeof HKT>(
  x: readonly [...X],
  f: F
): MapTuple<X, Assume<InstanceOf<F>, HKT>>;
```

## Real-World Examples

### Example 1: Option/Maybe Functor

When building a functional programming library with optional values:

```typescript
// Option type constructor
interface OptionT extends HKT {
  new: <A>(x: Assume<this["_1"], A>) => A | undefined;
}

// Apply to Option
type Option<A> = A | undefined;

// "hello" | undefined
type Result1 = Apply<OptionT, "hello">;

// [1, 2] | undefined
type Result2 = Apply<OptionT, [1, 2]>;

// Chainable map
interface OptionMap extends HKT {
  new: <A, B>(x: Assume<this["_1"], A>, f: (a: A) => B) => Option<B>;
}
```

### Example 2: Result/Either Functor

For error handling with Result types:

```typescript
interface ResultT extends HKT {
  new: <E, A>(x: Assume<this["_1"], { ok: true; value: A } | { ok: false; error: E }>) =>
    | { ok: true; value: A }
    | { ok: false; error: E };
}

type Result<E, A> = { ok: true; value: A } | { ok: false; error: E };

// Result<string, number>
type R = Apply<ResultT, { ok: true; value: number }>;
```

### Example 3: Array Functor

```typescript
interface ArrayT extends HKT {
  new: <A>(x: Assume<this["_1"], A>) => A[];
}

// number[]
type ArrNum = Apply<ArrayT, number>;

// string[]
type ArrStr = Apply<ArrayT, string>;

// Tuple mapping
type MapArrTuple<T extends readonly unknown[], F extends HKT> = {
  [K in keyof T]: Apply<F, T[K]>;
};

// [number[], string[]]
type Mixed = MapArrTuple<[number, string], ArrayT>;
```

### Example 4: Chaining Monadic Operations

When building a query builder or DSL:

```typescript
interface QueryT extends HKT {
  new: <A>(x: Assume<this["_1"], A>) => {
    execute: () => Promise<A>;
    map: <B>(f: (a: A) => B) => Query<B>;
  };
}

declare function query<A>(sql: string): Query<A>;

interface Query<A> {
  execute: () => Promise<A>;
  map<B>(f: (a: A) => B): Query<B>;
  flatMap<B>(f: (a: A) => Query<B>): Query<B>;
}

// Composable queries
const userQuery = query<User>("SELECT * FROM users WHERE id = 1");
const userNameQuery = userQuery.map((u) => u.name);
```

### Example 5: Type-Safe Plugin System

A practical HKT-based plugin system that enriches objects dynamically:

```typescript
// Base HKT infrastructure
type GenericFunction = (...x: never[]) => unknown;

abstract class HKT {
  readonly _1?: unknown;
  new?: GenericFunction;
}

type Assume<T, U> = T extends U ? T : U;

// The core: Plugin takes an object type and returns an enriched object type
interface Plugin extends HKT {
  new: <A extends object>(obj: Assume<this["_1"], A>) => A & this["_2"];
}

// Apply a plugin to an object type
type ApplyPlugin<P extends Plugin, A extends object> = ReturnType<
  (P & { readonly _1: A })["new"]
>;

// Example: Logging plugin
interface WithLogging extends Plugin {
  readonly _2: {
    log: (message: string) => void;
    logCount: number;
  };
}

class LoggingPlugin {
  readonly _1!: object;
  readonly _2!: { log: (message: string) => void; logCount: number };

  new = <A extends object>(obj: A): A & WithLogging["_2"] => ({
    ...obj,
    log: (message: string) => console.log(`[LOG] ${message}`),
    logCount: 0,
  });
}

// Example: Caching plugin
interface WithCache extends Plugin {
  readonly _2: {
    cache: Map<string, unknown>;
    get: <K extends string>(key: K) => unknown | undefined;
    set: <K extends string>(key: K, value: unknown) => void;
    clear: () => void;
  };
}

class CachePlugin {
  readonly _1!: object;
  readonly _2!: WithCache["_2"];

  new = <A extends object>(obj: A): A & WithCache["_2"] => {
    const cache = new Map<string, unknown>();
    return {
      ...obj,
      cache,
      get: (key) => cache.get(key),
      set: (key, value) => cache.set(key, value),
      clear: () => cache.clear(),
    };
  };
}

// Example: Validation plugin
interface WithValidation extends Plugin {
  readonly _2: {
    validate: (data: unknown) => boolean;
    errors: string[];
  };
}

class ValidationPlugin {
  readonly _1!: object;
  readonly _2!: WithValidation["_2"];

  new = <A extends object>(obj: A): A & WithValidation["_2"] => ({
    ...obj,
    validate: (data) => {
      console.log("Validating:", data);
      return true;
    },
    errors: [],
  });
}

// Compose multiple plugins
type ComposePlugins<P1 extends Plugin, P2 extends Plugin> = Plugin & {
  new: <A extends object>(obj: A) => ApplyPlugin<P2, ApplyPlugin<P1, A>>;
};

// Usage: Create an enriched object
function createEnriched<A extends object, P extends Plugin>(
  obj: A,
  plugin: { new(): P }
): ApplyPlugin<P, A> {
  const p = new plugin();
  return p.new(obj) as ApplyPlugin<P, A>;
}

// User class that can be enriched
interface User {
  id: string;
  name: string;
  email: string;
}

// Enrich a User with logging
const userWithLogging = createEnriched(
  { id: "1", name: "John", email: "john@example.com" },
  LoggingPlugin
);
userWithLogging.log("User created"); // Works! Type-safe
userWithLogging.logCount; // number

// Enrich with multiple plugins
type UserWithAll = User & WithLogging["_2"] & WithCache["_2"] & WithValidation["_2"];

function createFullyEnriched<A extends object>(
  obj: A,
  plugins: Array<{ new(): Plugin }>
): A {
  let result = obj as object;
  for (const plugin of plugins) {
    const p = new plugin();
    result = (p as any).new(result);
  }
  return result as A;
}

const userFullyEnriched = createFullyEnriched(
  { id: "1", name: "John", email: "john@example.com" },
  [LoggingPlugin, CachePlugin, ValidationPlugin]
);

userFullyEnriched.log("User updated");     // Type-safe logging
userFullyEnriched.set("profile", userFullyEnriched); // Type-safe cache
userFullyEnriched.validate({ name: "Test" });        // Type-safe validation
```

**Type-safe plugin registry:**

```typescript
// Plugin registry with full type inference
interface PluginRegistry {
  logging: WithLogging;
  cache: WithCache;
  validation: WithValidation;
}

type RegisteredPlugin<K extends keyof PluginRegistry> = PluginRegistry[K];

function registerPlugin<K extends keyof PluginRegistry>(
  name: K,
  plugin: { new(): RegisteredPlugin<K> }
): RegisteredPlugin<K> {
  return new plugin() as RegisteredPlugin<K>;
}

// Type-safe plugin application
function applyPlugin<P extends Plugin, A extends object>(
  obj: A,
  plugin: { new(): P }
): ApplyPlugin<P, A> {
  const p = new plugin();
  return p.new(obj) as ApplyPlugin<P, A>;
}

// Build objects with specific plugins
type UserWithLogging = ApplyPlugin<WithLogging, User>;
type UserWithCache = ApplyPlugin<WithCache, User>;
type UserWithValidation = ApplyPlugin<WithValidation, User>;

declare const user: User;
const loggingUser = applyPlugin(user, LoggingPlugin);
// loggingUser is UserWithLogging - fully typed!
```

**When to use this pattern:**

| Use Case | Example |
|----------|---------|
| Web frameworks | Express middleware, Koa context enrichment |
| Database ORMs | Adding query-building methods to models |
| UI Components | Adding mixin capabilities |
| API clients | Adding retry, caching, logging middleware |
| State management | Adding persistence, undo/redo to stores |

## HKT Utilities

### Compose

Compose two HKTs:

```typescript
type Compose<F extends HKT, G extends HKT> = HKT & {
  new: <A>(x: Apply<G, Apply<F, A>>) => Apply<G, Apply<F, A>>;
};
```

### Tuple of HKTs

For multiple type parameters:

```typescript
abstract class HKT2 {
  readonly _1?: unknown;
  readonly _2?: unknown;
}

type Apply2<F extends HKT2, _1, _2> = ReturnType<
  (F & { readonly _1: _1; readonly _2: _2 })["new"]
>;
```

## When to Use HKTs

| Scenario | Example |
|----------|---------|
| Functional libraries | Implementing Functor/Monad interfaces |
| Query builders | Composable, chainable type-safe queries |
| ORM query generation | Type-safe SQL builders |
| Event systems | Composable event transformations |
| Parser combinators | Chained parser transformations |
| Plugin systems | Dynamically enriching objects while staying typed |
| Middleware/decorators | Composable cross-cutting concerns |

## Limitations

1. **Syntax overhead** — Requires class-based encoding with `this["_1"]`
2. **No native variance** — Must manually handle covariance/contravariance
3. **Complexity** — Harder to debug and understand
4. **Not perfect** — TypeScript's structural typing limits the encoding

## Key Takeaways

- HKTs enable "type constructors taking type constructors"
- Encode using classes with `this["_1"]` to represent the type parameter
- `Apply<F, A>` applies type `A` to HKT `F`
- Enables Functor/Monad patterns in TypeScript
- Use when building functional programming libraries
- Consider if simpler patterns suffice before reaching for HKTs
