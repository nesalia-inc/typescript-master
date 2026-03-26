# Migration Expertise

Strategies for migrating JavaScript to TypeScript and modernizing tooling.

## Topics

- [JS to TS Migration](./js-to-ts.md) — Incremental migration from JavaScript
- [Tools Migration](./tools-migration.md) — Tool transitions (ESLint to Biome, etc.)

## Migration Paths

| From | To | When | Effort |
|------|-----|------|--------|
| JS | TS | Starting fresh on existing codebase | Medium |
| ESLint + Prettier | Biome | Need speed, single tool | Low (1 day) |
| Lerna | Nx/Turborepo | Need caching, parallel builds | High |
| CJS | ESM | Node 18+, modern tooling | High |

## Strategy

1. Enable gradually with `allowJs`
2. Rename files incrementally
3. Add types file by file
4. Enable strict mode feature by feature
