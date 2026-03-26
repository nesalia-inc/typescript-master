# Advanced Patterns

Real-world applications of TypeScript's type system for building robust, type-safe libraries and utilities.

## Topics

- [Type-Safe Event Emitter](./type-safe-event-emitter.md) — Generic event system
- [Type-Safe API Client](./type-safe-api-client.md) — End-to-end typed HTTP client
- [Builder Pattern](./builder-pattern.md) — Type-safe fluent builders
- [Deep Readonly/Partial](./deep-readonly-partial.md) — Recursive utility types
- [Form Validation](./form-validation.md) — Type-safe validation system
- [Discriminated Unions](./discriminated-unions.md) — Type narrowing and state machines

## Pattern Overview

| Pattern | Use Case | Key Technique |
|---------|----------|---------------|
| Event Emitter | Pub/sub systems | Mapped types for listeners |
| API Client | HTTP abstraction | Conditional types for request/response |
| Builder | Complex configuration | Method chaining with type inference |
| Deep Partial | Nested objects | Recursive mapped types |
| Form Validation | User input | Type-safe validation rules |
| Discriminated Unions | State machines | Exhaustive narrowing |

These patterns demonstrate how to apply core concepts to solve real problems.
