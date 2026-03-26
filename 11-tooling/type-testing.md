# Type Testing

Strategies for testing TypeScript types.

## The Library: `@deessejs/type-testing`

A micro library for compile-time type testing in TypeScript.

```bash
npm install @deessejs/type-testing
```

### Core Features

| Category | Types |
|----------|-------|
| **Equality** | `Equal<T, U>`, `NotEqual<T, U>`, `IsNeverEqual<T, U>` |
| **Special Types** | `IsAny<T>`, `IsNever<T>`, `IsUnknown<T>`, `IsNullable<T>` |
| **Structures** | `IsUnion<T>`, `IsTuple<T>`, `IsArray<T>` |
| **Property** | `HasProperty<T, K>`, `IsReadonly<T>`, `IsRequired<T>` |
| **Deep** | `DeepReadonly<T>`, `DeepPartial<T>`, `RequiredKeys<T>` |

### Simple Usage

```typescript
import { Equal, check, assert, expect } from '@deessejs/type-testing';

// Simple equality
type Test1 = Equal<string, string>;  // true
type Test2 = Equal<string, number>;  // false

// Chainable API
check<string>().equals<string>();   // passes

// Assert with clear errors
assert<{ a: string }>().hasProperty('a');

// Expect syntax
expect<string, string>().toBeEqual();
```

## When to Test Types

Test your types when:
- Publishing libraries
- Complex generic functions
- Type-level utilities
- API contracts

## Vitest Type Testing (Recommended)

```bash
npm install --save-dev vitest
```

```typescript
// avatar.test-d.ts
import { expectTypeOf } from "vitest";
import type { Avatar } from "./avatar";

describe("Avatar types", () => {
  test("props are correctly typed", () => {
    expectTypeOf<Avatar>().toHaveProperty("size");
    expectTypeOf<Avatar["size"]>().toEqualTypeOf<"sm" | "md" | "lg">();
  });

  test("default size is md", () => {
    expectTypeOf<Avatar>().toHaveProperty("defaultSize");
    expectTypeOf<Avatar["defaultSize"]>().toEqualTypeOf<"md">();
  });
});
```

## Type Equality with `@deessejs/type-testing`

### Simple Equality

```typescript
import { Equal } from '@deessejs/type-testing';

type Test1 = Equal<string, string>;  // true
type Test2 = Equal<string, number>;  // false
```

### AssertEqual (Manual)

```typescript
type AssertEqual<T, U> = [T] extends [U]
  ? [U] extends [T]
    ? true
    : false
    : false;
```

## Using the Chainable API

```typescript
import { check } from '@deessejs/type-testing';

// Check utility types
check<Partial<{ a: string; b: number }>>()
  .equals<{ a?: string; b?: number }>(); // passes

// Check complex types
check<DeepPartial<{ name: string; address: { city: string } }>>()
  .equals<{ name?: string; address?: { city?: string } }>(); // passes
```

## Using Assert

```typescript
import { assert } from '@deessejs/type-testing';

// Assert with property checking
assert<User>()
  .hasProperty('id')
  .hasProperty<'name', string>()
  .propertyType<'email', string>();

// Assert modifiers
assert<User>()
  .isReadonly('id')
  .isRequired('email');
```

## Using Expect

```typescript
import { expect } from '@deessejs/type-testing';

// Expect-style assertions
expect<string, string>().toBeEqual();
expect<number, number>().toBeEqual();

// With negation
expect<string, number>().not.toBeEqual();
```

## Special Type Detection

```typescript
import { IsAny, IsNever, IsUnknown, IsNullable, IsUnion } from '@deessejs/type-testing';

// Detect special types
type T1 = IsAny<any>;      // true
type T2 = IsAny<unknown>;  // false
type T3 = IsNever<never>;  // true
type T4 = IsUnknown<unknown>; // true
type T5 = IsNullable<string | null>; // true

// Detect union vs single type
type T6 = IsUnion<string | number>; // true
type T7 = IsUnion<string>;          // false
```

## Property Testing

```typescript
import { HasProperty, IsReadonly, IsRequired, IsPublic } from '@deessejs/type-testing';

interface User {
  readonly id: string;
  name: string;
  email?: string;
}

// Property existence
type T1 = HasProperty<User, 'id'>;     // true
type T2 = HasProperty<User, 'name'>;    // true
type T3 = HasProperty<User, 'missing'>; // false

// Property modifiers
type T4 = IsReadonly<User, 'id'>;      // true
type T5 = IsRequired<User, 'name'>;     // true
type T6 = IsRequired<User, 'email'>;    // false (optional)
```

## Deep Type Manipulation

```typescript
import { DeepReadonly, DeepPartial, RequiredKeys, OptionalKeys } from '@deessejs/type-testing';

interface User {
  id: string;
  address: { city: string; zip: string };
}

// Deep transformations
type T1 = DeepReadonly<User>;
// { readonly id: string; readonly address: { readonly city: string; readonly zip: string } }

type T2 = DeepPartial<User>;
// { id?: string; address?: { city?: string; zip?: string } }

// Key extraction
type T3 = RequiredKeys<User>;     // 'id' | 'address'
type T4 = OptionalKeys<User>;     // 'email'
```

## Function Type Testing

```typescript
import { Parameters, ReturnType, IsConstructor, IsAbstract } from '@deessejs/type-testing';

function greet(name: string, age: number): string {
  return `Hello ${name}`;
}

type T1 = Parameters<typeof greet>;  // [name: string, age: number]
type T2 = ReturnType<typeof greet>;  // string

class MyClass {}
type T3 = IsConstructor<typeof MyClass>; // true
```

## Runtime Tests for Types

```typescript
// builder.test.ts
import { describe, expect, test } from "vitest";
import { Builder } from "./builder";

interface User {
  id: string;
  name: string;
  email: string;
}

describe("Builder", () => {
  test("builds valid user", () => {
    const user = new Builder<User>()
      .set("id", "1")
      .set("name", "John")
      .set("email", "john@example.com")
      .build();

    expect(user.id).toBe("1");
    expect(user.name).toBe("John");
  });

  // This won't compile - type error!
  test("missing required field", () => {
    const user = new Builder<User>()
      .set("id", "1")
      .build();
  });
});
```

## CI Integration

```yaml
- name: Type tests
  run: npx vitest run --reporter=basic
```

## Key Takeaways

- Use `@deessejs/type-testing` for comprehensive type testing
- Use `check()`, `assert()`, and `expect()` for fluent, readable tests
- Use `Equal<T, U>` for exact type matching
- Test public API types, not internals
- Combine type tests with runtime tests when possible
- Run type tests in CI
- Use `IsAny<T>` to catch the "any" escape hatch
- Use `DeepPartial<T>` and `DeepReadonly<T>` for nested type testing
