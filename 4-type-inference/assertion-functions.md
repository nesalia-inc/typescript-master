# Assertion Functions

Compile-time type enforcement after runtime checks.

## Basic Assertion Function

```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error(`Expected string, got ${typeof value}`);
  }
}

function process(value: unknown) {
  assertIsString(value);
  // value is typed as string here
  console.log(value.toUpperCase());
}

process("hello"); // OK
process(123);    // Throws Error
```

## Assertion with Custom Error

```typescript
class ValidationError extends Error {
  constructor(
    message: string,
    public path: string
  ) {
    super(message);
    this.name = "ValidationError";
  }
}

function assertHasProperty<
  T extends object,
  K extends string
>(
  obj: T,
  key: K,
  message?: string
): asserts obj is T & Record<K, unknown> {
  if (!(key in obj)) {
    throw new ValidationError(
      message ?? `Missing property: ${key}`,
      key
    );
  }
}

const data = { name: "John", age: 30 };

assertHasProperty(data, "name", "Name is required");
console.log(data.name); // Type: string

assertHasProperty(data, "email", "Email is required"); // Throws ValidationError
```

## Combining with Type Guards

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function assertIsString(value: unknown): asserts value is string {
  if (!isString(value)) {
    throw new Error("Not a string");
  }
}

// Assertion functions can use type guards internally
function isNonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}

function assertNonNullable<T>(
  value: T,
  message = "Value cannot be null or undefined"
): asserts value is NonNullable<T> {
  if (!isNonNullable(value)) {
    throw new Error(message);
  }
}
```

## Assertion for Required Properties

```typescript
type User = {
  name: string;
  email: string;
  age?: number;
};

function assertValidUser(obj: unknown): asserts obj is User {
  if (typeof obj !== "object" || obj === null) {
    throw new Error("User must be an object");
  }

  const user = obj as Record<string, unknown>;

  if (typeof user.name !== "string" || user.name.length === 0) {
    throw new Error("User name is required");
  }

  if (typeof user.email !== "string" || !user.email.includes("@")) {
    throw new Error("Valid email is required");
  }
}

const rawData = JSON.parse('{ "name": "John", "email": "john@example.com" }');

assertValidUser(rawData);
// rawData is now typed as User
console.log(rawData.name); // string
```

## vs Type Guards

| Aspect | Type Guard | Assertion Function |
|--------|------------|-------------------|
| Return type | `value is T` | `asserts value is T` |
| Control flow | `if/else` needed | Narrows after call |
| Error handling | Returns boolean | Throws on failure |
| Use case | Conditional logic | Validation |

## Practical: API Response Validation

```typescript
interface ApiSuccess<T> {
  success: true;
  data: T;
}

interface ApiFailure {
  success: false;
  error: string;
}

type ApiResponse<T> = ApiSuccess<T> | ApiFailure;

function assertSuccess<T>(
  response: ApiResponse<T>
): asserts response is ApiSuccess<T> {
  if (!response.success) {
    throw new Error(`API Error: ${response.error}`);
  }
}

async function fetchUser(): Promise<User> {
  const response = await fetch("/api/user");
  const data: ApiResponse<User> = await response.json();

  assertSuccess(data);
  // data.data is available here
  return data.data;
}
```

## Exhaustive Assertions

```typescript
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${JSON.stringify(value)}`);
}

type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      return assertNever(shape); // Error if Shape is extended
  }
}
```

## Real-World Examples

### Example 1: Environment Variable Validation

```typescript
class EnvError extends Error {
  constructor(
    message: string,
    public variableName: string
  ) {
    super(message);
    this.name = "EnvError";
  }
}

function assertEnv(name: string): string {
  const value = process.env[name];
  if (value === undefined) {
    throw new EnvError(`Missing required environment variable: ${name}`, name);
  }
  return value;
}

function assertEnvNumber(name: string): number {
  const value = assertEnv(name);
  const parsed = parseInt(value, 10);
  if (isNaN(parsed)) {
    throw new EnvError(`Environment variable ${name} must be a number`, name);
  }
  return parsed;
}

// Usage
const PORT = assertEnvNumber("PORT");
const DATABASE_URL = assertEnv("DATABASE_URL");
const API_KEY = assertEnv("API_KEY");
```

### Example 2: JSON Schema Validation

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age?: number;
}

function assertIsUser(data: unknown): asserts data is User {
  if (typeof data !== "object" || data === null) {
    throw new Error("User must be an object");
  }

  const obj = data as Record<string, unknown>;

  if (typeof obj.id !== "string" || obj.id.length === 0) {
    throw new Error("User.id must be a non-empty string");
  }

  if (typeof obj.name !== "string" || obj.name.length === 0) {
    throw new Error("User.name must be a non-empty string");
  }

  if (typeof obj.email !== "string" || !obj.email.includes("@")) {
    throw new Error("User.email must be a valid email");
  }

  if (obj.age !== undefined && typeof obj.age !== "number") {
    throw new Error("User.age must be a number if provided");
  }
}

// Usage
const data = JSON.parse('{"id": "1", "name": "John", "email": "john@example.com"}');
assertIsUser(data);
console.log(data.name); // TypeScript knows data is User
```

### Example 3: State Machine Assertions

```typescript
type RequestState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: Error };

function assertSuccess<T>(state: RequestState): asserts state is { status: "success"; data: T } {
  if (state.status !== "success") {
    throw new Error(`Expected success state, got ${state.status}`);
  }
}

function assertHasData(state: RequestState): asserts state is { status: "success"; data: unknown } {
  if (state.status !== "success" || !("data" in state)) {
    throw new Error("State must have data");
  }
}

// Usage
async function loadUser(): Promise<User> {
  const state: RequestState = await fetchUserState();

  assertSuccess<User>(state);
  // state.data is User here
  return state.data;
}
```

### Example 4: Assert for Exhaustive Matching

```typescript
type PaymentMethod = "credit_card" | "debit_card" | "paypal" | "bank_transfer";

type PaymentState =
  | { method: "credit_card"; cardNumber: string }
  | { method: "debit_card"; cardNumber: string }
  | { method: "paypal"; email: string }
  | { method: "bank_transfer"; accountNumber: string };

function assertPaymentMethod(
  state: PaymentState,
  expected: PaymentMethod
): asserts state is PaymentState {
  if (state.method !== expected) {
    throw new Error(`Expected payment method ${expected}, got ${state.method}`);
  }
}

function processPayment(state: PaymentState): string {
  switch (state.method) {
    case "credit_card":
      return `Charging card ${state.cardNumber}`;
    case "debit_card":
      return `Debiting card ${state.cardNumber}`;
    case "paypal":
      return `PayPal to ${state.email}`;
    case "bank_transfer":
      return `Bank transfer to ${state.accountNumber}`;
    default:
      // Exhaustive check - if we add a new method, this fails
      const _exhaustive: never = state;
      throw new Error(`Unhandled payment method: ${_exhaustive}`);
  }
}
```

### Example 5: Type-Safe Config Loading

```typescript
interface Config {
  server: {
    port: number;
    host: string;
  };
  database: {
    url: string;
    poolSize: number;
  };
}

function assertValidConfig(data: unknown): asserts data is Config {
  if (typeof data !== "object" || data === null) {
    throw new Error("Config must be an object");
  }

  const config = data as Record<string, unknown>;

  // Check server
  if (typeof config.server !== "object" || config.server === null) {
    throw new Error("Config.server must be an object");
  }
  const server = config.server as Record<string, unknown>;
  if (typeof server.port !== "number") {
    throw new Error("Config.server.port must be a number");
  }
  if (typeof server.host !== "string") {
    throw new Error("Config.server.host must be a string");
  }

  // Check database
  if (typeof config.database !== "object" || config.database === null) {
    throw new Error("Config.database must be an object");
  }
  const db = config.database as Record<string, unknown>;
  if (typeof db.url !== "string") {
    throw new Error("Config.database.url must be a string");
  }
  if (typeof db.poolSize !== "number") {
    throw new Error("Config.database.poolSize must be a number");
  }
}

// Usage
const rawConfig = JSON.parse(fs.readFileSync("config.json", "utf-8"));
assertValidConfig(rawConfig);
// rawConfig is Config here
console.log(rawConfig.server.port);
```

## Best Practices

1. Use assertion functions for validation that should not return
2. Provide clear error messages with context
3. Combine with custom error classes for better error handling
4. Consider using libraries like `zod` or `yup` for complex validation
5. Use `asserts` at module load time for config validation
6. For complex schemas, prefer validation libraries over manual assertions
