# Performance Considerations

How TypeScript's type system affects compilation speed.

## Complex Types Slow Compilation

Deeply nested conditional and mapped types increase type-checking time:

```typescript
// These compile slower
type DeepNested = A<B<C<D<E>>>>;
type ComplexUnion = Exclude<Extract<...>, ...>;
```

## Simplify Where Possible

```typescript
// Instead of complex conditional chains:
type Complex<T> = T extends string
  ? "string"
  : T extends number
    ? "number"
    : T extends boolean
      ? "boolean"
      : T extends undefined
        ? "undefined"
        : T extends null
          ? "null"
          : "object";

// Consider a lookup object:
type PrimitiveNames = {
  string: "string";
  number: "number";
  boolean: "boolean";
  undefined: "undefined";
  null: "null";
  object: "object";
};

type Simple<T> = PrimitiveNames[typeof T];
```

## Recursive Type Depth

Each level of recursion adds compilation time:

```typescript
// Each level adds ~10-50ms to compile
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};
```

### Mitigation Strategies

```typescript
// 1. Limit recursion depth
type DeepReadonly<T, Depth extends 5 = 5> = Depth extends 0
  ? T
  : {
      readonly [P in keyof T]: T[P] extends object
        ? { [K in keyof T[P]]: T[P][K] }
        : T[P];
    };

// 2. Cache recursive results (for union types)
type DeepReadonlyCached<T> = T extends any ? _DeepReadonly<T> : never;
type _DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? _DeepReadonly<T[P]> : T[P];
};
```

## Orphan Types

Types not referenced by any runtime code are still checked:

```typescript
// These types are checked even if never used
type ComplexUtility<T> = // ... heavy computation
```

### Workaround: Conditional Selection

```typescript
type OnlyIfUsed<T, Use extends true> = Use extends true ? T : never;
```

## Compiler Options

```json
{
  "compilerOptions": {
    // Enable incremental compilation
    "incremental": true,

    // Skip library check (don't re-check node_modules)
    "skipLibCheck": true,

    // Disable辩论type generation if not needed
    "noEmit": true,

    // Enable project references for large codebases
    "composite": true
  }
}
```

## Build Mode

Use `tsc --build` (project references) for monorepos:

```json
// tsconfig.json
{
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/api" }
  ]
}
```

## When to Optimize

| Scenario | Action |
|----------|--------|
| Compile time > 30s | Investigate complex types |
| IDE lag on hover | Simplify or cache types |
| Type checking > 50% of build | Enable incremental, skipLibCheck |

## Signs of Over-Engineered Types

- Type instantiation errors
- "Type instantiation is excessively deep" error
- IDE freezes when editing type files
- Builds pass locally but fail in CI (different hardware)

## Quick Wins

1. Add `"skipLibCheck": true` if not already set
2. Use `// @ts-nocheck` for test files with complex types
3. Split large generic types into smaller pieces
4. Avoid `infer` in deeply nested conditions
5. Use `type` aliases to cache intermediate results
