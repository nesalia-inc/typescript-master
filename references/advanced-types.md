# Advanced Types Reference

Generics, conditional types, mapped types, and template literal types.

## Generics

### Basic Generic Function

```typescript
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(42);
const str = identity("hello"); // TypeScript infers
```

### Generic Constraints

```typescript
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): T {
  console.log(item.length);
  return item;
}
```

### Multiple Type Parameters

```typescript
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}
```

## Conditional Types

### Basic Conditional

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```

### With infer

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type R = ReturnType<() => User>; // User
```

### Distributive

```typescript
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>; // string[] | number[]
```

## Mapped Types

### Basic

```typescript
type Readonly<T> = { readonly [P in keyof T]: T[P] };
type Partial<T> = { [P in keyof T]?: T[P] };
```

### Key Remapping

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
```

## Template Literal Types

```typescript
type EventName = "click" | "focus";
type Handler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus"
```

### Pattern Matching

```typescript
type PropEventSource<T> = {
  on<K extends keyof T & string>(
    event: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void;
};
```

## Branded Types

```typescript
type Brand<T, B extends string> = T & { readonly __brand: B };
type UserId = Brand<string, "UserId">;

const toUserId = (id: string): UserId => id as UserId;
```

## Recursive Types

```typescript
type DeepReadonly<T> = T extends (...args: any[]) => any
  ? T
  : T extends object
    ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
    : T;
```
