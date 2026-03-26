# Quick References

Quick decision trees and checklists for TypeScript work.

## Topics

- [Decision Trees](./decision-trees.md) — Quick "which tool/approach" answers
- [Code Review](./code-review.md) — TypeScript-specific review checklist

## Quick Answers

| Question | Answer |
|----------|--------|
| Type checking only? | `npx tsc --noEmit` |
| Fast linting? | Biome |
| Type testing? | Vitest `expectTypeOf` |
| Monorepo < 20 pkgs? | Turborepo |
| Monorepo > 50 pkgs? | Nx |
