# Monorepo Management

Strategies for TypeScript monorepos.

## Topics

- [Nx vs Turborepo](./nx-vs-turborepo.md) — Choose the right tool
- [Project References](./project-references.md) — TypeScript project references

## Decision Matrix

| Tool | Best For | Packages | Complexity |
|------|----------|----------|-------------|
| Turborepo | Simple structure, need speed | < 20 | Low |
| Nx | Complex dependencies, visualization | > 50 | High |
| Project References | TypeScript-only, no caching | Any | Medium |

## Quick Start

```bash
# Turborepo
npm install --save-dev turbo
npx turbo init

# Nx
npx create-nx-workspace@latest
```
