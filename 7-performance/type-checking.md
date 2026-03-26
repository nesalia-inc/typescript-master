# Type Checking Performance

Diagnose and fix slow TypeScript type checking.

## Diagnosing Issues

```bash
# Basic diagnostics
npx tsc --extendedDiagnostics --incremental false | grep -E "Check time|Files:|Lines:|Nodes:"

# Memory usage
node --max-old-space-size=8192 node_modules/typescript/lib/tsc.js

# Trace module resolution
npx tsc --traceResolution > resolution.log 2>&1
grep "Module resolution" resolution.log
```

## Common Issues and Fixes

### 1. "Type instantiation is excessively deep"

```typescript
// Bad: Recursive type without depth limit
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Better: Add depth parameter
type DeepPartial<T, Depth extends 5 = 5> =
  Depth extends 0 ? T : {
    [P in keyof T]?: T[P] extends object
      ? { [K in keyof T[P]]: T[P][K] }  // Simplified
      : T[P];
  };
```

### 2. Slow type checking on large codebase

```json
// tsconfig.json optimizations
{
  "compilerOptions": {
    "skipLibCheck": true,           // Skip .d.ts files
    "incremental": true,             // Cache build info
    "disableReferencedProjectLoad": true
  },
  "exclude": ["node_modules", "dist", "build"]
}
```

### 3. Complex union types

```typescript
// Bad: Large union
type AllTheThings = A | B | C | D | ... | Z;

// Better: Split into related groups
type Group1 = A | B | C;
type Group2 = D | E | F;
```

### 4. Circular type references

```typescript
// Bad
type Tree<T> = { value: T; children: Tree<T>[] };

// Better: Use interface
interface Tree<T> {
  value: T;
  children: Tree<T>[];
}
```

## Performance Patterns

### Use Type Aliases to Cache

```typescript
//重复 complex calculations
type Complex<T> = /* heavy computation */;

// Instead of inline usage, cache:
type CachedComplex<T> = Complex<T>;
```

### Prefer Interfaces Over Type Aliases

```typescript
// Type alias with intersection can be slow
type Slow = A & B & C & { /* more */ };

// Interface extends can be faster
interface Fast extends A, B, C {
  // properties
}
```

### Avoid `extends` in Generics When Possible

```typescript
// This triggers instantiation
function process<T extends string | number | boolean | object>(t: T) {}

// Often equivalent but faster:
function process<T>(t: T) {
  // runtime check if needed
}
```

## Build Mode for Large Projects

Use `tsc --build` for incremental compilation:

```json
// tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true
  }
}
```

```bash
# Build with references
npx tsc --build --verbose
```

## Monitoring CI

```yaml
# Log type check times
- name: Type check
  run: |
    START=$(date +%s)
    npx tsc --noEmit
    END=$(date +%s)
    echo "Type check took $((END-START)) seconds"
```

## Key Takeaways

- Use `skipLibCheck: true` for library code
- Enable `incremental: true` for all projects
- Limit recursive type depth
- Split large unions
- Use interfaces over type aliases for complex objects
- Monitor in CI to catch regressions
