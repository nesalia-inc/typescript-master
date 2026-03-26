# Core Concepts

Foundational TypeScript concepts for building type-safe applications and packages.

## Topics

- [Generics](./generics.md) — Reusable type-flexible components
- [Conditional Types](./conditional-types.md) — Types that depend on conditions
- [Mapped Types](./mapped-types.md) — Transform existing types
- [Template Literal Types](./template-literal-types.md) — String pattern matching
- [Utility Types](./utility-types.md) — Built-in type transformations

## At a Glance

| Concept | Purpose | Key Pattern |
|---------|---------|-------------|
| Generics | Reusable, type-safe components | `T extends Constraint` |
| Conditional Types | Type logic | `T extends U ? X : Y` |
| Mapped Types | Property transformation | `[P in keyof T]: T[P]` |
| Template Literals | String patterns | `` `on${Capitalize<T>}` `` |
| Utility Types | Built-in transformations | `Partial<T>`, `Readonly<T>` |

## When to Use Each Concept

### Generics

Use generics when you need to write reusable code that works with any type while maintaining type safety.

| Use Case | Example |
|----------|---------|
| Building libraries/packages | Event emitters, repositories, middleware |
| Data structures | `Stack<T>`, `Tree<T>`, `Map<K,V>` |
| Type-safe utilities | `Array.map<U>()`, `Promise.then<U>()` |

### Conditional Types

Use conditional types when you need type-level logic that depends on conditions.

| Use Case | Example |
|----------|---------|
| Extracting types | `Awaited<Promise<T>>`, `ReturnType<T>` |
| Filtering unions | `SuccessState<State>` |
| Type-safe APIs | Response unwrapping, query builders |

### Mapped Types

Use mapped types when you need to transform all properties of a type.

| Use Case | Example |
|----------|---------|
| Type transformations | `Partial<T>`, `Readonly<T>` |
| Code generation | API schemas, database columns |
| Property renaming | Getters, key remapping |

### Template Literal Types

Use template literal types when you need type-safe string patterns.

| Use Case | Example |
|----------|---------|
| Event naming | `"user:created"`, `"post:updated"` |
| API endpoints | `"GET /api/v1/users"` |
| CSS classes | `"bg-${color}-${variant}"` |
| i18n keys | `"common.buttons.save"` |

### Utility Types

Use built-in utility types for common type transformations.

| Use Case | Utility |
|----------|---------|
| Make properties optional | `Partial<T>` |
| Pick specific properties | `Pick<T, K>` |
| Remove properties | `Omit<T, K>` |
| Filter unions | `Extract<T, U>`, `Exclude<T, U>` |

## How to Read This

1. **Generics first** — Understanding generics is foundational to all other concepts
2. **Conditional types** — Enable sophisticated type-level logic
3. **Mapped types** — Transform and filter type properties
4. **Template literals** — Create precise string-based types
5. **Utility types** — Apply built-in transformations
