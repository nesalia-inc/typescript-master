# Patterns Reference

Builder, factory, and type-safe API patterns.

## Builder Pattern

```typescript
type BuilderState<T> = { [K in keyof T]?: T[K] };

class Builder<T, S extends BuilderState<T> = {}> {
  private state: S = {} as S;

  set<K extends keyof T>(key: K, value: T[K]): Builder<T, S & Record<K, T[K]>> {
    (this.state as any)[key] = value;
    return this as any;
  }

  build(this: { state: S } & (IsComplete<T, S> extends true ? {} : never)): T {
    return this.state as T;
  }
}
```

## Factory Pattern

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

type UserCreationParams = Omit<User, "id">;

function createUser(params: UserCreationParams): User {
  return {
    id: crypto.randomUUID(),
    ...params,
  };
}
```

## Result Type

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function tryCatch<T>(fn: () => T): Result<T> {
  try {
    return { ok: true, value: fn() };
  } catch (error) {
    return { ok: false, error: error as Error };
  }
}
```

## Type-Safe Event Emitter

```typescript
type EventMap = Record<string, any>;

class TypedEmitter<T extends EventMap> {
  private listeners: { [K in keyof T]?: Array<(data: T[K]) => void> } = {};

  on<K extends keyof T>(event: K, fn: (data: T[K]) => void): () => void {
    if (!this.listeners[event]) this.listeners[event] = [];
    this.listeners[event]!.push(fn);
    return () => this.off(event, fn);
  }

  emit<K extends keyof T>(event: K, data: T[K]): void {
    this.listeners[event]?.forEach((fn) => fn(data));
  }
}
```

## Annotated Pattern

```typescript
type Annotated<T, Ann> = T & Ann;

type Validated<T> = T & { __validated: true };

function validate<T>(value: T): Validated<T> {
  // validation logic
  return value as Validated<T>;
}
```

## Discriminated Union State Machine

```typescript
type State =
  | { type: "idle" }
  | { type: "fetching" }
  | { type: "success"; data: unknown }
  | { type: "error"; error: Error };

type Event =
  | { type: "FETCH" }
  | { type: "SUCCESS"; data: unknown }
  | { type: "ERROR"; error: Error }
  | { type: "RESET" };

function transition(state: State, event: Event): State {
  switch (state.type) {
    case "idle":
      return event.type === "FETCH" ? { type: "fetching" } : state;
    case "fetching":
      if (event.type === "SUCCESS") return { type: "success", data: event.data };
      if (event.type === "ERROR") return { type: "error", error: event.error };
      return state;
    case "success":
    case "error":
      return event.type === "RESET" ? { type: "idle" } : state;
  }
}
```
