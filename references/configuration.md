# Configuration Reference

tsconfig options, strict mode, and project references.

## Recommended tsconfig

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

## Strict Mode Flags

```json
{
  "compilerOptions": {
    "strict": true,
    // Implies:
    // - strictNullChecks
    // - strictFunctionTypes
    // - strictBindCallApply
    // - strictPropertyInitialization
    // - noImplicitAny
    // - noImplicitThis
    // - alwaysStrict
  }
}
```

## Individual Strict Flags

| Flag | What it does |
|------|--------------|
| `strictNullChecks` | Null/undefined must be handled |
| `noImplicitAny` | No implicit `any` type |
| `noImplicitOverride` | Must use `override` keyword |
| `exactOptionalPropertyTypes` | `?` means maybe undefined, not missing |
| `noUncheckedIndexedAccess` | Array index access may be undefined |
| `isolatedModules` | Each file must be independently compilable |

## Module Settings

```json
{
  "compilerOptions": {
    // For Node.js ESM
    "module": "NodeNext",
    "moduleResolution": "NodeNext",

    // For bundlers
    "module": "ESNext",
    "moduleResolution": "Bundler",

    // For React Native
    "module": "CommonJS",
    "moduleResolution": "Node"
  }
}
```

## Incremental Builds

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo"
  }
}
```

## Project References

```json
// Root tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/api" }
  ]
}
```

```json
// packages/core/tsconfig.json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true
  }
}
```

## Declaration Files

```json
{
  "compilerOptions": {
    "declaration": true,
    "declarationDir": "./dist",
    "declarationMap": true
  }
}
```

## Performance Options

```json
{
  "compilerOptions": {
    "skipLibCheck": true,
    "incremental": true
  },
  "exclude": ["node_modules", "dist"]
}
```
