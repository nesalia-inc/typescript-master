# Advanced Type-Level Programming

Advanced patterns for creating type-safe, domain-modeled applications.

## Topics

- [Branded Types](./branded-types.md) — Nominal types for domain primitives
- [Conditional Types](./conditional-types.md) — Recursive type manipulation
- [Template Literals](./template-literals.md) — Template literal type magic
- [Satisfies Operator](./satisfies-operator.md) — TS 5.0+ constraint validation
- [Higher Kinded Types](./higher-kinded-types.md) — Type constructors taking type constructors

## At a Glance

| Pattern | Use Case | Key Technique |
|---------|----------|---------------|
| Branded types | Domain primitives | `K & { __brand: T }` |
| Conditional types | Type-level logic | `T extends U ? X : Y` |
| Template literals | String APIs | `` `${K}Changed` `` |
| Satisfies | Validation | `satisfies Record<string, string>` |
| HKTs | Functor/Monad patterns | Class + `this["_1"]` encoding |

## When to Use Each Pattern

### Branded Types

Use when you need nominal typing for domain primitives that share the same underlying type.

| Use Case | Example |
|----------|---------|
| Domain IDs | `UserId`, `PostId`, `OrderId` |
| Currency | `USD`, `EUR` with type-safe arithmetic |
| Validated strings | `EmailAddress`, `Password` |
| API response types | `ApiSuccess<T>`, `ApiFailure` |

### Conditional Types

Use when you need type-level branching or recursive type manipulation.

| Use Case | Example |
|----------|---------|
| Type extraction | `Awaited<Promise<T>>`, `ReturnType<T>` |
| Union filtering | `NonNullable<T>`, `SuccessState<S>` |
| Recursive types | `DeepReadonly<T>`, `DeepPartial<T>` |

### Template Literals

Use when you need type-safe string patterns.

| Use Case | Example |
|----------|---------|
| Event names | `"user:created"`, `"post:updated"` |
| API endpoints | `"GET /api/v1/users"` |
| CSS classes | `"bg-${color}-${variant}"` |

### Satisfies Operator

Use when you want to validate a value against a type without widening.

| Use Case | Example |
|----------|---------|
| Config validation | `satisfies Record<string, Color>` |
| Route definitions | `satisfies Endpoints` |
| Theme objects | `satisfies Theme` |

### Higher Kinded Types

Use when building functional programming abstractions (Functors, Monads).

| Use Case | Example |
|----------|---------|
| Functor mapping | `Option<T>`, `Result<T, E>` |
| Query builders | Composable SQL-like builders |
| Plugin systems | Type-safe object enrichment |

## How to Read This

1. **Start with Branded Types** — Domain modeling is the foundation of type-safe code
2. **Master Conditional Types** — Enable sophisticated type-level logic
3. **Template Literals** — Create precise string-based APIs
4. **Satisfies Operator** — Validate without losing inference
5. **HKTs** — Advanced functional patterns (for library authors)
