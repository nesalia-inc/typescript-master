# Advanced Conditional Types

Recursive type manipulation and advanced patterns.

## Basic Conditional Type

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false
```

## Recursive Deep Readonly

```typescript
type DeepReadonly<T> = T extends (...args: any[]) => any
  ? T
  : T extends object
    ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
    : T;

interface Config {
  server: {
    host: string;
    port: number;
  };
}

type ReadonlyConfig = DeepReadonly<Config>;
// All nested properties are readonly
```

## Distributive Conditional Types

When T is a union, the conditional distributes:

```typescript
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>;
// string[] | number[]

// Non-distributive version
type ToArrayStrict<T> = [T] extends [any] ? T[] : never;

type ResultStrict = ToArrayStrict<string | number>;
// (string | number)[]
```

## Template Literal Type Magic

```typescript
type PropEventSource<T> = {
  on<Key extends string & keyof T>(
    eventName: `${Key}Changed`,
    callback: (newValue: T[Key]) => void
  ): void;
};

interface Person {
  name: string;
  age: number;
}

declare const source: PropEventSource<Person>;

// Valid: "nameChanged" and "ageChanged"
source.on("nameChanged", (value) => {
  console.log(value); // string
});

source.on("ageChanged", (value) => {
  console.log(value); // number
});

source.on("invalidChanged", (value) => {}); // Error!
```

## Chaining Inferences

```typescript
// Extract first and rest from tuple
type First<T> = T extends [infer F, ...infer Rest] ? F : never;
type Rest<T> = T extends [infer F, ...infer Rest] ? Rest : never;

type F = First<[string, number, boolean]>; // string
type R = Rest<[string, number, boolean]>; // [number, boolean]
```

## Recursive Awaited

```typescript
type DeepAwaited<T> = T extends Promise<infer U>
  ? DeepAwaited<U>
  : T;

type Result = DeepAwaited<Promise<Promise<User>>>;
// User
```

## Conditional Type with Default

```typescript
type InferDefault<T, Default> = T extends infer U ? U : Default;

type A = InferDefault<string, never>;     // string
type B = InferDefault<unknown, string>;    // unknown (doesn't match)
type C = InferDefault<never, string>;      // string
```

## Common Patterns

### Pick by Value Type

```typescript
type PickByValueType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface Mixed {
  id: number;
  name: string;
  age: number;
  active: boolean;
}

type OnlyStrings = PickByValueType<Mixed, string>;
// { name: string }
```

### Exclude by Function Signature

```typescript
type ExcludeMethods<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any ? never : K]: T[K];
};

class User {
  name: string;
  getName() { return this.name; }
  setName(n: string) { this.name = n; }
}

type DataOnly = ExcludeMethods<User>;
// { name: string }
```

## Performance Warning

Deeply recursive types cause "Type instantiation is excessively deep" errors:

```typescript
// This can cause compilation to fail or be very slow
type DeepPartial<T> =
  T extends (...args: any[]) => any ? T :
  T extends object ? { [K in keyof T]?: DeepPartial<T[K]> } : T;
```

Mitigate by:
- Limiting recursion depth
- Using interfaces instead of type aliases
- Breaking unions into smaller pieces

## Key Takeaways

- Conditional types distribute over unions
- Use `[T] extends [U]` to prevent distribution
- Recursive types are powerful but can be slow
- Template literals + mapped types = type-safe event systems
