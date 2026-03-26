# Const Type Parameters

TypeScript 5.0 feature for forcing literal type inference instead of widened types.

## The Problem: Widened Literals

Without `const`, TypeScript widens literal types:

```typescript
function infer<T>(value: T) {
  return value;
}

const name = infer("John");
//    ^? string (widened!), not "John"
```

This is often undesirable when you need the literal type.

## The Solution: `const` Modifier

Use `const` on type parameters to force literal inference:

```typescript
function inferConst<T extends const>(value: T) {
  return value;
}

const name = inferConst("John");
//    ^? "John" (literal type!)

const nums = inferConst([1, 2, 3]);
//    ^? readonly [1, 2, 3] (not number[])
```

## How It Works

The `const` modifier tells TypeScript to infer the narrowest possible type:

```typescript
// Without const - widened
function id<T>(x: T): T {
  return x;
}
const a = id("hello");  // string

// With const - literal
function idConst<T extends const>(x: T): T {
  return x;
}
const b = idConst("hello");  // "hello"
```

## Real-World Examples

### Example 1: Route Definition Builder

```typescript
// Without const - routes become generic strings
function createRoute<T extends string>(path: T): { path: T } {
  return { path };
}
const route = createRoute("/users");
// route.path is string, not "/users"

// With const - preserves literal
function createRouteConst<T extends const>(path: T): { path: T } {
  return { path };
}
const routeConst = createRouteConst("/users");
// routeConst.path is "/users" (literal!)
```

**Key difference**: `T extends string` infers `"users"` as `string`. `T extends const` infers `"users"` as `"users"`.
const routeConst = createRouteConst("/users");
// routeConst.path is "/users" (literal!)
```

### Example 2: Event Name Registry

```typescript
// Define event names that preserve literal types
function defineEvents<T extends Record<string, string>>(events: T): T {
  return events;
}

const appEvents = defineEvents({
  click: "button:click",
  submit: "form:submit",
  error: "app:error",
});
// All values are literal types: "button:click" | "form:submit" | "app:error"
```

### Example 3: Config Schema Builder

```typescript
interface ConfigSchema<T> {
  defaults: T;
  validate: (config: unknown) => T;
}

function defineConfig<T extends const>(schema: {
  [K in keyof T]: { default: T[K]; description: string }
}): ConfigSchema<T> {
  return {
    defaults: Object.fromEntries(
      Object.entries(schema).map(([k, v]) => [k, v.default])
    ) as T,
    validate: (config: unknown) => config as T,
  };
}

const config = defineConfig({
  port: { default: 3000, description: "Server port" },
  host: { default: "localhost", description: "Server host" },
  debug: { default: false, description: "Debug mode" },
});
// config.defaults.port is 3000 (literal), not number
// config.defaults.host is "localhost" (literal), not string
// config.defaults.debug is false (literal), not boolean
```

### Example 4: CSS Class Name Builder

```typescript
type CssClass<T extends string> = {
  className: T;
  withPrefix: (prefix: string) => `${prefix}-${T}`;
};

function defineClass<T extends string>(name: T): CssClass<T> {
  return {
    className: name,
    withPrefix: (prefix) => `${prefix}-${name}`,
  };
}

const buttonClass = defineClass("btn");
// buttonClass.className is "btn" (literal)
// buttonClass.withPrefix("primary") is "primary-btn" (literal)
```

### Example 5: Union of Literal Objects

```typescript
// Preserve individual object shapes
function defineActions<T extends { type: string } & const>(
  actions: T[]
): T[] {
  return actions;
}

const actions = defineActions([
  { type: "LOGIN", username: "" },
  { type: "LOGOUT" },
  { type: "REFRESH", token: "" },
]);

// Each action preserves its literal type
// actions[0] is { type: "LOGIN"; username: string }
// actions[1] is { type: "LOGOUT" }
// actions[2] is { type: "REFRESH"; token: string }
```

## Comparison Table

| Inferring with | Result for `"hello"` | Result for `[1, 2]` |
|----------------|---------------------|---------------------|
| Normal `<T>` | `string` | `number[]` |
| `<T extends const>` | `"hello"` | `readonly [1, 2]` |
| `as const` on call | `"hello"` | `readonly [1, 2]` |

## When to Use

| Scenario | Use `const` modifier |
|----------|---------------------|
| Route paths | ✓ Preserve literal paths |
| Event names | ✓ Keep specific event strings |
| Config values | ✓ Preserve defaults as literals |
| CSS class names | ✓ Maintain atomic class names |
| Action types | ✓ Keep discriminated union literals |
| Numeric constants | ✓ Preserve specific port numbers, etc. |

## Caveats

1. **TS 5.0+ required** — Not available in earlier versions
2. **Readonly by default** — Arrays become readonly tuples
3. **Cannot be used everywhere** — Sometimes widened types are intentional
4. **Nested inference** — Only top-level, nested objects still widen unless `as const`

```typescript
// Nested objects still widen without const
function test<T extends const>(obj: { nested: T }): T {
  return obj.nested;
}

// This still widens nested properties
const result = test({ nested: { x: 1, y: "hello" } });
// result.x is number, not 1
// result.y is string, not "hello"
```

## Key Takeaways

- `const` modifier forces narrowest possible type inference
- Essential for builders that should preserve literal types
- Enables type-safe string constants, route definitions, config schemas
- Available in TypeScript 5.0+
- Combine with `as const` for objects and arrays when needed
