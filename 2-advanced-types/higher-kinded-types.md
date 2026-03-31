# Higher Kinded Types (HKT)

> **Decision Tree**: Can't find what you need? Jump to the section:
> - I need to work with `Array<T>`, `Promise<T>`, `Option<T>` generically → HKT
> - I need Functor/Monad patterns (map, flatMap) → HKT
> - I just need function overloads → **Function Overloads** (simpler!)
> - I'm building a library/framework → HKT
> - I'm writing business code (Services, Controllers) → **Don't use HKT**

---

## Quick Decision Table

| If your need is... | Use this pattern |
|--------------------|------------------|
| Create a function that accepts `T` | **Simple Generics** |
| Create a function that accepts `Array<T>` specifically | **Generics with constraints** |
| Create a function that accepts **any container** `F<T>` (Functor/Monad) | **HKT** |
| Build a plugin system or ORM | **HKT** |
| Chain operations on Options/Results | **HKT** |

---

## Concept: What is an HKT?

TypeScript doesn't natively support HKTs. The problem:

```typescript
// What we WANT: map over ANY container type
map(["hi", "bye"], doubleString); // ["hihi", "byebye"]
map(Promise.resolve("hi"), doubleString); // Promise<"hihi">

// What we CAN'T easily express in TypeScript:
declare function map<F extends ???, A, B>(
  container: F<A>,
  fn: (a: A) => B
): F<B>; // How to say "F is a type constructor"?
```

**HKT encoding** simulates this by using a "placeholder" property `_1`.

---

## 1. When: Abstract Over Containers (Functor Pattern)

**Scenario**: "I want a single `map` function that works on Arrays, Promises, Options—without rewriting logic for each type."

**Solution**: HKT encoding with `Apply`

```typescript
// The HKT infrastructure (the "slot")
abstract class HKT {
  readonly _1?: unknown;
  new?: (...args: any[]) => unknown;
}

// "Apply" fills the slot with a type
type Apply<F extends HKT, A> = ReturnType<(F & { readonly _1: A })["new"]>;

// Define a container (e.g., Array)
interface ArrayHKT extends HKT {
  new: <A>(x: Apply<this["_1"], A>) => A[];
}

// Now we can write container-agnostic operations
type Map<F extends HKT, A, B> = (container: Apply<F, A>, fn: (a: A) => B) => Apply<F, B>;
```

---

## 2. When: Functional Programming (Option/Maybe)

**Scenario**: "I build a FP library with `Option<T>` that has `map` and `flatMap`. I want these to work generically."

```typescript
interface OptionHKT extends HKT {
  new: <A>(x: Apply<this["_1"], A>) => A | undefined;
}

// Apply Option to "string"
type OptString = Apply<OptionHKT, string>;
// string | undefined
```

---

## 3. When: Composable Query Builder

**Scenario**: "I build an ORM with chainable methods. `query().map().flatMap()` should be fully typed."

```typescript
interface QueryHKT extends HKT {
  new: <A>(x: Apply<this["_1"], A>) => Query<A>;
}

interface Query<A> {
  execute(): Promise<A>;
  map<B>(f: (a: A) => B): Query<B>;
  flatMap<B>(f: (a: A) => Query<B>): Query<B>;
}

// Fully typed, chainable queries
declare function query<A>(sql: string): Query<A>;

const userQuery = query<User>("SELECT * FROM users WHERE id = 1");
const userNameQuery = userQuery.map((u) => u.name); // Query<string>
```

---

## 4. When: Plugin/Decorator System

**Scenario**: "I build a system where plugins enrich objects. I want to compose `Logging`, `Cache`, `Validation` plugins and get exact types."

**Solution**: HKT-based plugin composition

```typescript
// Base HKT
abstract class HKT {
  readonly _1?: unknown;
  readonly _2?: unknown; // For plugins that add properties
}

type Assume<T, U> = T extends U ? T : U;

interface Plugin extends HKT {
  new: <A extends object>(obj: Assume<this["_1"], A>) => A & this["_2"];
}

// Apply a plugin to an object type
type ApplyPlugin<P extends Plugin, A extends object> = ReturnType<
  (P & { readonly _1: A })["new"]
>;

// Logging plugin
interface WithLogging extends Plugin {
  readonly _2: { log: (msg: string) => void; logCount: number };
}

// Usage
type UserWithLogging = ApplyPlugin<WithLogging, User>;
// User & { log: (msg: string) => void; logCount: number }

declare function addLogging<A extends object>(obj: A): ApplyPlugin<WithLogging, A>;

const user = addLogging({ id: "1", name: "John" });
user.log("User created"); // ✅ Fully typed
```

---

## ⚠️ Senior Notes: When NOT to Use HKT

### 1. Function Overloads First

Before reaching for HKT, ask: **can 3-4 overloads suffice?**

```typescript
// ✅ SIMPLER: Function overloads
function map<T>(arr: T[], fn: (t: T) => unknown): unknown[];
function map<T>(promise: Promise<T>, fn: (t: T) => unknown): Promise<unknown>;
function map(arrOrPromise: unknown, fn: Function): unknown {
  // implementation
}

// ❌ MORE COMPLEX: HKT (only if you truly need generic containers)
```

### 2. Compiler Performance

HKTs use heavy intersection types and `this`. On large projects, this **slows down VS Code autocomplete**.

### 3. Team Readability

HKT is "Type Over-engineering" for 90% of developers. If your team can't read it, it's a liability.

### 4. Documentation Requirement

If you use HKT in a project, **this file should be the most documented**. Explain "Why" before "How".

---

## The `Assume` Utility

Crucial for HKT encoding—forces TypeScript to accept a type:

```typescript
type Assume<T, U> = T extends U ? T : U;

// Without Assume:
this["_1"]; // Type is unknown

// With Assume:
function process<A>(x: Apply<this["_1"], A>) {
  // A is assumed to be the correct type
}
```

---

## When to Use HKTs (Summary)

| Use Case | HKT Recommended? |
|----------|------------------|
| Business logic (Services, Controllers) | ❌ No |
| Simple utility functions | ❌ No |
| Functor/Alternative/Monad libraries | ✅ Yes |
| ORM query builders | ✅ Yes |
| Plugin systems | ✅ Yes |
| Parser combinators | ✅ Yes |

---

## Key Takeaways

- HKTs encode "type constructors taking type constructors" in TypeScript
- Use `_1` property as a "slot" that gets filled by `Apply<F, A>`
- **`Assume<T, U>`** forces TS to accept a type in the slot
- HKTs enable Functor/Monad patterns and plugin composition
- **Don't use for business code**—overloads are simpler and faster
- HKTs slow down IDE autocomplete on large projects
- If you use HKTs, document the WHY before the HOW
