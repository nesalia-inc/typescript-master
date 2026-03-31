# Generics

> **Decision Tree**: Can't find what you need? Jump to the section:
> - I need flexibility on data type → Basic Generics
> - I need to enforce a minimum shape (has `id`, has `length`) → Generic Constraints
> - I need the compiler to infer types automatically → Type Inference
> - I need reusable data structures (Stack, Queue, Cache) → Generic Classes
> - I need API flexibility (Result<T, E>) → Generic Defaults

---

## Quick Decision Table

| When your problem is... | Use this pattern |
|-------------------------|------------------|
| "I accept any data but it MUST have property X" | `T extends HasProperty` |
| "Return type should match input type exactly" | Inference (no explicit `<T>`) |
| "90% use the same error type" | Generic defaults: `T, E = Error` |
| "I have more than 3 type params" | Split into smaller types |
| "The callback loses type info" | `TEntity`, `TContext` (descriptive names) |

---

## Basic Generics

> **When**: You create a component (Stack, Queue, Wrapper) that shouldn't care about the data it holds.

```typescript
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(42);    // Type: number
const str = identity<string>("hello"); // Type: string
const auto = identity(true);          // Type inferred: boolean
```

**Benefit**: The same function works with any type while maintaining full type safety.

---

## Generic Constraints

> **When**: Your function accepts any object, but that object **MUST** possess a specific property (`id`, `length`, `name`, etc.).

### The `extends` Keyword

```typescript
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): T {
  console.log(item.length);
  return item;
}

logLength("hello");           // OK: string has length
logLength([1, 2, 3]);         // OK: array has length
logLength({ length: 10 });    // OK: object has length
// logLength(42);             // Error: number has no length
```

### Constraint Power: `keyof`

> **When**: One type parameter should constrain another (get property by key).

```typescript
// KEY INSIGHT: K is constrained by T's keys
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "John", age: 30 };
const name = getProperty(user, "name"); // Type: string
// getProperty(user, "email"); // Error: "email" is not a key of user
```

**Senior Note**: This is the "bread and butter" of type-safe libraries like `react-hook-form` and `zod`.

---

## Type Inference

> **When**: You want the return type to be exactly the same as the input type (without losing information).

The compiler infers `T` automatically:

```typescript
// No <T> needed - compiler figures it out
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge({ name: "John" }, { age: 30 });
// Type: { name: string } & { age: number }
```

> **Senior Note**: If users have to manually write `<string>` every time, your generic design can be improved. Aim for "invisible" generics through inference.

---

## Generic Interfaces

> **When**: You need a contract that works across multiple types (like `Promise<T>` or `Array<T>`).

```typescript
interface Container<T> {
  value: T;
  map<U>(fn: (value: T) => U): Container<U>;
}

function wrap<T>(value: T): Container<T> {
  return {
    value,
    map(fn) {
      return wrap(fn(value));
    },
  };
}

const num = wrap(42).map((n) => n.toString());
// Type: Container<string>
```

---

## Generic Classes

> **When**: You build reusable data structures (Stack, Queue, Tree, Cache).

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
  peek(): T | undefined { return this.items[this.items.length - 1]; }
}

const stack = new Stack<number>();
stack.push(1);
stack.push(2);
const top = stack.pop(); // Type: number | undefined
```

---

## Generic Type Aliases

> **When**: You need a Result type for error handling, or similar patterns.

```typescript
type Nullable<T> = T | null;
type Optional<T> = T | undefined;

// Generic with DEFAULT type parameter
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) return { ok: false, error: "Division by zero" };
  return { ok: true, value: a / b };
}
```

> **Best Practice for Library Authors**: Generic defaults like `E = Error` keep the API simple for 90% of users while providing flexibility for the 10% who need custom error types.

---

## Real-World Decision Trees

### Decision: I need to validate/provide data access

**🟢 Scenario**: I'm building an ORM package with type-safe CRUD operations.

```
Problem: Each table has different columns, but CRUD logic is the same.
Solution:
```

```typescript
interface Entity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Repository<TEntity extends Entity> {
  findById(id: string): Promise<TEntity | null>;
  findAll(options?: { limit?: number; offset?: number }): Promise<TEntity[]>;
  save(entity: Omit<TEntity, "id" | "createdAt" | "updatedAt">): Promise<TEntity>;
  update(id: string, data: Partial<Omit<TEntity, keyof Entity>>): Promise<TEntity>;
  delete(id: string): Promise<void>;
}
```

---

### Decision: I need type-safe middleware chain

**🟢 Scenario**: I'm building a web framework with async middleware.

```
Problem: async functions return Promise<T>, not T. The interface must account for this.
Solution:
```

```typescript
// KEY: TResult is the RESOLVED value, handle returns Promise<TResult>
interface Middleware<TContext, TResponse> {
  handle(context: TContext, next: () => Promise<TResponse>): Promise<TResponse>;
}

class MiddlewareChain<TContext, TResponse> {
  private middlewares: Middleware<TContext, TResponse>[] = [];

  use(middleware: Middleware<TContext, TResponse>): this {
    this.middlewares.push(middleware);
    return this;
  }

  async execute(context: TContext): Promise<TResponse> {
    let index = 0;
    const next = async (): Promise<TResponse> => {
      if (index >= this.middlewares.length) {
        throw new Error("Middleware chain exhausted");
      }
      return this.middlewares[index++].handle(context, next);
    };
    return next();
  }
}
```

---

### Decision: I need a pub/sub system with typed payloads

**🟢 Scenario**: I'm building an event emitter where `on("user:created", callback)` knows the exact user shape.

```
Problem: Passing `Function` loses type safety on callback data.
Solution: Map event names to their payload types.
```

```typescript
interface EventPayloads {
  "user:created": { id: string; email: string };
  "user:updated": { id: string; changes: Partial<{ email: string; name: string }> };
  "user:deleted": { id: string; reason?: string };
}

class TypedEventEmitter {
  private listeners: Map<string, Set<Function>> = new Map();

  on<E extends keyof EventPayloads>(
    event: E,
    callback: (data: EventPayloads[E]) => void
  ): void {
    if (!this.listeners.has(event)) this.listeners.set(event, new Set());
    this.listeners.get(event)!.add(callback);
  }

  emit<E extends keyof EventPayloads>(event: E, data: EventPayloads[E]): void {
    this.listeners.get(event)?.forEach((cb) => cb(data));
  }
}

const emitter = new TypedEventEmitter();
emitter.on("user:created", (data) => {
  console.log(data.email); // TypeScript KNOWS data has email: string
});
```

---

### Decision: I need to merge objects with property collision handling

**🟢 Scenario**: I'm building a library that merges configs where second object's properties should win.

```
Problem: { id: number } & { id: string } = { id: never }.
Solution: Use Omit<T, keyof U> & U to ensure overwrite semantics.
```

```typescript
// Basic merge (has collision problem)
type SimpleMerge<T, U> = T & U;

// Safe merge (second object wins)
type SafeMerge<T, U> = Omit<T, keyof U> & U;

type A = SafeMerge<{ id: number }, { id: string }>;
// { id: string } - second object's type wins
```

---

## ⚠️ Common Pitfalls & Solutions

### Pitfall 1: "Generic Soup" (Too Many Type Parameters)

> **When**: You have `T, U, V, W, X` in one function/class.

**Problem**: Unreadable, hard to maintain.

**Solution**: Split into smaller, focused types.

```typescript
// BAD
function process<T, U, V, W, X>(t: T, u: U, v: V): W { ... }

// GOOD: Group related types into meaningful interfaces
interface Input { t: T; u: U; v: V; }
function process(input: Input): W { ... }
```

### Pitfall 2: Weak Naming

> **When**: Using `T`, `U`, `K` everywhere, even in complex interfaces.

**Solution**: Use descriptive names for complex types:

```typescript
// BAD
Middleware<T, R>

// GOOD
Middleware<TContext, TResponse>
Repository<TEntity extends Entity>
TypedEventEmitter<TEvents extends Events>
```

### Pitfall 3: Casting Instead of Type Guards

> **When**: You use `value as T` in validators.

**Solution**: Use type guards (`value is T`) that prove the type through runtime checks:

```typescript
// BAD
validate(value: unknown): T {
  return value as T; // "Trust me" to compiler
}

// GOOD
function isString(value: unknown): value is string {
  return typeof value === "string";
}

validate(value: unknown): T {
  if (!isString(value)) throw new Error("Not a string");
  return value; // Proven to be string
}
```

### Pitfall 4: Property Collision in Intersections

> **When**: Merging objects that might have the same keys with different types.

**Solution**: Use `Omit<T, keyof U> & U`:

```typescript
type SafeMerge<T, U> = Omit<T, keyof U> & U;
```

---

## When to Use Generics (Summary Table)

| Scenario | Example |
|----------|---------|
| Building reusable utilities/libraries | `Array.map<U>`, `Promise.then<U>` |
| Creating type-safe data structures | `Stack<T>`, `Tree<T>`, `Map<K, V>` |
| Implementing repository/data access layers | `Repository<TEntity extends Entity>` |
| Building middleware or chain-of-responsibility | `Middleware<TContext, TResponse>` |
| Creating event systems | `TypedEventEmitter<TEvents>` |
| Type-safe configuration systems | `ConfigSchema<T>` |

---

## Key Takeaways

- **Generics provide flexibility while maintaining type safety**
- **`extends` constrains to a minimum shape** (not "any")
- **`keyof` lets one generic constrain another**
- **Inference is king**: if users must manually specify `<T>`, improve the design
- **Generic defaults** (`E = Error`) simplify APIs for 90%
- **Descriptive names** (`TEntity`, `TContext`) > single letters for complex types
- **Type guards** (`value is T`) > casting (`as T`)
- **Avoid "Generic Soup"**: >3 params usually means design needs refactoring
- **Constraints are documentation**: `T extends Entity` tells what data is expected
