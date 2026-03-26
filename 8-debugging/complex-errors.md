# Complex Error Patterns

Common TypeScript errors and their solutions.

## "The inferred type of X cannot be named"

**Cause:** Missing type export or circular dependency

```typescript
// Error: The inferred type of 'result' cannot be named
function createPair<T, U>(a: T, b: U) {
  return { a, b };
}
```

**Fix:** Export the required type explicitly

```typescript
export function createPair<T, U>(a: T, b: U): { a: T; b: U } {
  return { a, b };
}

// Or use ReturnType
export function createPair<T, U>(a: T, b: U) {
  return { a, b };
}
export type Pair<T, U> = ReturnType<typeof createPair<T, U>>;
```

## Missing Type Declarations

**Quick fix with ambient declarations:**

```typescript
// types/ambient.d.ts
declare module "some-untyped-package" {
  const value: unknown;
  export default value;
  export = value; // if CJS interop
}
```

## "Excessive stack depth comparing types"

**Cause:** Circular or deeply recursive types

```typescript
// Bad: Infinite recursion
type InfiniteArray<T> = T | InfiniteArray<T>[];

// Good: Limited depth
type NestedArray<T, D extends number = 5> =
  D extends 0 ? T : T | NestedArray<T, [-1, 0, 1, 2, 3, 4][D]>[];
```

## "Type 'X' is not assignable to type 'Y'"

Usually straightforward, but check for:

```typescript
// Unexpected: width and height are string, not number
const dimensions = { width: "100", height: "100" };

// Fix: Ensure literal types
const dimensions: { width: number; height: number } = {
  width: 100,
  height: 100,
};
```

## "Property does not exist on type"

```typescript
// May need type assertion or narrowing
const value = getData() as { name: string };

if ("name" in value) {
  console.log(value.name); // Still error!
}

// Fix: Use discriminated union or type guard
```

## Circular Dependencies

```typescript
// A.ts
import { B } from "./B";
export interface A { b: B }

// B.ts
import { A } from "./A";
export interface B { a: A }

// Solution: Use forward reference
// A.ts
export interface A { b: B }
export type { B } from "./B";

// Or use type-only imports
import type { B } from "./B";
```

## Declaration Files (.d.ts)

For missing type definitions:

```typescript
// global.d.ts
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      DATABASE_URL: string;
      NODE_ENV: "development" | "production";
    }
  }
}
export {};
```

## Generic Instantiation Errors

```typescript
// "Type instantiation is excessively deep"
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Fix: Simplify or limit depth
```

## Key Takeaways

- "Cannot be named" → Export types explicitly
- Missing declarations → Use ambient declarations
- Circular types → Use interfaces, limit recursion
- Excessive depth → Simplify type structure
- Check moduleResolution matches your environment
