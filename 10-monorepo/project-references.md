# TypeScript Project References

Use TypeScript's built-in monorepo support for type-safe builds.

## Basic Setup

```json
// Root tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/api" },
    { "path": "./apps/web" }
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
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Key Settings

```json
{
  "compilerOptions": {
    // Required for project references
    "composite": true,

    // Generate .d.ts files
    "declaration": true,

    // Enable source maps for .d.ts
    "declarationMap": true,

    // Don't include this project in parent builds
    "references": []
  }
}
```

## Build Commands

```bash
# Build all references
npx tsc --build

# Clean build artifacts
npx tsc --build --clean

# Force rebuild
npx tsc --build --force

# Build specific project
npx tsc --build --projects packages/core

# Verbose output
npx tsc --build --verbose
```

## Incremental Builds

```bash
# First build creates .tsbuildinfo
npx tsc --build

# Subsequent builds are incremental
npx tsc --build
```

## With Build Tools

### Vite

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [tsconfigPaths()],
  build: {
    // TypeScript project references work with bundler
  },
});
```

### Webpack

```typescript
// webpack.config.ts
export default {
  // Enable build references
  resolve: {
    alias: {
      "@core": require.resolve("./packages/core/src"),
    },
  },
};
```

## Common Issues

### "Project references cannot be built"

```bash
# Build in correct order
npx tsc --build --verbose

# Check if circular
npx tsc --build --dry
```

### OutDir conflicts

```json
// Each project needs unique outDir
{
  "compilerOptions": {
    "outDir": "./dist"
  }
}
```

## Monorepo with npm/yarn/pnpm

```json
// packages/core/package.json
{
  "name": "@repo/core",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts"
}
```

```json
// apps/web/package.json
{
  "dependencies": {
    "@repo/core": "workspace:*"
  }
}
```

## Key Takeaways

- Use `composite: true` for referenced projects
- Always `declaration: true` and `declarationMap: true`
- TypeScript handles build order automatically
- Works with any bundler or runtime
- Not as feature-rich as Turborepo/Nx but simpler
