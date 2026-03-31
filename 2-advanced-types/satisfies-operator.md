# The Satisfies Operator (TS 4.9+)

> **Decision Tree**: Can't find what you need? Jump to the section:
> - I need validation AND preserve exact literal types → `satisfies`
> - I need immutable config with validation → `as const satisfies`
> - I just want to force a type without inference → **Type Annotation** (`:`)
> - I need to silence a type error I know is safe → **Type Casting** (`as`) ⚠️

---

## Quick Decision Table

| If your need is... | Use this pattern |
|--------------------|------------------|
| Validate structure **and** keep exact literal types | **`satisfies`** |
| Validate immutables (routes, config) | **`as const satisfies`** |
| Force variable to be exactly a type (accept widening) | **Annotation** (`const x: T = ...`) |
| Silence a compiler error you know is safe | **Casting** (`as T`) ⚠️ Dangerous |

---

## Concept Clarification: Three Ways to Assign

| Operator | What it does | Inference preserved? |
|----------|--------------|---------------------|
| `const x: T = ...` | **Annotation**: Forces the type, accepts widening | ❌ No - widens to `T` |
| `const x = ... as T` | **Casting**: Tells TS "trust me" | ❌ No - widens to `T` |
| `const x = ... satisfies T` | **Satisfies**: Validates without widening | ✅ Yes - keeps exact type |

---

## 1. When: Validate Without Losing Precision

**Scenario**: "I create a configuration object. I want to ensure it matches my interface, but I want the IDE to show me the exact keys and values I defined."

**Solution**: `satisfies` vs `:`

```typescript
type Colors = "red" | "green" | "blue";
type Theme = Record<string, Colors | { custom: string }>;

// ❌ With annotation (:): loses exact values
const theme1: Theme = {
  primary: "red",
  secondary: "blue"
};
theme1.primary; // Type: Colors | { custom: string } (too broad!)

// ✅ With satisfies: validates AND preserves exact values
const theme2 = {
  primary: "red",
  secondary: "blue"
} satisfies Theme;

theme2.primary; // Type: "red" (exact literal!)
theme2.other = true; // Error: excess properties caught
```

---

## 2. When: Validate String Patterns (Template Literals)

**Scenario**: "I want to ensure all strings in my object are URLs or HexColors, without turning everything into a generic `string`."

**Solution**: Pattern validation without widening

```typescript
type HexColor = `#${string}`;
type URL = `https://${string}`;

const uiConfig = {
  background: "#ffffff",
  border: "#000000",
  homepage: "https://example.com",
  opacity: 0.5
} satisfies Record<string, HexColor | URL | number>;

// IDE knows background is "#ffffff", homepage is "https://example.com"
uiConfig.background.toUpperCase(); // ✅ Valid - knows exact string
uiConfig.opacity.toFixed();       // ✅ Valid - knows it's a number
uiConfig.background = "invalid";   // ❌ Error: not a HexColor
```

---

## 3. When: Ultimate Configuration (Immutable + Validated)

**Scenario**: "I define application routes. They must never change (readonly) AND I want to validate they follow my format."

**Solution**: `as const satisfies`

```typescript
type Route = { path: string; children?: Route[] };

const ROUTES = {
  HOME: { path: "/" },
  USER: {
    path: "/user",
    children: [{ path: "/profile" }]
  }
} as const satisfies Record<string, Route>;

// 1. satisfies validates the structure
// 2. as const makes everything readonly AND preserves literal strings
// 3. Full autocomplete on ROUTES.USER.children[0].path

ROUTES.HOME.path;                        // Type: "/"
ROUTES.HOME.path = "/other";             // ❌ Error: readonly
ROUTES.USER.children![0].path;          // Type: "/profile"
```

---

## 4. When: Tuple Inference

**Scenario**: "I want an array to be typed as exactly a 2-element tuple, not as `number[]`."

```typescript
// ❌ Annotation widens to array
const coords1: [number, number] = [10, 20];
coords1; // [number, number] ✅

// ✅ satisfies preserves tuple AND validates contents
const coords2 = [10, 20] satisfies [number, number];
coords2; // [10, 20] - exact values preserved

// Validation works
const invalid = [10, 20, 30] satisfies [number, number]; // ❌ Error
```

---

## 5. When: Mapped Types with Methods

**Scenario**: "I have an object with methods. I want to validate it matches an interface while keeping method signatures exact."

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

// Methods are preserved exactly
counter.increment(); // Type: () => void (not just Function)
```

---

## 6. When: API Endpoints Validation

**Scenario**: "I define API endpoints. I want method and path to be literal strings, not just `string`."

```typescript
type Method = "GET" | "POST" | "PUT" | "DELETE";

type Endpoints = {
  [K in string]: { method: Method; path: string };
};

const api = {
  getUser: { method: "GET", path: "/users/:id" },
  createUser: { method: "POST", path: "/users" },
  updateUser: { method: "PUT", path: "/users/:id" }
} satisfies Endpoints;

// api.getUser.method is "GET" (literal), not string
type MethodType = typeof api.getUser.method;
// "GET" | "POST" | "PUT"

api.getUser.method = "PATCH"; // ❌ Error: "PATCH" is not a valid Method
```

---

## ⚠️ Senior Best Practices

### 1. Stop Using `as Record<string, T>`

```typescript
// ❌ BAD: 'as' disables excess property checks
const bad = { extra: true } as Record<string, string | number>;

// ✅ GOOD: 'satisfies' catches excess properties
const good = { extra: true } satisfies Record<string, string | number>;
// Error: Object literal may only specify known properties
```

### 2. Localized Errors

`satisfies` errors appear **exactly on the problematic property**, not on the whole object:

```typescript
const config = {
  api: "https://api.example.com",
  timeout: "not-a-number" // ❌ Error here specifically
} satisfies Record<string, string | number>;
```

### 3. When to Use Each

| Pattern | When to use |
|---------|-------------|
| `satisfies` | Configuration objects, validation at definition |
| `as const satisfies` | Constants, routes, enum-like objects |
| `as T` | Only when you know better than TS (rare, dangerous) |
| `: T` annotation | When you intentionally want widening |

---

## Key Takeaways

- **`satisfies`** validates without widening—keeps exact literal types
- **`as const satisfies`** combines immutability + validation + literal preservation
- **Annotation (`:`)** widens—use when you intentionally want generalization
- **Casting (`as`)** bypasses checks—avoid unless necessary
- `satisfies` catches excess properties errors where `as` would allow them
- Errors are localized to the specific property, not the whole object
