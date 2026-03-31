# Common Pitfalls

> **Decision Tree**: Jump to your problem:
> - I don't know the type → `unknown` + Type Guard
> - Too many optional flags → Discriminated Union
> - Need to force a type → Type Guard or `satisfies`
> - Data mutating unexpectedly → `readonly` + Immutability
> - Need constants list → Union of Literals (not Enum)

---

## Quick Fix Reference

| If I see this... | My reaction should be... |
|------------------|------------------------|
| `any` | Use `unknown` (never `any`) |
| Optional flags (`loading?`, `error?`) | Discriminated Union |
| `as MyType` | Type Guard or `satisfies` |
| Unexpected mutations | `readonly` everywhere |
| Constants list | Union of Literals (not Enum) |

---

## 1. When: "I don't know the type yet"

**Pitfall**: Using `any` to "get it working". This disables the compiler entirely.

```typescript
// ❌ Bad: any authorises anything
function process(data: any) {
  data.toUpperCase(); // Potential crash if data is a number
}

// ✅ Good: unknown forces verification
function process(data: unknown) {
  if (typeof data === "string") {
    data.toUpperCase(); // Safe
  }
}
```

---

## 2. When: "My state has multiple flags"

**Pitfall**: Multiple optional properties create impossible states (e.g., `loading: true` AND `data: User` simultaneously).

**Then: Discriminated Unions**

```typescript
// ❌ Bad: Multiple states simultaneously possible
type State = {
  loading?: boolean;
  error?: string;
  data?: User;
};
// Problem: { loading: true, data: user } is allowed but nonsensical

// ✅ Good: Exactly one state at a time
type State =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; data: User }
  | { type: "error"; error: string };
```

---

## 3. When: "Deeply nested mutations"

**Pitfall**: Thinking that `{...obj}` protects from mutation. **Spread is shallow** (only one level deep).

**Then: `Readonly<T>` and Immutability**

```typescript
const user = { name: "John", meta: { role: "admin" } };
const admin = { ...user, isAdmin: true };

// user.name is NOT changed (primitive copy at first level)
admin.name = "Jane";
console.log(user.name); // "John" ✅

// BUT nested objects share the same reference!
admin.meta.role = "guest";
console.log(user.meta.role); // ❌ "guest" - both point to same object!
```

**Solution: Use `Readonly` or immutable libraries like Immer**

```typescript
function updateRole(user: Readonly<User>): User {
  // user.meta.role = "guest"; // ❌ Compile error immediately
  return { ...user, meta: { ...user.meta, role: "guest" } }; // Proper immutable update
}
```

---

## 4. When: "Forcing a type"

**Pitfall**: Abusing `as MyType`. Telling TypeScript "trust me" when you shouldn't.

**Then: Type Guards or `satisfies`**

```typescript
// ❌ Bad: Lying to the compiler
const user = JSON.parse(str) as User;
// user.name is NOT guaranteed - runtime can still be anything

// ✅ Good: Actual verification
if (isValidUser(user)) {
  // Here, user is truly a User
}

function isValidUser(obj: unknown): obj is User {
  return typeof obj === "object"
    && obj !== null
    && "name" in obj
    && typeof (obj as User).name === "string";
}
```

---

## 5. When: "Creating a constants list"

**Pitfall**: Using numeric `enum`. They're not type-safe (any number allowed) and generate bloated JS.

**Then: Union of Literals or Const Object**

```typescript
// ❌ Bad: Numeric enum
enum Status { Pending, Active }
// Status.Active could be set to 999 (no compiler error!)

// ✅ Good: String union
type Status = "pending" | "active";

// ✅ Or const object for runtime values
const Status = {
  Pending: "pending",
  Active: "active"
} as const;
type Status = typeof Status[keyof typeof Status];
```

---

## 6. When: "Complex types crashing the compiler"

**Pitfall**: Recursive conditional types causing "Type instantiation is excessively deep".

**Then: Since TS 4.2+, simple circular refs are fine**

```typescript
// ✅ This is VALID since TS 4.2+
interface TreeNode {
  value: string;
  children?: TreeNode[];
}

// ⚠️ This CAUSES problems: recursive conditional types
type DeepReadonly<T> =
  T extends (...args: any[]) => any ? T :
  T extends object ? { readonly [K in keyof T]: DeepReadonly<T[K]> } :
  T;
// Complex nested objects can hit compiler limits
```

**Solution: Add depth limits or use interfaces instead of type aliases**

```typescript
type DeepReadonly<T, D extends number = 5> =
  [D] extends [never] ? never :
  T extends (...args: any[]) => any ? T :
  T extends object ? { readonly [K in keyof T]: DeepReadonly<T[K], Prev[D]> } :
  T;

type Prev[N extends number] = N extends 0 ? never : [-1, 0, 1, 2, 3, 4][N];
```

---

## 7. When: "Ignoring null checks"

**Pitfall**: Thinking `strictNullChecks: false` is "simpler". It creates runtime crashes.

**Then: Enable `strictNullChecks` and handle null explicitly**

```typescript
// With strictNullChecks: false ❌
function getUser() { return { name: "John" }; }
const user = getUser();
user.email.toUpperCase(); // Runtime crash!

// With strictNullChecks: true ✅
function getUser(): { name: string } | null { return null; }
const user = getUser();
if (user) {
  console.log(user.name); // Safe!
}
```

---

## 8. When: "Complex types without testing"

**Pitfall**: Building complex utility types without validation. "It looks right" isn't enough.

**Then: Use `.test-d.ts` files for type testing**

```typescript
// type.test-d.ts
type _ = Expect<Equal<Strings<{ a: string; b: number }>, never>>;
type _ = Expect<Equal<Strings<{ a: string; b: string }>, { a: string; b: string }>>;
```

---

## 🛡️ Senior Code Review Checklist

### Type Safety
- [ ] No `any` without explicit justification comment
- [ ] `unknown` used instead when type is truly unknown
- [ ] Every `as` is justified or replaced by Type Guard

### Null Handling
- [ ] `strictNullChecks: true` enabled
- [ ] Null/undefined cases explicitly handled
- [ ] No forced unwrapping without checks

### Immutability
- [ ] Properties that shouldn't change are `readonly`
- [ ] Nested object updates use proper spreading
- [ ] No mutations in shared state

### State Management
- [ ] Discriminated unions for mutually exclusive states
- [ ] No impossible states possible (e.g., `loading: true` + `data: User`)
- [ ] State transitions are exhaustively handled

### Type Complexity
- [ ] Complex types (>10 lines) have explanatory comments
- [ ] No untyped recursive types without depth limits
- [ ] Utility types tested with `.test-d.ts`

### Architecture
- [ ] Discriminated unions preferred over optional flags
- [ ] `interface` preferred over `type` for object shapes
- [ ] Enums avoided in favor of const objects or unions

---

## Key Takeaways

1. **Be honest with the compiler**: No lies (`as`) and no ignorance (`any`)

2. **Make the impossible impossible**: Discriminated unions eliminate invalid states

3. **Remember runtime**: TypeScript disappears at runtime—ensure types match actual data

4. **Spread is shallow**: Nested objects share references; use `Readonly` or immutable patterns

5. **Complex types need tests**: Use `.test-d.ts` files to validate utility types

6. **Circular refs in interfaces are fine** since TS 4.2+—recursive conditionals are the danger
