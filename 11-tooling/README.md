# Modern Tooling

TypeScript-aware linting and type testing tools.

## Topics

- [Biome vs ESLint](./biome-vs-eslint.md) — Choose the right linter
- [Type Testing](./type-testing.md) — Test your types

## Tool Selection

| Need | Tool |
|------|------|
| Fast lint + format | Biome |
| Type-aware linting | ESLint + typescript-eslint |
| Type testing | Vitest `expectTypeOf` |
| Type assertions | `tsd` |

## Quick Reference

```bash
# Biome (fastest)
npx biome check .
npx biome format .

# ESLint (most complete)
npx eslint src
npx eslint src --fix

# Type test
npx vitest run avatar.test-d.ts
```
