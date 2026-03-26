# Debugging TypeScript

Resolve complex TypeScript errors and module resolution issues.

## Topics

- [Complex Errors](./complex-errors.md) — Common error patterns and solutions
- [Module Resolution](./module-resolution.md) — "Cannot find module" despite file existing

## Quick Diagnostics

```bash
# Trace module resolution
npx tsc --traceResolution > resolution.log 2>&1

# Debug type checking
npx tsc --extendedDiagnostics

# Check config
npx tsc --showConfig
```

## Error Categories

| Error | Cause | Fix |
|-------|-------|-----|
| "Cannot be named" | Missing export | Export type explicitly |
| "Excessively deep" | Circular types | Limit recursion |
| "Cannot find module" | Path mismatch | Check moduleResolution |
