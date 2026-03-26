# Utility Types Reference

Built-in and custom utility types.

## Built-in Transformation

```typescript
// Partial<T> - all optional
type Partial<T> = { [P in keyof T]?: T[P] };

// Required<T> - all required
type Required<T> = { [P in keyof T]-?: T[P] };

// Readonly<T> - all readonly
type Readonly<T> = { readonly [P in keyof T]: T[P] };
```

## Selection

```typescript
// Pick - select keys
type Pick<T, K extends keyof T> = { [P in K]: T[P] };

// Omit - remove keys
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```

## Union Helpers

```typescript
// Exclude - remove from union
type Exclude<T, U> = T extends U ? never : T;
type A = Exclude<"a" | "b" | "c", "a">; // "b" | "c"

// Extract - keep in union
type Extract<T, U> = T extends U ? T : never;
type B = Extract<"a" | "b" | "c", "a" | "b">; // "a" | "b"

// NonNullable - remove null/undefined
type NonNullable<T> = T extends null | undefined ? never : T;
```

## Conditional Helpers

```typescript
// ReturnType - extract return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

// Parameters - extract parameters
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

// InstanceType - extract instance type
type InstanceType<T extends new (...args: any[]) => any> =
  T extends new (...args: any[]) => infer R ? R : any;
```

## Record

```typescript
type Record<K extends keyof any, T> = { [P in K]: T };

type Page = Record<"home" | "about", { title: string }>;
// { home: { title: string }; about: { title: string } }
```

## Custom: DeepPartial

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```

## Custom: DeepReadonly

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};
```

## Custom: RequireExactlyOne

```typescript
type RequireExactlyOne<T, Keys extends keyof T = keyof T> =
  Pick<T, Exclude<keyof T, Keys>> &
  { [K in Keys]-?: Required<Pick<T, K>> }[Keys];
```

## Template Literal Utilities

```typescript
type Uppercase<S extends string> = intrinsic;
type Lowercase<S extends string> = intrinsic;
type Capitalize<S extends string> = intrinsic;
type Uncapitalize<S extends string> = intrinsic;
```
