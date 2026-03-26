# TypeScript Dos and Don'ts

Practical guidelines for writing better TypeScript.

## Types and Interfaces

### DO: Use `interface` for Object Shapes

```typescript
// Preferred
interface User {
  id: string;
  name: string;
  email: string;
}
```

### DO: Use `type` for Unions and Complex Types

```typescript
// Preferred
type ID = string | number;
type Result = Success | Error;
type Transform<T> = (input: T) => T;
```

### DON'T: Use `any`

```typescript
// Avoid
function processData(data: any) {
  return data.value; // No type safety!
}

// Preferred
function processData<T>(data: T) {
  // work with T
}
```

### DO: Use `unknown` When Type is Unknown

```typescript
// unknown requires you to narrow the type
function parseJSON(json: string): unknown {
  return JSON.parse(json);
}

// You must narrow before using
const data = parseJSON(jsonString);
if (typeof data === "object" && data !== null) {
  // now safe to use
}
```

## Null and Undefined

### DON'T: Use Non-Null Assertion Without Reason

```typescript
// Avoid
const name = user!.name; // DANGER: crashes if user is null

// Preferred: Handle the null case
const name = user?.name ?? "Anonymous";
```

### DO: Use Optional Chaining and Nullish Coalescing

```typescript
// Chain safely
const street = user?.address?.street;

// Provide defaults
const name = user?.name ?? "Unknown";
```

### DO: Enable Strict Null Checks

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

## Functions

### DO: Use Explicit Return Types for Public APIs

```typescript
// Preferred for library code
export function parseDate(input: string): Date | null {
  // ...
}
```

### DON'T: Use `Function`

```typescript
// Avoid
callback: Function;

// Preferred
callback: () => void;
callback: (error: Error) => void;
```

### DO: Prefer Type Inference

```typescript
// TypeScript infers: const name: string
const name = "John";

// Explicit type for clarity in complex cases
const transform: (n: number) => number = (n) => n * 2;
```

## Generics

### DO: Use Constraints

```typescript
// Constrained makes the type useful
function logLength<T extends { length: number }>(item: T): T {
  console.log(item.length);
  return item;
}
```

### DON'T: Use Generics When Not Needed

```typescript
// Over-engineered
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

// Simple when T isn't used elsewhere
function first(arr: unknown[]): unknown {
  return arr[0];
}
```

## Classes

### DO: Use readonly for Immutable Properties

```typescript
class User {
  readonly id: string;
  readonly createdAt: Date;

  constructor(id: string) {
    this.id = id;
    this.createdAt = new Date();
  }
}
```

### DON'T: Use Classes When Functions Suffice

```typescript
// Avoid unnecessary OOP
class Utils {
  static formatDate(date: Date): string {
    // ...
  }
}

// Simple export is fine
export function formatDate(date: Date): string {
  // ...
}
```

## Enums and Union Types

### PREFER: Union Types Over Enums

```typescript
// Preferred: const objects
const Status = {
  Pending: "pending",
  Active: "active",
  Closed: "closed",
} as const;

type Status = (typeof Status)[keyof typeof Status];

// Over enum
enum Status {
  Pending,
  Active,
  Closed,
}
```

## Utility Types

### DO: Use Built-in Utilities

```typescript
type PartialUser = Partial<User>;
type ReadonlyUser = Readonly<User>;
type UserPreview = Pick<User, "id" | "name">;
type UserWithoutPassword = Omit<User, "password">;
```

### DO: Compose Utility Types

```typescript
type UpdateUser = Partial<Omit<User, "id" | "createdAt">>;

// Deep operations for nested objects
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```

## General

### DO: Enable Strict Mode

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

### DON'T: Use Type Assertions Unless Necessary

```typescript
// Dangerous: lies to TypeScript
const user = {} as User;

// Preferred: describe the shape properly
const user: User = {
  id: "1",
  name: "John",
  email: "john@example.com",
};
```

### DO: Use `const` Assertions for Literals

```typescript
const config = {
  endpoint: "api",
  method: "GET",
} as const;
// Type: { readonly endpoint: "api"; readonly method: "GET"; }
```
