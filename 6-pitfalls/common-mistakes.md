# Common Pitfalls

Mistakes to avoid when working with TypeScript's type system.

## 1. Overusing `any`

The most common mistake. `any` defeats the purpose of TypeScript.

```typescript
// Bad
function process(data: any) {
  return data.value; // No safety!
}

// Good
function process<T extends { value: unknown }>(data: T) {
  return data.value; // T is known
}
```

## 2. Ignoring Strict Null Checks

```typescript
// With strictNullChecks: false (bad)
function getUser() {
  return { name: "John" };
}

const user = getUser();
console.log(user.email.toUpperCase()); // Runtime error!

// With strictNullChecks: true (good)
function getUser(): { name: string } | null {
  return null;
}

const user = getUser();
if (user) {
  console.log(user.name); // Safe!
}
```

## 3. Circular Type References

```typescript
// This causes compiler errors or infinite loops
type Bad = { items: Bad[] }; // Bad!

// Use interface (solved in TS 4.2+)
interface Item {
  items?: Item[];
}
```

## 4. Forgetting readonly

```typescript
const user = { name: "John" };
const admin = { ...user, isAdmin: true };

// user.name is still mutable if not readonly
admin.name = "Jane"; // Changes user too if spread was shallow!

// Use readonly
function processUser(user: Readonly<{ name: string }>) {
  // user.name cannot be modified
}
```

## 5. Not Using Discriminated Unions

```typescript
// Bad: multiple optional properties
type State = {
  loading?: boolean;
  data?: User;
  error?: Error;
};

// Good: exactly one state at a time
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: Error };
```

## 6. Complex Types Without Testing

```typescript
// You think this returns string[], but does it?
type Strings<T> = T extends { [K in keyof T]: string } ? T[K][] : never;

// Without tests, you won't know until runtime
```

## 7. Type Assertions Lying to TypeScript

```typescript
// Dangerous
const user = {
  id: (Math.random() * 1000) | 0,
} as { id: number; name: string };

// user.name is not guaranteed at runtime
console.log(user.name.toUpperCase()); // Crashes!
```

## 8. Over-Engineering Types

```typescript
// Instead of this 50-line generic masterpiece:
type Complex<T, U, V> = // ... hard to understand

// Consider a simpler approach:
type SimpleResult = {
  items: Item[];
  count: number;
  hasMore: boolean;
};
```

## 9. Using Enums When Unions Work

```typescript
// Enum (not recommended)
enum Status {
  Active,
  Inactive,
}

// Const object (preferred)
const Status = {
  Active: "active",
  Inactive: "inactive",
} as const;
type Status = (typeof Status)[keyof typeof Status];
```

## 10. Not Handling Edge Cases

```typescript
// Empty array edge case
type First<T> = T extends [infer F, ...infer Rest] ? F : never;

type A = First<[]>; // never (correct)
type B = First<[string]>; // string (correct)
```

## Quick Reference Table

| Mistake | Problem | Solution |
|---------|---------|----------|
| `any` everywhere | No type safety | Use `unknown` or generics |
| No strict null | Runtime errors | Enable `strictNullChecks` |
| Circular refs | Infinite type instantiation | Use interfaces |
| Missing readonly | Mutable data bugs | Use `Readonly<T>` |
| Complex conditionals | Slow compilation | Simplify, cache types |
| Assertions | Lying to compiler | Use proper types |
| Enums | Not type-safe | Use const objects |

## Prevention Checklist

- [ ] Enable `strict: true`
- [ ] No `any` without reason
- [ ] Test complex types
- [ ] Handle null/undefined
- [ ] Use discriminated unions
- [ ] Prefer `interface` for objects
- [ ] Use `readonly` where appropriate
- [ ] Keep types simple and readable
