# Generics

Create reusable, type-flexible components while maintaining full type safety.

## Basic Generic Function

```typescript
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(42);    // Type: number
const str = identity<string>("hello"); // Type: string
const auto = identity(true);          // Type inferred: boolean
```

## Generic Constraints

Constrain the type parameter to only accept types that satisfy certain conditions:

```typescript
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): T {
  console.log(item.length);
  return item;
}

logLength("hello");           // OK: string has length
logLength([1, 2, 3]);         // OK: array has length
logLength({ length: 10 });    // OK: object has length
// logLength(42);             // Error: number has no length
```

## Multiple Type Parameters

```typescript
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge({ name: "John" }, { age: 30 });
// Type: { name: string } & { age: number }
```

## Generic Interfaces

```typescript
interface Container<T> {
  value: T;
  map<U>(fn: (value: T) => U): Container<U>;
}

function wrap<T>(value: T): Container<T> {
  return {
    value,
    map(fn) {
      return wrap(fn(value));
    },
  };
}

const num = wrap(42).map((n) => n.toString());
// Type: Container<string>
```

## Generic Classes

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
}

const stack = new Stack<number>();
stack.push(1);
stack.push(2);
const top = stack.pop(); // Type: number | undefined
```

## Generic Type Aliases

```typescript
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) {
    return { ok: false, error: "Division by zero" };
  }
  return { ok: true, value: a / b };
}
```

## Real-World Examples

### Example 1: Type-Safe Event Emitter

When building a pub/sub system for a package, generics ensure event payloads are correctly typed:

```typescript
class TypedEventEmitter<Events extends Record<string, unknown>> {
  private listeners: {
    [K in keyof Events]?: Array<(data: Events[K]) => void>;
  } = {};

  on<K extends keyof Events>(event: K, callback: (data: Events[K]) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(callback);
  }

  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    this.listeners[event]?.forEach((cb) => cb(data));
  }
}

// Usage in a package for WebSocket messages
interface SocketEvents {
  message: { id: string; content: string };
  connection: { status: "connected" | "disconnected" };
  error: { code: number; message: string };
}

const emitter = new TypedEventEmitter<SocketEvents>();
emitter.emit("message", { id: "1", content: "Hello" }); // OK
emitter.emit("message", { id: "1" }); // Error: missing 'content'
```

### Example 2: Generic Repository Pattern for ORM

When building an ORM package, generics provide type-safe CRUD operations:

```typescript
interface Entity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Repository<T extends Entity> {
  findById(id: string): Promise<T | null>;
  findAll(options?: { limit?: number; offset?: number }): Promise<T[]>;
  save(entity: Omit<T, "id" | "createdAt" | "updatedAt">): Promise<T>;
  delete(id: string): Promise<void>;
}

class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User | null> {
    // Database query
    return db.query("SELECT * FROM users WHERE id = $1", [id]);
  }

  async findAll(options?: { limit?: number; offset?: number }): Promise<User[]> {
    // Database query with pagination
    return db.query("SELECT * FROM users LIMIT $1 OFFSET $2", [options?.limit, options?.offset]);
  }

  async save(data: Omit<User, "id" | "createdAt" | "updatedAt">): Promise<User> {
    // Insert and return created user
    return db.query("INSERT INTO users (...) VALUES (...) RETURNING *", data);
  }

  async delete(id: string): Promise<void> {
    db.query("DELETE FROM users WHERE id = $1", [id]);
  }
}
```

### Example 3: Generic Config Validator for Libraries

When building a configuration system for a package:

```typescript
interface ConfigSchema<T> {
  validate(config: unknown): T;
  defaults: Partial<T>;
}

class StringConfigOption<T extends string> {
  constructor(
    private key: string,
    private allowedValues: readonly T[]
  ) {}

  validate(value: unknown): T {
    if (typeof value !== "string" || !this.allowedValues.includes(value as T)) {
      throw new Error(`Invalid value for ${this.key}: expected one of ${this.allowedValues.join(", ")}`);
    }
    return value as T;
  }
}

// Usage
const envConfig: ConfigSchema<{ nodeEnv: "development" | "production" | "test"; port: number }> = {
  validate(raw) {
    const config = raw as Record<string, unknown>;
    return {
      nodeEnv: new StringConfigOption("nodeEnv", ["development", "production", "test"]).validate(config.nodeEnv),
      port: typeof config.port === "number" ? config.port : 3000,
    };
  },
  defaults: { port: 3000 },
};
```

### Example 4: Generic Middleware Chain for HTTP Frameworks

When building a web framework or middleware system:

```typescript
interface Middleware<TContext, TResult> {
  handle(context: TContext, next: () => TResult): TResult;
}

class MiddlewareChain<TContext, TResult> {
  private middlewares: Middleware<TContext, TResult>[] = [];

  use(middleware: Middleware<TContext, TResult>): this {
    this.middlewares.push(middleware);
    return this;
  }

  execute(context: TContext): TResult {
    let index = 0;

    const next = (): TResult => {
      if (index >= this.middlewares.length) {
        throw new Error("Middleware chain exhausted without result");
      }
      const middleware = this.middlewares[index++];
      return middleware.handle(context, next);
    };

    return next();
  }
}

// Usage for request processing
interface RequestContext {
  path: string;
  method: string;
  headers: Record<string, string>;
  body: unknown;
}

interface ResponseResult {
  status: number;
  body: unknown;
}

const chain = new MiddlewareChain<RequestContext, ResponseResult>();

chain.use(async (ctx, next) => {
  console.log(`[${ctx.method}] ${ctx.path}`);
  return next();
});

chain.use(async (ctx, next) => {
  if (!ctx.headers["authorization"]) {
    return { status: 401, body: { error: "Unauthorized" } };
  }
  return next();
});
```

## When to Use Generics

| Scenario | Example |
|----------|---------|
| Building reusable utilities/libraries | `Array.map<U>`, `Promise.then<U>` |
| Creating type-safe data structures | `Stack<T>`, `Tree<T>`, `Map<K, V>` |
| Implementing repository/data access layers | `Repository<T extends Entity>` |
| Building middleware or chain-of-responsibility | `Middleware<TContext, TResult>` |
| Creating event systems | `TypedEventEmitter<Events>` |
| Type-safe configuration systems | `ConfigValidator<T>` |

## Key Takeaways

- Generics let you write code that works with any type while maintaining type safety
- Use `extends` to constrain type parameters to specific shapes
- Multiple type parameters allow composition of generic types
- Type inference often eliminates the need to explicitly specify `<T>`
- Generic classes and interfaces are the foundation of reusable library code
