# Module Resolution Mysteries

Debug "Cannot find module" errors.

## Module Resolution Checklist

```bash
# 1. Check moduleResolution setting
npx tsc --showConfig | grep moduleResolution

# 2. Trace resolution
npx tsc --traceResolution 2>&1 | grep "some-module"

# 3. Verify file exists
ls -la src/some-module.ts
```

## Common Causes

### 1. Path Mismatch

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@utils/*": ["src/utils/*"]
    }
  }
}
```

Usage:
```typescript
// Correct
import { foo } from "@utils/foo";

// Wrong (relative path not following alias)
import { foo } from "./utils/foo";
```

### 2. Wrong moduleResolution Strategy

```json
{
  "compilerOptions": {
    // For Node.js with ESM
    "moduleResolution": "NodeNext",

    // For bundlers (webpack, vite, etc.)
    "moduleResolution": "Bundler",

    // For React Native
    "moduleResolution": "Node"
  }
}
```

### 3. Monorepo: Workspace Protocol

```json
{
  "dependencies": {
    "@repo/shared": "workspace:*"
  }
}
```

### 4. Extension Mismatch

```typescript
// File: src/utils.ts
// Wrong
import { foo } from "./utils"; // May not find .ts

// Correct
import { foo } from "./utils.js"; // Explicit .js extension for ESM
```

## Runtime vs Compile Time

TypeScript paths work at compile time only. For runtime:

```bash
# ts-node: use tsconfig-paths
npx ts-node -r tsconfig-paths/register src/index.ts

# Node ESM: requires experimental loader or build step
node --loader tsconfig-paths/esm-loader src/index.ts
```

## Clearing Caches

```bash
# Clear TS cache
rm -rf node_modules/.cache .tsbuildinfo

# Clear npm cache (if package resolution issue)
npm cache clean --force

# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

## Diagnostic Commands

```bash
# Show effective config
npx tsc --showConfig

# Trace what tsconfig is being used
npx tsc --noEmit --explainFiles > files.txt

# Check for duplicate tsconfigs
find . -name "tsconfig*.json" -not -path "*/node_modules/*"
```

## Key Takeaways

- `moduleResolution` must match your environment
- Paths are compile-time only, not runtime
- Use `workspace:*` for monorepo packages
- Clear caches when troubleshooting
- Use `--traceResolution` to debug
