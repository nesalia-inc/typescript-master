# Decision Trees

Quick answers to common TypeScript questions.

## "Which tool should I use?"

```
Need type checking only?
├── Yes → tsc --noEmit
└── No, need linting?
    ├── Need type-aware linting?
    │   ├── Yes → ESLint + typescript-eslint
    │   └── No, want speed?
    │       ├── Yes → Biome
    │       └── No → ESLint + Prettier

Need build caching?
├── Simple (< 20 packages)? → Turborepo
└── Complex (> 50 packages)? → Nx

Need type testing?
├── Yes → Vitest expectTypeOf
└── No → Runtime tests only
```

## "How do I fix this error?"

```
"Cannot find module" despite file existing?
├── Check moduleResolution matches environment
├── Verify baseUrl/paths alignment
├── Try clearing: rm -rf node_modules/.cache .tsbuildinfo
└── Check workspace protocol (workspace:*)

"Type instantiation is excessively deep"?
├── Limit recursive type depth
├── Simplify generic constraints
└── Use interface instead of type alias

"The inferred type cannot be named"?
├── Export type explicitly
├── Use ReturnType<> helper
└── Break circular imports

Slow type checking?
├── Enable skipLibCheck: true
├── Enable incremental: true
└── Simplify complex types
```

## "How do I migrate?"

```
JavaScript → TypeScript
├── Enable allowJs + checkJs first
├── Rename files incrementally
└── Add types gradually

ESLint → Biome
├── Install: npm install @biomejs/biome
├── Run: npx biome init
└── Migrate: npx biome migrate eslint

Lerna → Turborepo
├── Install turbo
├── Create turbo.json pipeline
└── Update scripts

CJS → ESM
├── Set "type": "module" in package.json
├── Update imports to use .js extension
└── Use dynamic imports for CJS
```

## "How do I optimize performance?"

```
Slow type checking?
├── skipLibCheck: true
├── incremental: true
└── Configure include/exclude precisely

Slow builds?
├── Enable incremental builds
├── Use project references
└── Check bundler config

Slow tests?
├── Vitest with worker_threads
└── Avoid type checking in tests
```
