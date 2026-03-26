# The Satisfies Operator (TS 4.9+)

Validate that an expression matches a type without widening.

## The Problem with `as`

```typescript
const config = {
  api: "https://api.example.com",
  timeout: 5000,
} as Record<string, string | number>;

config.api;     // string
config.timeout; // number

// But also:
config.other = true; // Allowed! No type safety!
```

## The `satisfies` Solution

```typescript
const config = {
  api: "https://api.example.com",
  timeout: 5000,
} satisfies Record<string, string | number>;

// Type is preserved: { api: string; timeout: number }
config.api;     // string
config.timeout; // number

// But invalid assignments fail:
config.other = true; // Error!
```

## Preserving Literal Types

```typescript
// With 'as': literal types are widened
const Routes = {
  home: "/",
  about: "/about",
  contact: "/contact",
} as const;
// Type: { readonly home: "/"; readonly about: "/about"; ... }

// With 'satisfies': validates AND preserves literal types
type Route = "/" | "/about" | "/contact";

const Routes = {
  home: "/",
  about: "/about",
  contact: "/contact",
} satisfies Record<string, Route>;

// Routes.home is "/" (literal), not just string
```

## Validation Without Widening

```typescript
type Color = "red" | "green" | "blue";
type Palette = Record<string, Color>;

const palette = {
  primary: "red",
  secondary: "blue",
  accent: "yellow", // Error! 'yellow' is not in Color
} satisfies Palette;
```

## With Mapped Types

```typescript
type Point = { x: number; y: number };

const points = {
  a: { x: 0, y: 0 },
  b: { x: 1, y: 1 },
} satisfies Record<string, Point>;

// Each point property is validated as Point
// AND each key is preserved as literal
type PointKey = keyof typeof points;
// "a" | "b"
```

## Practical Use Cases

### API Endpoints

```typescript
type Method = "GET" | "POST" | "PUT" | "DELETE";

type Endpoints = {
  [K in string]: { method: Method; path: string };
};

const api = {
  getUser: { method: "GET", path: "/users/:id" },
  createUser: { method: "POST", path: "/users" },
  updateUser: { method: "PUT", path: "/users/:id" },
} satisfies Endpoints;

// api.getUser.method is "GET", not string
```

### Theme Configuration

```typescript
type HexColor = `#${string}`;

const theme = {
  primary: "#ff0000",
  secondary: "#00ff00",
  accent: "#0000ff",
} satisfies Record<string, HexColor>;

// Accidentally using invalid color:
const badTheme = {
  primary: "red", // Error! Not a HexColor
} satisfies Record<string, HexColor>;
```

### Object with Methods

```typescript
type Counter = {
  increment: () => void;
  count: number;
};

const counter = {
  count: 0,
  increment() {
    this.count++;
  },
} satisfies Counter;

// Methods are preserved
counter.increment();
```

## Key Takeaways

- Use `satisfies` to validate against a type
- Preserves literal types unlike `as`
- Catches invalid values at compile time
- Useful for configuration objects
- Doesn't widen the type, keeps precise inference
