# Type Inference

Techniques for extracting and inferring types automatically from code.

## Topics

- [Infer Keyword](./infer-keyword.md) — Extract types from within types
- [Type Guards](./type-guards.md) — Narrow types at runtime
- [Assertion Functions](./assertion-functions.md) — Enforce types after checks
- [Const Type Parameters](./const-type-parameters.md) — Force literal inference (TS 5.0+)
- [Inference Failure](./inference-failure.md) — Why inference fails and how to fix

## Inference at a Glance

| Technique | Purpose | Example |
|-----------|---------|---------|
| `infer` | Extract nested types | `T extends Promise<infer U> ? U : never` |
| Type guards | Runtime narrowing | `value is string` |
| Assertion functions | Compile-time assertions | `asserts value is string` |
| `const` modifier | Force literal inference | `T extends const` |
| Contextual typing | Infer from usage context | Automatic parameter types |

## When to Use Each

### Infer Keyword

Use when extracting types from nested structures.

| Scenario | Example |
|----------|---------|
| Unwrapping Promises | `Awaited<Promise<T>>` |
| Extracting function params | `Parameters<typeof fn>` |
| Deep property access | `DeepRequired<T>` |

### Type Guards

Use when runtime checks should affect compile-time types.

| Scenario | Example |
|----------|---------|
| Validating external data | `isUser(obj)` |
| Discriminated union narrowing | `isAdmin(person)` |
| Filtering arrays | `isNonNullable(value)` |

### Assertion Functions

Use when a check should fail hard (throw) rather than return false.

| Scenario | Example |
|----------|---------|
| API response validation | `assertSuccess(response)` |
| JSON parsing | `assertIsUser(json)` |
| Exhaustive checks | `assertNever(value)` |

### Const Type Parameters

Use when you need to preserve literal types in builders.

| Scenario | Example |
|----------|---------|
| Route definitions | `createRoute("/users")` returns `{ path: "/users" }` |
| Config defaults | Port stays as `3000` not `number` |
| Event names | Preserves `"click"` not `string` |

### Inference Failure

Understand when and why inference breaks down.

| Scenario | Solution |
|----------|----------|
| Complex generics | Add explicit type parameters |
| Circular types | Use separate type definitions |
| Union ambiguity | Use type predicates or assertions |

## How to Read This

1. **Start with Infer Keyword** — Foundation of type extraction
2. **Type Guards** — Runtime checks for compile-time narrowing
3. **Assertion Functions** — When checks should throw
4. **Const Type Parameters** — TS 5.0 literal inference
5. **Inference Failure** — Debugging type inference issues
