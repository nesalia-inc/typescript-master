# TypeScript Code Review Checklist

Focus areas when reviewing TypeScript/JavaScript code.

## Type Safety

- [ ] No implicit `any` types
- [ ] `unknown` used where type is uncertain
- [ ] Strict null checks enabled and properly handled
- [ ] Type assertions (`as`) are minimal and justified
- [ ] Generic constraints properly defined
- [ ] Discriminated unions for error handling
- [ ] Return types explicitly declared for public APIs

## TypeScript Best Practices

- [ ] `interface` preferred for object shapes
- [ ] `type` for unions and complex types
- [ ] Const assertions for literal types
- [ ] Type guards and predicates used correctly
- [ ] Template literal types used appropriately
- [ ] Branded types for domain primitives
- [ ] No type gymnastics when simpler solution exists

## Performance

- [ ] Type complexity doesn't cause slow compilation
- [ ] No excessive type instantiation depth
- [ ] Avoid complex mapped types in hot paths
- [ ] `skipLibCheck: true` used appropriately
- [ ] Project references configured for monorepos

## Module System

- [ ] Consistent import/export patterns
- [ ] No circular dependencies
- [ ] Proper barrel exports (avoid over-bundling)
- [ ] ESM/CJS compatibility handled correctly
- [ ] Dynamic imports for code splitting

## Error Handling

- [ ] Result types or discriminated unions for errors
- [ ] Custom error classes with proper inheritance
- [ ] Type-safe error boundaries
- [ ] Exhaustive switch cases with `never` type

## Code Organization

- [ ] Types co-located with implementation
- [ ] Shared types in dedicated modules
- [ ] Avoid global type augmentation
- [ ] Proper declaration files (`.d.ts`)

## Common Issues to Watch

### Type Safety

```typescript
// Bad: Implicit any
function process(data) { }

// Good: Explicit type
function process(data: Data) { }

// Bad: any
const value: any = getData();

// Good: unknown
const value: unknown = getData();
```

### Null Handling

```typescript
// Bad
const name = user && user.profile && user.profile.name;

// Good
const name = user?.profile?.name;
```

### Object Types

```typescript
// Prefer interface for object shapes
interface User {
  name: string;
  email: string;
}

// Use type for unions
type Status = "pending" | "active" | "closed";
```
