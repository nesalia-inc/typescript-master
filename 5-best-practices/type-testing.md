# Type Testing

Strategies for verifying TypeScript types behave as expected.

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

## Basic Type Assertion Tests

Test that two types are exactly equal:

```typescript
type AssertEqual<T, U> = [T] extends [U]
  ? [U] extends [T]
    ? true
    : false
    : false;
```

**With `@deessejs/type-testing`:**

```typescript
import { Equal } from '@deessejs/type-testing';

type Test1 = Equal<string, string>;   // true
type Test2 = Equal<string, number>;  // false
type Test3 = Equal<string | number, string>; // false
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

## Runtime Tests for Type Generators

```typescript
// Type-safe builder runtime tests
interface User {
  id: string;
  name: string;
  email: string;
}

const builder = new Builder<User>();

// These should compile
const user1 = builder.set("id", "1").set("name", "John").set("email", "j@j.com").build();
const user2 = builder.set("id", "2").set("name", "Jane").set("email", "jane@j.com").build();

// These should NOT compile (uncomment to test)
// builder.build(); // Error: missing required fields
// builder.set("name", 123); // Error: wrong type
```

## CI Integration

```yaml
# .github/workflows/types.yml
- name: Type tests
  run: npx vitest run --reporter=basic
```

## Best Practices

1. Use `@deessejs/type-testing` for a comprehensive testing API
2. Test public API types, not internals
3. Use `Equal<T, U>` for exact type matching
4. Use chainable `check()`, `assert()`, and `expect()` for readable tests
5. Runtime test type-generators with valid and invalid inputs
6. Run type checking in CI alongside regular tests
7. Use `IsAny<T>` to catch the "any" escape hatch
8. Use `DeepPartial<T>` and `DeepReadonly<T>` for nested type testing
