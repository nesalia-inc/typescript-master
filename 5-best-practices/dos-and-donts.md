# TypeScript Dos and Don'ts

> **Decision Tree**: Jump to your situation:
> - Describing an object shape → `interface`
> - Handling external data (API, user input) → `unknown` + Type Guard
> - Creating a constants list → Union of Literals (not Enum)
> - Configuring a new project → `strict: true`
> - A type is becoming unreadable → Split into smaller named types

---

## Quick Decision Reference

### `interface` vs `type`

| If your need is... | Use... | Why? |
|-------------------|--------|-------|
| Define object/class shape | `interface` | Better compiler performance, clearer error messages |
| Alias a primitive (`ID = string`) | `type` | Interfaces can't extend primitives |
| Union (`A \| B`) or Intersection | `type` | Impossible with interface |
| Extend existing structure (open-ended) | `interface` | Supports Declaration Merging |

---

## 1. When: Handling External Data (API, User Input)

**Scenario**: "I receive data from an external source and don't know its shape."

### DON'T: Use `any`

```typescript
// ❌ Bad: any disables all type safety
const user = JSON.parse(input) as any;
user.name; // TS doesn't know anything about this
```

### DO: Use `unknown` + Type Guard

```typescript
// ✅ Good: unknown forces validation
const data: unknown = JSON.parse(input);

if (isUser(data)) {
  // TypeScript knows data is User here
  console.log(data.name);
}

// Type guard
function isUser(obj: unknown): obj is User {
  return typeof obj === "object"
    && obj !== null
    && "name" in obj
    && typeof (obj as User).name === "string";
}
```

---

## 2. When: Defining Object Shapes

**Scenario**: "I need to describe a data model in my application."

### DO: Use `interface`

```typescript
// ✅ Preferred for objects
interface User {
  readonly id: string;
  name: string;
  email: string;
  createdAt: Date;
}
```

### DO: Use `readonly` for Immutable Properties

```typescript
interface User {
  readonly id: string;       // Cannot be changed after creation
  readonly createdAt: Date;  // Cannot be changed after creation
  name: string;              // Can be modified
}
```

### DON'T: Use Classes Just for Data Transfer

```typescript
// ❌ Bad: Classes for data transfer are unnecessary complexity
class UserDTO {
  constructor(
    public id: string,
    public name: string
  ) {}
}

// ✅ Good: Plain objects + interface
interface UserDTO {
  id: string;
  name: string;
}
```

---

## 3. When: Creating Union or Complex Types

**Scenario**: "I need to combine types or create aliases."

### DO: Use `type` for Unions and Aliases

```typescript
// ✅ Good
type ID = string | number;
type Result = Success | Error;
type Transform<T> = (input: T) => T;
type Status = "pending" | "active" | "closed";
```

### DON'T: Use Numeric Enums

```typescript
// ❌ Bad: Numeric enum - not type-safe, generates extra JS
enum Status {
  Pending,  // 0
  Active,   // 1
}
// Status.Active = 999; // No compiler error!

// ✅ Good: String union or const object
type Status = "pending" | "active" | "closed";

const Status = {
  Pending: "pending",
  Active: "active",
} as const;
type Status = typeof Status[keyof typeof Status];
```

---

## 4. When: Defining Function Signatures

**Scenario**: "I need to type a callback or exported function."

### DON'T: Use the Generic `Function` Type

```typescript
// ❌ Bad: Too broad, same problem as `any`
callback: Function;

// ✅ Good: Precise function signatures
callback: () => void;
callback: (error: Error, result: string) => void;
callback: (data: User) => Promise<void>;
```

### DO: Use Explicit Return Types for Public APIs

```typescript
// ✅ Good: Return type explicit for library exports
export function parseDate(input: string): Date | null {
  // Implementation
}
```

### DO: Prefer Type Inference

```typescript
// ✅ TypeScript infers: const name: string
const name = "John";

// Use explicit types for clarity in complex cases
const transform: (n: number) => number = (n) => n * 2;
```

---

## 5. When: Using Generics

**Scenario**: "I want my function to work with multiple types while maintaining relationships."

### DON'T: Use Generics When Type Appears Only Once

```typescript
// ❌ Over-engineered: T is only used in one place
function log<T>(message: T): void {
  console.log(message);
}
// Better: just use unknown or no type at all

// ✅ Simple alternative
function log(message: unknown): void {
  console.log(message);
}
```

### DO: Use Generics When Type Links Two Values

```typescript
// ✅ Good: T links input array to return type
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const num = first([1, 2, 3]);   // number | undefined
const str = first(["a", "b"]);  // string | undefined
```

### DO: Use Constraints (`extends`)

```typescript
// ✅ Good: Constraint makes the type useful
function logLength<T extends { length: number }>(item: T): T {
  console.log(item.length);
  return item;
}

logLength("hello");     // OK
logLength([1, 2, 3]);  // OK
logLength(42);         // Error: number has no length
```

---

## 6. When: Handling Null and Undefined

**Scenario**: "I need to safely access potentially missing values."

### DON'T: Use Non-Null Assertion (`!`) Outside Tests

```typescript
// ❌ Bad: Crashes if user is null
const name = user!.name;

// ✅ Good: Optional chaining + nullish coalescing
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

---

## 7. When: Type Assertions

**Scenario**: "I need to tell TypeScript to trust a type."

### DON'T: Use `as` to Lie to the Compiler

```typescript
// ❌ Bad: Lying to TypeScript
const user = {} as User;
// user.name might not exist at runtime!

// ✅ Good: Let the compiler verify
const user: User = {
  id: "1",
  name: "John",
  email: "john@example.com"
};
```

### DO: Use `satisfies` When You Want Validation + Inference

```typescript
// ✅ Good: Validates AND preserves exact types
const config = {
  port: 3000,
  host: "localhost"
} satisfies Record<string, string | number>;

config.port; // number (not string | number)
```

---

## 8. When: Configuring Your Project

**Scenario**: "I start a new project or want to harden an existing one."

### DO: Enable `strict: true`

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### Migration Strategy

Enable strict options **one by one** when migrating legacy code:

```json
{
  "compilerOptions": {
    "strict": false,
    "strictNullChecks": true  // Start here
  }
}
```

---

## 9. When: Creating Configuration Constants

**Scenario**: "I need to define immutable configuration values."

### DO: Use `as const satisfies`

```typescript
const ROUTES = {
  HOME: "/",
  USER: "/user",
  ADMIN: "/admin"
} as const satisfies Record<string, string>;

// ✅ Validated (invalid values cause error)
// ✅ Immutable (cannot be reassigned)
// ✅ Exact types preserved ("/" not just string)
```

---

## 10. When: Complex Types Become Unreadable

**Scenario**: "My utility type is 20 lines of nested conditionals."

### DO: Split Into Named Intermediate Types

```typescript
// ❌ Unreadable: One giant type
type ComplexResult<T, U, V> =
  T extends infer T1 ?
    T1 extends object ?
      { [K in keyof T1]: T1[K] extends infer V1 ? V1 : never } :
      T1 :
    never;

// ✅ Readable: Named, focused types
type UnwrapObject<T> = { [K in keyof T]: T[K] };
type FilterNever<T> = { [K in keyof T as T[K] extends never ? never : K]: T[K] };
```

---

## 🛡️ Senior Code Style Checklist

### Type Safety
- [ ] No `any` without explicit justification comment
- [ ] `unknown` used for external/unknown data
- [ ] `as` used only when necessary or replaced by Type Guard
- [ ] `satisfies` used for config validation

### Object Design
- [ ] `interface` used for object shapes
- [ ] `readonly` used for immutable properties
- [ ] Classes used only for actual behavior (not DTOs)

### Functions
- [ ] Explicit return types on exported functions
- [ ] No `Function` type - use precise signatures
- [ ] Generics used only when linking multiple type parameters

### State Management
- [ ] String unions used instead of numeric enums
- [ ] Optional chaining (`?.`) used instead of `!`
- [ ] Nullish coalescing (`??`) for defaults

### Project Configuration
- [ ] `strict: true` enabled
- [ ] `strictNullChecks: true` enabled
- [ ] New projects start with all strict options

---

## Key Takeaways

1. **Describing an object?** → `interface`

2. **Receiving external data?** → `unknown` + Type Guard (never `any`)

3. **Constants list?** → String union (not Enum)

4. **Freezing config?** → `as const satisfies`

5. **Type becoming unreadable?** → Split into smaller named types

6. **Lying to the compiler (`as`)?** → Ask yourself: "Am I sure?" If yes, add a comment. If no, use a Type Guard.

7. **Inference is your friend** - Don't annotate everything explicitly when TS can infer

8. **Documentation through types** - A well-named type (`AuthenticatedUser`) is better than 3 lines of comments
