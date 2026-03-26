# Performance Optimization

Diagnose and fix TypeScript type checking and build performance issues.

## Topics

- [Type Checking](./type-checking.md) — Diagnose and fix slow type checking
- [Build Optimization](./build-optimization.md) — Build performance patterns

## Quick Diagnostics

```bash
# Extended diagnostics for type checking
npx tsc --extendedDiagnostics --incremental false | grep -E "Check time|Files:|Lines:|Nodes:"

# Generate trace for deep analysis
npx tsc --generateTrace trace --incremental false
```

## Common Symptoms

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Slow first compile | No incremental | Enable `incremental: true` |
| Slow rebuilds | Large .tsbuildinfo | Verify caching |
| Excessive memory | Too many files | Configure `include/exclude` |
| "Excessively deep" errors | Circular types | Simplify generics |

## At a Glance

Type checking slowness → Use `skipLibCheck`, incremental builds, simplify types
Build slowness → Check bundler config, enable caching
