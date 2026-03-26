# Builder Pattern with Type Safety

A fluent builder that enforces all required fields are set before `build()` is called.

## The Problem

Regular builders allow calling `build()` before all required fields are set:

```typescript
// This compiles but fails at runtime
const user = {
  id: "1",
  // missing name!
};
```

## Solution: State Tracking with Conditional Types

```typescript
// Track which keys have been set
type BuilderState<T> = {
  [K in keyof T]?: T[K];
};

// Extract required keys
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

// Check if builder state is complete
type IsComplete<T, S extends BuilderState<T>> =
  RequiredKeys<T> extends keyof S
    ? S[RequiredKeys<T>] extends undefined
      ? false
      : true
    : false;
```

## The Builder Class

```typescript
class Builder<T, S extends BuilderState<T> = {}> {
  private state: S = {} as S;

  set<K extends keyof T>(key: K, value: T[K]): Builder<T, S & Record<K, T[K]>> {
    (this.state as any)[key] = value;
    return this as any;
  }

  build(this: { state: S } & (IsComplete<T, S> extends true ? {} : never)): T {
    return this.state as T;
  }

  // Reset the builder
  reset(): Builder<T, {}> {
    this.state = {} as S;
    return this as any;
  }
}
```

## Usage

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age?: number;
  role: "admin" | "user" | "guest";
}

const builder = new Builder<User>();

// ✅ All required fields set
const validUser = builder
  .set("id", "1")
  .set("name", "John")
  .set("email", "john@example.com")
  .set("role", "user")
  .build();

// ✅ Optional field can be omitted
const userWithoutAge = builder
  .set("id", "2")
  .set("name", "Jane")
  .set("email", "jane@example.com")
  .set("role", "admin")
  .build();

// ❌ Compile error: missing 'email'
const invalid1 = builder
  .set("id", "3")
  .set("name", "Bob")
  .set("role", "user")
  .build(); // Error!

// ❌ Compile error: wrong type
builder.set("age", "not a number"); // Error!

// ❌ Compile error: invalid enum value
builder.set("role", "superadmin"); // Error!
```

## With Nested Builders

```typescript
interface Address {
  street: string;
  city: string;
  zip: string;
}

interface Company {
  name: string;
  address: Address;
}

class NestedBuilder<T> {
  private state: Partial<T> = {};

  set<K extends keyof T>(key: K, value: T[K]): this {
    (this.state as any)[key] = value;
    return this;
  }

  setNested<K extends keyof T>(
    key: K,
    fn: (builder: Builder<T[K]>) => Builder<T[K]>
  ): this {
    const builder = new Builder<T[K]>();
    (this.state as any)[key] = fn(builder).build();
    return this;
  }

  build(this: { state: T }): T {
    return this.state as T;
  }
}

const company = new NestedBuilder<Company>()
  .set("name", "Acme Corp")
  .set("address", {
    street: "123 Main St",
    city: "Springfield",
    zip: "12345",
  })
  .build();
```

## Key Techniques

1. **State tracking** — `BuilderState<T>` tracks which fields have been set
2. **Method chaining returns new type** — `Builder<T, S & Record<K, T[K]>>`
3. **Conditional `build()` method** — Only available when all required fields are set
4. **`this` parameter type guard** — `this: { state: S } & (IsComplete<...> extends true ? {} : never)`
