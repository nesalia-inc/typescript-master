# Type Guards Reference

Type narrowing, discriminated unions, and assertion functions.

## Type Predicates

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Usage
if (isString(value)) {
  console.log(value.toUpperCase()); // value is string
}
```

## Custom Type Guards

```typescript
interface Cat { meow(): void; }
interface Dog { bark(): void; }
type Animal = Cat | Dog;

function isCat(animal: Animal): animal is Cat {
  return "meow" in animal;
}
```

## typeof Guards

```typescript
function process(value: string | number) {
  if (typeof value === "string") {
    value.toUpperCase(); // value is string
  } else {
    value.toFixed(2); // value is number
  }
}
```

## instanceof Guards

```typescript
class ErrorA extends Error {}
class ErrorB extends Error {}

function handle(error: ErrorA | ErrorB) {
  if (error instanceof ErrorA) {
    error.code; // ErrorA
  } else {
    error.reason; // ErrorB
  }
}
```

## Discriminated Unions

```typescript
type State =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; data: User }
  | { type: "error"; error: Error };

function render(state: State) {
  switch (state.type) {
    case "idle": return "Idle";
    case "loading": return "Loading...";
    case "success": return state.data.name;
    case "error": return state.error.message;
  }
}
```

## Exhaustive Checks

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

function process(state: State): string {
  switch (state.type) {
    case "idle": return "Idle";
    case "loading": return "Loading";
    case "success": return state.data;
    case "error": return state.error;
    default: return assertNever(state);
  }
}
```

## Assertion Functions

```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Not a string");
  }
}

// After call, value is narrowed to string
assertIsString(value);
console.log(value.toUpperCase());
```

## Combining Guards

```typescript
function isNonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```
