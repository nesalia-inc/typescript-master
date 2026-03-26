# TypeScript Master

Comprehensive TypeScript skill documentation for AI agents and developers.

## Overview

This repository contains detailed guides and references for mastering TypeScript, from foundational concepts to advanced patterns for building type-safe applications and libraries.

## Structure

```
typescript-master/
├── 1-core-concepts/       # Foundational TypeScript concepts
│   ├── generics.md
│   ├── conditional-types.md
│   ├── mapped-types.md
│   ├── template-literal-types.md
│   └── utility-types.md
├── 2-advanced-types/       # Advanced type-level programming
│   ├── branded-types.md
│   ├── conditional-types.md
│   ├── higher-kinded-types.md
│   ├── satisfies-operator.md
│   └── template-literals.md
├── 3-patterns/            # Real-world patterns
│   ├── builder-pattern.md
│   ├── deep-readonly-partial.md
│   ├── discriminated-unions.md
│   ├── form-validation.md
│   ├── type-safe-api-client.md
│   └── type-safe-event-emitter.md
├── 4-type-inference/       # Type inference techniques
│   ├── assertion-functions.md
│   ├── const-type-parameters.md
│   ├── infer-keyword.md
│   ├── inference-failure.md
│   └── type-guards.md
├── 5-best-practices/       # Guidelines and recommendations
│   ├── dos-and-donts.md
│   ├── performance.md
│   └── type-testing.md
├── 6-pitfalls/            # Common mistakes and how to avoid them
├── 7-performance/         # Type checking and build optimization
├── 8-debugging/           # Complex errors and module resolution
├── 9-migration/           # JS to TS and tool migrations
├── 10-monorepo/           # Nx, Turborepo, project references
├── 11-tooling/            # Biome, ESLint, type testing
├── 12-checklists/          # Decision trees and code review
├── references/            # Detailed reference guides
└── SKILL.md              # Main skill file
```

## Topics Covered

- **Generics & Constraints** — Reusable, type-safe components
- **Conditional Types** — Type-level logic and branching
- **Mapped Types** — Property transformation and iteration
- **Template Literal Types** — Type-safe string patterns
- **Branded Types** — Nominal typing for domain primitives
- **Higher Kinded Types** — Functor/Monad patterns in TypeScript
- **Type Inference** — Extraction and automatic type discovery
- **Type Guards & Assertions** — Runtime narrowing for compile-time safety
- **Design Patterns** — Event emitters, API clients, builders, validators
- **Performance** — Type checking speed and build optimization
- **Migration** — JavaScript to TypeScript, tool upgrades
- **Monorepo** — Project references, Nx, Turborepo

## Quick Start

1. Start with [Core Concepts](./1-core-concepts/README.md) for foundational knowledge
2. Progress to [Advanced Types](./2-advanced-types/README.md) for type-level programming
3. Apply [Patterns](./3-patterns/README.md) to real-world scenarios
4. Master [Type Inference](./4-type-inference/README.md) for automatic typing
5. Follow [Best Practices](./5-best-practices/README.md) for clean code

## For AI Agents

This documentation follows specific constraints:

- Non-interactive operations only
- Never skip hooks or validation
- Use `--force-with-lease` for branch updates
- Type-first API design workflow
- Branded types for domain modeling
- Discriminated unions for state machines

## Contributing

Contributions welcome! Please read the documentation and follow the existing patterns.

## License

MIT
