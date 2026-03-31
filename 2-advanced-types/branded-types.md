# Branded Types

> **Decision Tree**: Can't find what you need? Jump to the section:
> - I have multiple ID types that get mixed up → Strict Branding
> - I need to validate strings (Email, URL, Password) at input → Brand Factory
> - I need to prevent USD + EUR calculations → Number Branding
> - I just want better IDE readability without constraints → Type Flavoring
> - I need success/failure branching at runtime → Discriminated Union (not a Brand!)

---

## Quick Decision Table

| If your need is... | Use this pattern |
|--------------------|------------------|
| Prevent `string` mixing (UserId, PostId) | **Strict Branding** with `unique symbol` |
| Validate data at system entry (Email, URL) | **Brand Factory** (validation + type cast) |
| Prevent currency/unit calculations | **Strict Number Branding** |
| Better IDE autocomplete without blocking | **Type Flavoring** (optional property) |
| Runtime branching based on state | **Discriminated Union** |

---

## Concept Clarification

> **Important**: Branded Types and Discriminated Unions solve different problems:

- **Branded Types**: Compile-time protection against mixing incompatible *values* (e.g., `UserId` vs `PostId`)
- **Discriminated Unions**: Runtime branching based on a *tag property* (e.g., `{ status: "success", data: T } | { status: "error", error: E })

---

## 1. When: Maximum Safety for IDs

**Scenario**: "I have multiple identifier types (User, Post, Org) and I want the compiler to explode if I accidentally pass a PostId where a UserId is expected."

**Solution**: Strict Branding with `unique symbol`

```typescript
// types/brand.ts
declare const __brand: unique symbol;

export type Brand<B> = { readonly [__brand]: B };
export type Branded<T, B> = T & Brand<B>;

// usage.ts
type UserId = Branded<string, "UserId">;
type PostId = Branded<string, "PostId">;

function deleteUser(id: UserId): void { ... }

const postId = "post_1" as PostId;
deleteUser(postId); // ❌ Error: PostId is not assignable to UserId
```

**Why `unique symbol`?** String-based `__brand: "UserId"` can collide or be manually forged. `unique symbol` is a distinct compiler-level symbol that cannot be reproduced.

---

## 2. When: Validate Format (Email, URL, Password)

**Scenario**: "I want to guarantee that a `string` was validated by my Regex before being used in my services."

**Solution**: Brand Factory (validation + type cast)

```typescript
type Email = Branded<string, "Email">;
type URL = Branded<string, "URL">;
type Password = Branded<string, "Password">;

// The ONLY place where "as Email" is allowed
function parseEmail(input: string): Email {
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input)) {
    throw new Error("Invalid email format");
  }
  return input as Email;
}

function sendWelcomeEmail(email: Email): void { ... }

// Forces validation at entry point
const email = parseEmail("user@example.com");
sendWelcomeEmail(email); // ✅ OK
sendWelcomeEmail("not-an-email"); // ❌ Error before even reaching validation
```

> **Senior Note**: In a clean architecture, `as MyBrand` should **only** appear in factory/parser functions. If you see `as Brand` in a UI component, it's technical debt.

---

## 3. When: Financial Calculations or Physical Units

**Scenario**: "I manipulate Euros and Dollars, or Grams and Kilograms. `EUR + USD` must be impossible."

**Solution**: Number Branding

```typescript
type EUR = Branded<number, "EUR">;
type USD = Branded<number, "USD">;
type Grams = Branded<number, "Grams">;
type Kilograms = Branded<number, "Kilograms">;

const wallet = 10 as EUR;
const price = 5 as USD;

// @ts-expect-error: Cannot add different currencies
const total = wallet + price;

const weight = 1000 as Grams;
const maxWeight = 5 as Kilograms;

// @ts-expect-error: Cannot compare different units
if (weight > maxWeight) { ... }
```

---

## 4. When: Developer Experience (DX) Without Constraints

**Scenario**: "I want the IDE to show 'OrderId' instead of 'string' for clarity, but I don't want to force explicit conversions everywhere."

**Solution**: Type Flavoring

```typescript
interface Flavoring<K> {
  readonly _type?: K; // Optional - doesn't really exist at runtime
}

export type Flavor<T, K> = T & Flavoring<K>;

type OrderId = Flavor<string, "OrderId">;

// No explicit conversion needed
let id: OrderId = "123"; // ✅ OK

// IDE shows "OrderId" on hover, but plain string also works
function process(id: OrderId): void { ... }
process("raw_string"); // ✅ Works - helpful but not enforced
```

**When to use Flavoring vs Branding?**

| Use Flavoring when... | Use Branding when... |
|----------------------|---------------------|
| Team needs clearer types during dev | Mixing types causes production bugs |
| You're migrating from `any` | Invalid data causes critical errors |
| You want self-documenting code | External/untrusted data enters your system |

---

## 5. When: Runtime Branching Based on State

**Scenario**: "I need to handle API responses where success and failure follow different paths."

**Solution**: Discriminated Union (not a Brand!)

```typescript
// This is a DISCRIMINATED UNION, not a Brand
type ApiSuccess<T> = T & { readonly _tag: "success" };
type ApiFailure = { readonly _tag: "failure"; error: Error };
type ApiResponse<T> = ApiSuccess<T> | ApiFailure;

function handleResponse<T>(response: ApiResponse<T>): void {
  if (response._tag === "success") {
    // TypeScript knows response is ApiSuccess<T>
    console.log("Data:", response);
  } else {
    // TypeScript knows response is ApiFailure
    console.error("Error:", response.error);
  }
}
```

> **Note**: The `_tag` property exists at runtime and controls flow. A Brand's `_brand` property is compile-time only and is erased.

---

## ⚠️ Senior Best Practices

### 1. Zero-Cost Abstraction

Branded Types are **completely erased at compilation**. There's no impact on bundle size or runtime performance:

```typescript
type UserId = Branded<string, "UserId">;
// Compiles to just: string
```

### 2. JSON Persistence

When you receive JSON from an API, Brand information is lost. You **must** re-instantiate via your factory functions:

```typescript
// Send: UserId is preserved through serialization
const userId: UserId = parseUserId("user_123");
localStorage.setItem("userId", userId); // Stored as plain string

// Receive: Must re-validate
const stored = localStorage.getItem("userId");
if (stored) {
  const userId = parseUserId(stored); // Re-instantiate with brand
}
```

### 3. Interoperability

A `UserId` (which is `string & Brand<"UserId">`) **remains a string**. You can call all string methods without unwrapping:

```typescript
type UserId = Branded<string, "UserId">;
const id = "USER_123" as UserId;

id.toLowerCase(); // ✅ Works - string methods preserved
id.length;        // ✅ Works
id.split("_");    // ✅ Works
```

### 4. Where `as` Should Appear

```typescript
// ✅ GOOD: Factory function - the only place "as Brand" belongs
function parseEmail(input: string): Email {
  if (!isValid(input)) throw new Error("Invalid");
  return input as Email; // Validated here
}

// ❌ BAD: Component - indicates technical debt
const email = userInput as Email; // Should use parseEmail() instead
```

---

## Quick Reference: Branding vs Flavoring vs Discriminated Union

| Pattern | Use Case | Runtime Property | Enforcement |
|---------|----------|-----------------|-------------|
| **Strict Brand** | IDs, Currencies, Units | None (erased) | Compile-time |
| **Flavoring** | DX clarity, self-documenting code | Optional `?` | None (helpful only) |
| **Discriminated Union** | Success/error states, state machines | Required tag | Runtime branching |

---

## Key Takeaways

- **Branded Types** prevent mixing incompatible values at compile-time
- **`unique symbol`** is safer than string-based brands (cannot be forged)
- **Brand Factories** enforce validation at the system entry point
- **Flavoring** is for DX only—doesn't actually prevent errors
- **Discriminated Unions** are for runtime branching, not type safety
- Brands are **zero-cost**—erased at compilation
- JSON **destroys** brands—re-validate on deserialization
- **`as Brand` belongs only in factory/parser functions**
