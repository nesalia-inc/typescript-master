# Type-Safe Event Emitter

A generic event system with full type safety for event names and payloads.

## The Pattern

```typescript
type EventMap = {
  "user:created": { id: string; name: string };
  "user:updated": { id: string; changes: Partial<{ name: string; email: string }> };
  "user:deleted": { id: string };
  "session:expired": { userId: string; reason: "timeout" | "logout" };
};

class TypedEventEmitter<T extends Record<string, any>> {
  private listeners: {
    [K in keyof T]?: Array<(data: T[K]) => void>;
  } = {};

  on<K extends keyof T>(event: K, callback: (data: T[K]) => void): () => void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(callback);

    // Return unsubscribe function
    return () => this.off(event, callback);
  }

  off<K extends keyof T>(event: K, callback: (data: T[K]) => void): void {
    const callbacks = this.listeners[event];
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }

  emit<K extends keyof T>(event: K, data: T[K]): void {
    const callbacks = this.listeners[event];
    if (callbacks) {
      callbacks.forEach((callback) => callback(data));
    }
  }

  once<K extends keyof T>(event: K, callback: (data: T[K]) => void): void {
    const unsubscribe = this.on(event, (data) => {
      callback(data);
      unsubscribe();
    });
  }
}
```

## Usage

```typescript
const emitter = new TypedEventEmitter<EventMap>();

// Type-safe listener
emitter.on("user:created", (data) => {
  console.log(data.id, data.name); // Both fully typed!
});

// Type-safe emit
emitter.emit("user:created", { id: "1", name: "John" });

// Error: missing 'name'
emitter.emit("user:created", { id: "1" }); // Error!

// Error: 'email' doesn't exist in type
emitter.emit("user:updated", { id: "1", email: "j@x.com" }); // Error!

// Correct partial update
emitter.emit("user:updated", { id: "1", changes: { name: "Jane" } });
```

## Async Event Emitter

```typescript
class AsyncTypedEventEmitter<T extends Record<string, any>> {
  private listeners: {
    [K in keyof T]?: Array<(data: T[K]) => Promise<void>>;
  } = {};

  async on<K extends keyof T>(
    event: K,
    callback: (data: T[K]) => Promise<void>
  ): Promise<() => void> {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(callback);
    return () => this.off(event, callback);
  }

  off<K extends keyof T>(event: K, callback: (data: T[K]) => Promise<void>): void {
    const callbacks = this.listeners[event];
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index > -1) callbacks.splice(index, 1);
    }
  }

  async emit<K extends keyof T>(event: K, data: T[K]): Promise<void> {
    const callbacks = this.listeners[event];
    if (callbacks) {
      await Promise.all(callbacks.map((cb) => cb(data)));
    }
  }
}
```

## Key Techniques

1. **Mapped type for listeners** — `{ [K in keyof T]?: Array<(data: T[K]) => void> }`
2. **Constrained type parameter** — `T extends Record<string, any>`
3. **Generic method constraints** — `K extends keyof T`
4. **Unsubscribe pattern** — Return cleanup function from `on()`

## When to Use

- Building pub/sub systems
- Event-driven architectures
- State management systems
- WebSocket or real-time communication layers
- Component communication
