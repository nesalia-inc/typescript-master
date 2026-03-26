# Inference Failure

Why TypeScript sometimes can't infer types, and how to fix it.

## Common Failure Scenarios

### 1. Complex Generic Inference

```typescript
// TypeScript can't infer deeply nested generics
function chain<T, U, V>(a: T, b: U, c: V): [T, U, V] {
  return [a, b, c];
}

// TS infers as any when too complex
const result = chain("hello", 42, true);
//    ^? [string, number, boolean] - OK in this case

// But this fails:
function compose<A, B, C>(fn1: (a: A) => B, fn2: (b: B) => C): (a: A) => C {
  return (a) => fn2(fn1(a));
}

// TypeScript may fail to infer A, B, C in complex cases
```

### 2. Circular References

```typescript
// TypeScript can't resolve circular type references
interface Tree<T> {
  value: T;
  children: Tree<T>[];
}

// When inference creates cycles, TS falls back to any
type DeepTree = Tree<Tree<Tree<...>>> // Eventually gives up
```

### 3. Union Types in Callbacks

```typescript
// Union types in callbacks can confuse inference
function process<T>(
  items: T[],
  fn: (item: T) => T extends string ? string : number
): (string | number)[] {
  return items.map(fn);
}

// When T is a union, behavior may be unexpected
const results = process(["a", "b", "c"], (x) => x.length);
// Results is (string | number)[] - unclear which
```

## How to Diagnose

### Error Messages to Watch

```typescript
// "Type 'X' is not assignable to type 'Y'"
function fail<T extends string>(value: T): T {
  return value; // OK
}
fail(123); // Error: number is not assignable to string

// "Type 'X' could be null" (strictNullChecks)
function demo(value: string | null): string {
  return value; // Error: value could be null
}

// "Parameter 'x' implicitly has 'any' type"
function implicitAny(fn) {
  return fn(42); // fn parameter has implicit any
}
```

### Using `noImplicitAny`

This compiler flag makes TypeScript error on implicit `any`:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true
  }
}
```

When inference fails with `noImplicitAny`, TypeScript tells you where to add types:

```typescript
// Error: Parameter 'fn' implicitly has an 'any' type
function callIt(fn) {  // ^ Add type annotation here
  return fn(42);
}

// Fix: Add explicit types
function callIt(fn: (x: number) => number) {
  return fn(42);
}
```

## Solutions

### 1. Explicit Type Annotations

```typescript
// Let TypeScript know what you expect
function parseJSON<T>(json: string): T {
  return JSON.parse(json) as T;
}

const user = parseJSON<User>('{"name": "John"}');
```

### 2. Type Predicates for Narrowing

```typescript
// When union types are too complex, use predicates
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function process(value: unknown): string | number {
  if (isString(value)) {
    return value.toUpperCase();
  }
  return 0;
}
```

### 3. Const Assertions

```typescript
// When literals widen unexpectedly
const config = {
  port: 3000,
  host: "localhost",
} as const;
// Now config.port is 3000 literal, not number
// config.host is "localhost" literal, not string
```

### 4. Double Assert (use carefully)

```typescript
// When you know better than TypeScript (use sparingly!)
const value = "hello" as unknown as { name: string };

// Only use when:
// 1. You're certain the runtime value is correct
// 2. Adding explicit types would be too verbose
// 3. Working with poorly-typed external libraries
```

### 5. Helper Types for Complex Inference

```typescript
// Create helper types to clarify intent
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type ExtractArrayElement<T> = T extends (infer E)[] ? E : never;

// Use in function signatures
async function fetchAndProcess<T>(
  url: string,
  processor: (data: T) => Promise<T>
): Promise<T> {
  const response = await fetch(url);
  const data: T = await response.json();
  return processor(data);
}
```

## Contextual Typing

TypeScript can infer types from context (where the value is used).

### Example: Object Method Inference

```typescript
const handlers = {
  onClick: () => console.log("clicked"),
  onHover: (e: MouseEvent) => console.log(e.clientX),
};
// TypeScript infers handlers based on usage
```

### Example: Callback Parameter Types

```typescript
// TypeScript infers 'string' for 'name' from the array type
const names = ["Alice", "Bob", "Charlie"];

names.forEach(function(name) {
  console.log(name.toUpperCase()); // name is string
});
```

### Example: Return Type from Context

```typescript
// TypeScript knows this must return { id: number }
function createRecord(id: number) {
  return { id }; // No annotation needed
}
```

### When Contextual Typing Fails

```typescript
// Two possible types conflict
function process(
  items: string[],
  fn: (item: string) => string | number
): string | number[] {
  return items.map(fn);
}

const results = process(["a", "b"], (x) => {
  // Is x string here? Yes, from items
  // But return type is ambiguous
  return x.length; // number
});
```

## Best Practices

| Practice | Why |
|----------|-----|
| Enable `strict: true` | Catches inference failures early |
| Use `noImplicitAny: true` | Forces explicit types where inference fails |
| Add explicit return types on exports | Helps downstream inference |
| Use type predicates for complex narrowing | Makes control flow predictable |
| Avoid `as any` | Masks real problems |
| Prefer `as unknown as X` over `as X` | More intentional |

## Key Takeaways

- Complex generics can exceed TypeScript's inference depth
- Circular types break inference
- `noImplicitAny` flags inference failures
- Explicit annotations are the fix when inference fails
- Contextual typing works top-down but has limits
- Understanding where inference fails helps write better typed code
