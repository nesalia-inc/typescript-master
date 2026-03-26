# Build Optimization

Build performance patterns for TypeScript projects.

## Project Structure

```
{
  "compilerOptions": {
    // Enable caching
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",

    // Reduce scope
    "include": ["src"],
    "exclude": ["node_modules", "dist", "**/*.test.ts"]
  }
}
```

## Monorepo: Project References

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
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist"
  }
}
```

## Build Commands

```bash
# Incremental build
npx tsc --build --verbose

# Force rebuild
npx tsc --build --force

# Clean build artifacts
npx tsc --build --clean
```

## Common Slow Build Causes

| Cause | Solution |
|-------|----------|
| No incremental | Enable `incremental: true` |
| Rebuilding everything | Configure project references |
| Large node_modules | Use `skipLibCheck: true` |
| Many declaration files | Use `declaration: false` for apps |
| No outDir | Set explicit outDir |

## Fast Feedback Loop

For development, separate type checking from compilation:

```bash
# Fast: type check only (no emit)
npx tsc --noEmit

# Incremental type check
npx tsc --incremental --noEmit
```

## Bundle-Time Type Stripping

For production builds, strip types before bundling:

```typescript
// esbuild.config.js
export default {
  entryPoints: ["src/index.ts"],
  bundle: true,
  platform: "node",
  outfile: "dist/index.js",
  format: "esm",
  target: "node18",
  drop: ["typescript"],
};
```

## Watch Mode

For development, use watch mode:

```bash
# Watch mode (don't use in CI)
npx tsc --watch --incremental
```

## CI Optimization

```yaml
# GitHub Actions example
- name: Type check
  run: npx tsc --noEmit --incremental

- name: Build
  run: npx tsc --build --force
  # Only runs if files changed
```

## Key Takeaways

- Enable `incremental: true` always
- Use project references for monorepos
- Set precise `include` and `exclude`
- Use `skipLibCheck: true` for libraries
- Separate type checking from bundling
- Monitor build times in CI
