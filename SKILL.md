# TypeScript Master

Comprehensive guidance for mastering TypeScript—from foundational type-level programming to expert-level performance optimization, debugging, and real-world problem solving.

## When to Use This Skill

- Building type-safe libraries or frameworks
- Creating reusable generic components
- Implementing complex type inference logic
- Designing type-safe API clients
- Implementing domain modeling with branded types
- Type system performance bottlenecks
- JavaScript to TypeScript migrations
- Monorepo configuration and build optimization
- Tool upgrades (ESLint to Biome, Lerna to Nx/Turborepo)

## Core Workflow

**Type-First API Design**

1. **Analyze** — Review tsconfig, type coverage, build performance
2. **Design** — Create branded types, generics, utility types
3. **Implement** — Write type guards, discriminated unions, conditional types; run `tsc --noEmit`
4. **Optimize** — Configure project references, incremental compilation; run `tsc --noEmit` again
5. **Test** — Validate type coverage with `type-coverage`; confirm zero errors before deployment

## Project Analysis Workflow

Always start by understanding the project setup:

```bash
# Core versions
npx tsc --version
node -v

# Detect tooling ecosystem
node -e "const p=require('./package.json');console.log(Object.keys({...p.devDependencies,...p.dependencies}||{}).join('\n'))" 2>/dev/null | grep -E 'biome|eslint|prettier|vitest|jest|turborepo|nx|trpc' || echo "No tooling detected"

# Check for monorepo
(test -f pnpm-workspace.yaml || test -f lerna.json || test -f nx.json || test -f turbo.json) && echo "Monorepo detected"
```

## Validation Approach

After analysis, validate thoroughly:

```bash
# Fast fail (no watch processes)
npm run -s typecheck || npx tsc --noEmit
npm test -s || npx vitest run --reporter=basic --no-watch

# Type coverage validation
npx type-coverage --detail
```

## Structure

```
typescript-master/
├── SKILL.md                    # This file
├── 1-core-concepts/            # Foundational concepts
│   ├── generics.md
│   ├── conditional-types.md
│   ├── mapped-types.md
│   ├── template-literal-types.md
│   └── utility-types.md
├── 2-advanced-types/           # Deep type-level programming
│   ├── branded-types.md
│   ├── satisfies-operator.md
│   └── recursive-types.md
├── 3-patterns/                 # Real-world patterns
│   ├── type-safe-event-emitter.md
│   ├── type-safe-api-client.md
│   ├── builder-pattern.md
│   ├── deep-readonly-partial.md
│   ├── form-validation.md
│   └── discriminated-unions.md
├── 4-type-inference/           # Inference techniques
│   ├── infer-keyword.md
│   ├── type-guards.md
│   └── assertion-functions.md
├── 5-best-practices/           # Guidelines
│   ├── dos-and-donts.md
│   ├── type-testing.md
│   └── performance.md
├── 6-pitfalls/                 # Common mistakes
│   └── common-mistakes.md
├── 7-performance/             # Performance optimization
│   ├── type-checking.md
│   └── build-optimization.md
├── 8-debugging/                # Real-world problem resolution
│   ├── complex-errors.md
│   └── module-resolution.md
├── 9-migration/                # Migration expertise
│   ├── js-to-ts.md
│   └── tools-migration.md
├── 10-monorepo/                # Monorepo patterns
│   ├── nx-vs-turborepo.md
│   └── project-references.md
├── 11-tooling/                 # Modern tooling
│   └── biome-vs-eslint.md
├── 12-checklists/               # Quick references
│   ├── decision-trees.md
│   └── code-review.md
└── references/                  # Detailed reference guides
    ├── type-guards.md
    ├── patterns.md
    ├── configuration.md
    └── trpc.md
```

## How to Read This

1. **Start with Core Concepts** — understand generics, conditional types, mapped types, template literal types, and utility types
2. **Study Advanced Types** — branded types, satisfies operator, recursive types
3. **Apply Advanced Patterns** — real-world scenarios like API clients and event emitters
4. **Master Type Inference** — extract and infer types automatically
5. **Follow Best Practices** — adopt recommended patterns and avoid pitfalls
6. **Optimize Performance** — type checking and build optimization
7. **Debug Effectively** — complex errors and module resolution
8. **Migrate with Confidence** — JavaScript to TypeScript and tool migrations
9. **Scale with Monorepos** — Nx, Turborepo, and project references

## Constraints

### MUST DO

- Enable strict mode with all compiler flags
- Use type-first API design
- Implement branded types for domain modeling
- Use `satisfies` operator for type validation
- Create discriminated unions for state machines
- Use type predicates for narrowing
- Generate declaration files for libraries
- Optimize for type inference

### MUST NOT DO

- Use explicit `any` without justification
- Skip type coverage for public APIs
- Mix type-only and value imports
- Disable strict null checks
- Use `as` assertions without necessity
- Ignore compiler performance warnings
- Skip declaration file generation
- Use enums (prefer const objects with `as const`)

## Quick Reference

### Generics

```typescript
function identity<T>(value: T): T {
  return value;
}

function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}
```

### Conditional Type

```typescript
type IsString<T> = T extends string ? true : false;
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

### Mapped Type

```typescript
type Readonly<T> = { readonly [P in keyof T]: T[P] };
type Partial<T> = { [P in keyof T]?: T[P] };
```

### Template Literal Type

```typescript
type EventName = "click" | "focus";
type Handler = `on${Capitalize<EventName>}`; // "onClick" | "onFocus"
```

### Branded Type

```typescript
declare const __brand: unique symbol;
type Brand<B> = { [__brand]: B };
export type Branded<T, B> = T & Brand<B>;
```

### Discriminated Union

```typescript
type RequestState =
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; error: Error };
```

### Recommended tsconfig

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "isolatedModules": true,
    "declaration": true,
    "declarationMap": true,
    "incremental": true,
    "skipLibCheck": false
  }
}
```
